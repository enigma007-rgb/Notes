Let me be upfront about something important before we go further.

---

## The Honest Assessment First

SailPoint IdentityIQ is enterprise identity governance software. It is:

**Closed source** — the internal implementation is not publicly available. Training data about its internals is sparse.

**Enterprise proprietary** — most production troubleshooting knowledge lives in: internal customer support tickets, private Compass community posts behind login walls, internal SailPoint consultant knowledge, enterprise customer Slack channels and Teams conversations. Almost none of this is publicly indexed.

**Version-sensitive** — behavior differences between 8.1, 8.2, 8.3, 8.4 are significant and sparsely documented publicly.

**Configuration-dependent** — IIQ behavior is heavily driven by your specific connectors, rules, workflows, and customizations. Generic answers are particularly dangerous here.

This means the hallucination risk map from the previous response applies strongly here:

The LLM has good knowledge of: IIQ concepts, general architecture, publicly documented features, common integration patterns documented in public blogs and the Compass community.

The LLM has moderate knowledge of: common troubleshooting patterns, known issues documented in public forums, general connector behavior.

The LLM has high hallucination risk for: specific version behaviors, internal rule execution details, specific connector edge cases, your customization interactions, anything that requires reading SailPoint source code.

---

## Why DERIVE Still Helps Despite These Limitations

Even with high hallucination risk, DERIVE provides three genuine values for IIQ troubleshooting:

**Value 1 — Structured diagnostic reasoning**
DERIVE forces you to organize your evidence before asking questions. The act of constructing a DERIVE prompt often surfaces the answer yourself. You realize mid-prompt that you haven't checked something obvious.

**Value 2 — Pattern matching against public knowledge**
IIQ troubleshooting patterns that appear in public Compass posts, blogs, and community discussions are in the training data. DERIVE reaches this knowledge better than generic prompts.

**Value 3 — Elimination framework**
Even when the LLM cannot give you the exact answer, it can help you eliminate possibilities systematically and identify what to investigate next. This is diagnostic scaffolding, not diagnostic certainty.

---

## The Modified DERIVE for IIQ Troubleshooting

Because of IIQ's proprietary nature, the V principle (Verify) becomes mandatory at every step, not optional. The framework becomes:

**D — Describe the exact failure with evidence**
**E — Eliminate what you have already ruled out**
**R — Request pattern-matching against known IIQ failure modes**
**I — Insist on diagnostic chain before conclusion**
**V — Verify every specific claim against SailPoint Compass or logs before acting**
**E — Expand one hypothesis at a time**

---

## Scenario 1: Aggregation Failing for a Specific Application

### The Setup

You have an IIQ environment. An aggregation job for a specific application connector is failing. Some accounts aggregate successfully. Some fail silently. Some fail with errors. Users are calling the help desk saying their access review shows missing accounts.

---

### Round 1 — D (Describe the Exact Failure With Evidence)

**Generic prompt:**
"My SailPoint IIQ aggregation is failing. How do I fix it?"

**Generic response:**
```
Check connector configuration.
Review application logs.
Verify network connectivity.
Check account schema.
Review aggregation task settings.
```

This is the IIQ equivalent of "have you tried turning it off and on again." Useless.

**DERIVE Round 1:**

"I am troubleshooting an aggregation failure in SailPoint IdentityIQ 8.3. The application is an Active Directory connector aggregating from a specific OU. The aggregation task is scheduled to run nightly at 2am.

Observed behavior:
- Task starts and shows status RUNNING
- Task runs for approximately 45 minutes (normal runtime is 15 minutes)
- Task completes with status SUCCESS — no error shown in task results
- When I check the aggregation statistics in the task result, it shows: Accounts Scanned: 4,847, Accounts Created: 0, Accounts Updated: 12, Accounts Deleted: 0
- Last week the same task showed: Accounts Scanned: 5,203, Accounts Created: 8, Accounts Updated: 156, Accounts Deleted: 3
- I have verified in Active Directory that there are 5,203 accounts in the target OU
- The 356 missing accounts (5,203 - 4,847) are all in a specific sub-OU that was migrated from a different domain 2 weeks ago

Do NOT tell me to check connector configuration generally — I need to know specifically what about this sub-OU migration could cause accounts to be silently skipped without error. Do NOT suggest this is a permissions issue generally — the service account has identical permissions in both the old and new sub-OU.

Before recommending any fix: walk through every mechanism by which IIQ AD aggregation can silently skip accounts without logging an error. Specifically: what happens when IIQ encounters an account it cannot fully read mid-aggregation? Does it skip and continue or fail the whole task?"

**What DERIVE extracts:**

The model is forced to reason about silent skip mechanisms specifically:

**Silent skip mechanism 1 — Schema attribute read failures:**

When IIQ aggregates an AD account, it attempts to read every attribute defined in the application schema. If a specific account has an attribute value that cannot be serialized — binary data in an unexpected attribute, a value that exceeds configured length limits, a null value in an attribute marked required in the schema — IIQ may log a warning in the application server log (not the task result) and skip the account. The task result shows SUCCESS because the task itself succeeded. The skipped accounts simply never appear in the statistics.

The migration from a different domain is the key signal. Domain migrations often leave accounts with:
- SIDHistory attributes containing values from the old domain
- ObjectGUID values that IIQ has cached from the old domain mapping
- Custom attributes populated in the old domain that are empty or different format in the new domain
- Group memberships that reference groups in the old domain that no longer resolve

**Silent skip mechanism 2 — ObjectGUID collision:**

If IIQ uses ObjectGUID as the native identity (common AD connector configuration), and the migrated accounts have the same sAMAccountName as accounts IIQ previously knew about (even deleted ones), IIQ may be confused about whether to create, update, or skip these accounts. The correlation logic finds no match by ObjectGUID (new domain, new GUID), tries to create, finds a correlation match by sAMAccountName to a deleted account, enters an ambiguous state, and skips.

**Where to look in logs:**

```
# This is NOT in the IIQ UI task results
# Look in: <IIQ_HOME>/logs/sailpoint.log

grep -i "skip\|exception\|error\|warn" sailpoint.log | grep -i "aggregat" | \
  grep "2024-01-15 02:" > aggregation_errors.txt

# Specific patterns to look for:
# "Skipping account" - explicit skip with reason
# "Failed to read attribute" - attribute read failure
# "Duplicate identity" - correlation collision
# "Unable to correlate" - correlation failure that may cause skip
```

---

### Round 2 — E + R (Eliminate and Reach Long Tail)

**DERIVE Round 2:**

"Your Round 1 response identified two mechanisms: attribute read failures and ObjectGUID collision. I checked the sailpoint.log file during the aggregation window.

I found these log entries repeating for the 356 missing accounts:
```
WARN  Aggregator - Failed to process ResourceObject for account: 
      CN=John Smith,OU=MigratedUsers,DC=company,DC=com
      Error: com.sailpoint.connector.ConnectorException: 
      Attribute 'msDS-cloudExtensionAttribute10' value exceeds maximum length
```

I also found:
```
WARN  SailPointContext - Cannot find object of class sailpoint.object.ManagedAttribute 
      for value: S-1-5-21-OLD-DOMAIN-SID-XXXX in attribute memberOf
```

Both errors appear for all 356 missing accounts. The accounts were skipped because of these errors.

Do NOT tell me to check schema configuration generally. Tell me specifically:
1. What the 'value exceeds maximum length' error means in IIQ schema context and where the length limit is configured
2. What the 'Cannot find ManagedAttribute' error for old SID values means for aggregation behavior
3. Which error is causing the skip — both, or is one the primary cause

You are a SailPoint IIQ consultant who has seen AD domain migration aggregation failures specifically. Walk through which of these two errors would cause a skip versus which would cause a warning but continue processing."

**What DERIVE extracts with the evidence:**

Now the model has real log evidence to work with. This is where DERIVE's E (evidence) principle pays off most:

**Error 1 analysis — Attribute length:**

The `msDS-cloudExtensionAttribute10` error is an IIQ schema configuration issue, not an AD issue. IIQ schema definitions for application attributes have a `maxLength` property. If the AD attribute contains a value longer than what the IIQ schema allows, IIQ throws this exception. In the AD connector, this typically causes the entire ResourceObject processing to fail for that account — meaning the account is skipped entirely, not just the specific attribute.

Where the length limit is configured:
```
IIQ UI → Applications → [Your AD Application] → 
Account Schema → Find attribute 'msDS-cloudExtensionAttribute10' → 
Check 'Max Length' field

OR in the application XML:
<AttributeDefinition name="msDS-cloudExtensionAttribute10" 
                     type="string" 
                     maxLength="255">  <!-- This is the limit -->
```

The migrated accounts likely have longer values in this attribute because they came from a different domain where the attribute was used differently.

**Error 2 analysis — Old SID in memberOf:**

The `Cannot find ManagedAttribute` warning for old domain SIDs is a different type of error. IIQ is trying to resolve group memberships (memberOf attribute) and cannot find the group definition in its ManagedAttribute table because the group SID from the old domain no longer exists in IIQ's catalog. This is a warning, not a fatal error — IIQ typically continues processing the account and simply omits the unresolvable group membership.

**Which error causes the skip:**

Error 1 (maxLength) is the primary cause of the skip. It throws a ConnectorException during ResourceObject processing which terminates processing of that account. Error 2 (old SID) is a warning that would not by itself cause a skip — but because Error 1 terminates processing first, you see both errors for the same accounts.

**Immediate diagnostic test:**

Find one of the 356 missing accounts in AD. Check the actual length of `msDS-cloudExtensionAttribute10` for that account:

```powershell
Get-ADUser -Identity "john.smith" -Properties msDS-cloudExtensionAttribute10 | 
  Select-Object @{N='AttrLength';E={$_.msDS-cloudExtensionAttribute10.Length}},
               msDS-cloudExtensionAttribute10
```

If the value is longer than your IIQ schema's maxLength setting, you have confirmed the root cause.

---

### Round 3 — I (Insert Reasoning Before Fix)

**DERIVE Round 3:**

"The diagnostic test confirmed: all 356 missing accounts have msDS-cloudExtensionAttribute10 values between 280-340 characters. The IIQ schema has maxLength set to 255.

Before recommending a fix:

Step 1: Walk through the implications of simply increasing maxLength to 500 in the IIQ schema. What else does maxLength affect — is it purely validation or does it affect database storage?

Step 2: Walk through the alternative of excluding this attribute from the schema entirely. What would we lose?

Step 3: Walk through whether there is an IIQ rule-based approach to truncate or transform the value during aggregation instead of changing the schema.

Step 4: Walk through what happens to the 356 accounts that have been missing from IIQ for 2 weeks — when they successfully aggregate after the fix, will IIQ treat them as new accounts or will it find existing identities? What is the risk of duplicate identity creation?

Only after all four — give me the recommended fix sequence."

**What Round 3 extracts:**

**Step 1 — maxLength implications:**

In IIQ, maxLength on a schema attribute affects two things: validation during aggregation (what you are hitting) and the database column size in the IdentityIQ database. The `spt_link_attribute` table stores account attribute values. If the IIQ database column was created with a specific size constraint, increasing maxLength in the schema without a corresponding database migration may cause a different error — a database truncation or overflow error — when IIQ tries to persist the longer value.

**Critical verification before proceeding:**

```sql
-- Check the actual column definition in your IIQ database
-- (PostgreSQL example)
SELECT column_name, data_type, character_maximum_length
FROM information_schema.columns
WHERE table_name = 'spt_link_attribute' 
  AND column_name LIKE '%value%';

-- If column is VARCHAR(255), increasing schema maxLength to 500
-- without altering the column will move your error from aggregation
-- to database persistence — a different error, not a fix
```

**Step 2 — Excluding the attribute:**

Excluding `msDS-cloudExtensionAttribute10` from the schema means IIQ stops reading and storing this attribute entirely. Implications: any access certification, role mining, or policy that references this attribute stops working. Any report that includes this attribute returns null. Before excluding, search your IIQ configuration:

```
IIQ UI → Debug → Search for 'msDS-cloudExtensionAttribute10' 
across: Roles, Policies, Certifications, Reports, Rules, Workflows
```

If no results: safe to exclude. If results: excluding breaks those configurations.

**Step 3 — Rule-based transformation:**

IIQ supports a Customization Rule on application connectors that runs during aggregation and can modify the ResourceObject before IIQ processes it. This is the cleanest fix — transform the attribute value during ingestion without changing schema or losing data:

```java
// Customization Rule in IIQ (Beanshell)
// Applied to AD application: Application → Aggregation → Customization Rule

import sailpoint.object.ResourceObject;
import org.apache.log4j.Logger;

Logger log = Logger.getLogger("com.company.rules.ADCustomization");

String attrName = "msDS-cloudExtensionAttribute10";
String value = (String) object.getAttribute(attrName);

if (value != null && value.length() > 255) {
    log.warn("Truncating " + attrName + " for account: " + 
             object.getIdentity() + " from length " + value.length());
    object.setAttribute(attrName, value.substring(0, 255));
}

return object;
```

**Step 4 — The 356 missing accounts risk:**

This is the most important step and the one most teams miss. The 356 accounts have been missing from IIQ for 2 weeks. During this time:

If IIQ uses sAMAccountName for correlation: when these accounts successfully aggregate, IIQ finds existing identities with matching sAMAccountNames and updates them. Safe — no duplicates.

If IIQ uses ObjectGUID for native identity AND sAMAccountName for correlation: IIQ creates a new Link object (because ObjectGUID from new domain doesn't match old domain GUID), then finds an existing identity via sAMAccountName correlation, and attaches the new link to the existing identity. This may create a duplicate link on the identity — old domain link (now stale) and new domain link. The old link may trigger false "orphaned account" certifications.

**Before running the fix aggregation:**

```
IIQ UI → Debug → Object Browser → Identity → 
Search for one of the 356 affected users →
Check: how many Links exist for this identity?
Check: what is the nativeIdentity value of the AD link?
Does it match the current ObjectGUID in the new domain?
```

---

### Round 4 — V (Verify Before Acting)

**DERIVE Round 4:**

"Before I implement the Customization Rule fix:

1. What is your confidence level on the Customization Rule approach for the attribute truncation? Is this documented IIQ behavior or are you reasoning from general IIQ architecture?

2. What specifically should I test in a non-production IIQ environment before running this in production?

3. What is the rollback plan if the Customization Rule causes unexpected behavior?

4. What monitoring should I have in place during the first post-fix aggregation run?"

**What Round 4 extracts:**

**Confidence assessment (model should flag):**

The Customization Rule mechanism for modifying ResourceObjects during aggregation is documented IIQ behavior — Compass documentation covers this. Moderate-high confidence.

The specific Beanshell syntax should be verified against your IIQ version's Beanshell API documentation on Compass before use. Low-medium confidence on exact syntax — test in non-production first.

The database column size warning is based on general knowledge of how IIQ stores attribute data — this should be verified by actually checking your database schema. High priority to verify before proceeding.

**The specific non-production test:**

```
Step 1: Apply Customization Rule in non-production IIQ
Step 2: Run aggregation against a test OU containing 5 of the 356 problem accounts
Step 3: Verify in sailpoint.log: no 'value exceeds maximum length' errors
Step 4: Verify in IIQ UI: all 5 accounts appear in application account list
Step 5: Verify identity correlation: no duplicate links created
Step 6: Verify truncated attribute value stored correctly on the Link object
Step 7: Check database: spt_link_attribute table for these 5 accounts - 
        value stored without database error
```

**Rollback plan:**

```
If Customization Rule causes unexpected behavior:
1. Remove rule from application (Application → Aggregation → Customization Rule → clear)
2. System returns to previous state immediately — no data migration needed
3. Rule removal takes effect on next aggregation run
4. Previous behavior (skip with error) resumes
```

**Monitoring during first production fix run:**

```bash
# Terminal 1: Watch sailpoint.log in real-time during aggregation
tail -f <IIQ_HOME>/logs/sailpoint.log | grep -E "ERROR|WARN|Skipping|truncat"

# Terminal 2: Watch aggregation statistics
# IIQ UI → Tasks → [Aggregation Task] → refresh every 2 minutes
# Watch: Accounts Scanned counter — should climb to 5,203 not stop at 4,847

# After completion:
# Check task result statistics:
# Accounts Scanned should be 5,203
# Accounts Created: expect ~0 (if correlation finds existing identities)
# Accounts Updated: expect ~356 (existing identities updated with new link)
# If Accounts Created shows ~356: correlation not working — investigate before certifications run
```

---

## Scenario 2: Provisioning Workflow Stuck in Pending State

### Full DERIVE Prompt (All Principles Applied Together):

"I am troubleshooting a provisioning issue in SailPoint IIQ 8.2. We have a joiner workflow that provisions access for new employees. The workflow: triggers on identity create event → runs approval → provisions to 5 applications → sends welcome email.

Observed behavior:
- Workflow triggers correctly for new employee hires
- Approval step completes — approvers receive and approve work items
- After approval, workflow shows status: Pending in the IIQ UI
- The 5 provisioning requests show status: Queued
- They have been in Queued state for 6 hours — normally provisioning completes in 10 minutes
- No error shown in workflow UI
- This started happening 3 days ago — worked fine before that
- 3 days ago we deployed a new version of a custom connector for one of the 5 applications
- The other 4 applications provision successfully when we test them individually

Do NOT tell me to check workflow configuration generally — the workflow worked before. Do NOT tell me to check provisioning configuration generally — 4 of 5 applications work.

You are a SailPoint IIQ integration engineer who has debugged stuck provisioning queues specifically. You know the difference between a provisioning request that is stuck because of a connector error versus one that is stuck because the provisioning engine itself is blocked.

Step 1: Walk through the specific mechanism by which a single failing connector can cause provisioning requests for other connectors to also queue indefinitely.

Step 2: Walk through what the Queued state means technically — is the request waiting for a thread, waiting for a connector response, waiting for a retry timer?

Step 3: Walk through where in IIQ logs I find the actual error for a provisioning request that shows no error in the UI.

Step 4: Give me the diagnostic sequence before any fix."

**What DERIVE extracts:**

**Step 1 — How one connector blocks others:**

IIQ's provisioning engine uses a thread pool for connector provisioning operations. By default, IIQ has a limited number of provisioning threads. If the new custom connector is hanging — making an outbound connection that never times out, waiting for a response that never comes — those threads are held indefinitely.

With a small thread pool (IIQ default is often 5-10 provisioning threads) and a hanging connector consuming threads, new provisioning requests for all connectors queue waiting for a free thread. This is why your 4 working applications are also stuck — not because they have errors, but because there are no threads available to execute them.

The specific mechanism: hanging connector call → thread held → thread pool exhausted → all provisioning requests queue → no apparent error because each request is technically "waiting for execution" not "failed."

**Step 2 — What Queued state means technically:**

In IIQ, a provisioning request in Queued state is waiting to be picked up by the PersistentRequestHandler — a background service that dequeues and executes provisioning operations. The request is persisted in the `spt_provisioning_transaction` table with status=Queued.

The PersistentRequestHandler polls this table and executes requests using threads from the provisioning thread pool. If threads are held by hanging operations, the handler picks up the request, attempts to assign it a thread, finds none available, and puts it back in Queued state.

This means: Queued for 6 hours with no error = thread pool exhaustion, not connector configuration error.

**Step 3 — Where the actual error lives:**

```bash
# The UI shows no error because the thread is hung, not errored
# The actual evidence is in:

# 1. sailpoint.log - look for the custom connector's operations
grep -i "customconnector\|[connector_class_name]" sailpoint.log | \
  tail -100

# 2. Thread dump - shows hung threads explicitly
# Take a thread dump of the IIQ application server JVM:
# If Tomcat: 
kill -3 [tomcat_pid]  # Thread dump goes to catalina.out

# OR via JVM tools:
jstack [pid] > threaddump_$(date +%Y%m%d_%H%M%S).txt

# In the thread dump, look for threads blocked in your connector code:
grep -A 20 "BLOCKED\|WAITING" threaddump.txt | \
  grep -A 20 "customconnector\|[your connector package]"

# 3. Check active provisioning transactions in database:
SELECT id, identity_name, plan, status, created, modified
FROM spt_provisioning_transaction  
WHERE status = 'Queued'
ORDER BY created ASC;
-- If rows have old created timestamps but recent modified: 
-- they are being picked up and re-queued repeatedly
-- confirming thread pool exhaustion
```

**Step 4 — Diagnostic sequence:**

```
Step 1: Take a thread dump immediately
        Look for: threads blocked in custom connector code
        Confirms: thread pool exhaustion hypothesis

Step 2: Check custom connector operation in isolation
        IIQ UI → Applications → [Custom Connector App] → 
        Test Connection
        If test hangs: connector has a connection/timeout issue
        If test succeeds: issue may be specific to provisioning operation

Step 3: Check connector timeout configuration
        Custom connectors should have explicit connection and read timeouts
        Without timeouts, a dropped connection hangs the thread indefinitely
        Check connector source code or configuration for timeout settings

Step 4: Temporarily disable the custom connector application from the workflow
        Test if provisioning for the other 4 applications proceeds
        If yes: confirms the custom connector is blocking the thread pool
        This is a safe diagnostic — does not affect production users
        just confirms the hypothesis before you touch the connector

Step 5: If thread pool exhaustion confirmed:
        Restart the IIQ application server (releases hung threads)
        Immediately disable or fix the custom connector
        Re-run provisioning for stuck identities
```

---

## Scenario 3: Access Certification Campaign Showing Wrong Entitlements

### Full DERIVE Prompt:

"I am troubleshooting an access certification issue in SailPoint IIQ 8.3. We ran a manager certification campaign. Certifiers are reporting that some direct reports show entitlements that the employee does not actually have in the source system.

Specific observations:
- Employee John Smith shows entitlement 'Finance-ReadOnly' on the Certification
- I checked John's account in the actual AD application: this group is NOT in his memberOf attribute
- The entitlement appears as 'Active' on his certification item
- John's last aggregation was 18 hours ago
- The entitlement 'Finance-ReadOnly' appears in IIQ's Link object for John under the AD application
- In the entitlement catalog, Finance-ReadOnly shows last aggregation 3 days ago

Do NOT tell me to re-aggregate. I need to understand the mechanism first.

You are a SailPoint IIQ consultant who has debugged phantom entitlements in certifications. You know the specific IIQ data model relationship between: the Link object, the entitlement catalog, the certification item, and the source system. Walk through the exact data flow that produces a certification item showing an entitlement that no longer exists in the source system. Then tell me which specific IIQ table or object contains the stale data."

**What DERIVE extracts:**

**The exact data flow producing phantom entitlements:**

IIQ maintains entitlement data in two separate places that can get out of sync:

**The Link object** — stored in `spt_link` and `spt_link_attribute` tables. This represents what IIQ saw during the last aggregation of that specific account. John's last aggregation was 18 hours ago. At that time, Finance-ReadOnly WAS in his memberOf. He was subsequently removed from the group in AD. IIQ does not know this yet — the Link still reflects the state from 18 hours ago.

**The entitlement catalog** — stored in `spt_managed_attribute` table. This is a separate catalog of known entitlements, aggregated from the application independently of individual account aggregations.

**The certification item** — generated from a snapshot of the Link object at the time the certification was launched. If the certification was launched AFTER John's last aggregation but BEFORE the next aggregation that would have removed Finance-ReadOnly, the certification snapshot captured the stale Link data.

**The critical insight:** re-aggregation of John's account will update his Link object. But if the certification was already generated from the snapshot, the certification item still shows Finance-ReadOnly even after re-aggregation. The certification and the live data are now permanently out of sync until the certification is closed and a new one generated.

**Which object contains the stale data:**

```sql
-- The certification item snapshot:
SELECT ci.id, ci.identity_name, ci.application, ci.attribute, ci.value, 
       ci.action, ci.status
FROM spt_certification_item ci
JOIN spt_certification c ON ci.certification_id = c.id
WHERE ci.identity_name = 'john.smith'
  AND ci.value = 'Finance-ReadOnly';
-- If this row exists: certification was generated with this entitlement
-- It will show in certification regardless of current account state

-- The current Link state (what IIQ currently believes):
SELECT la.attribute_name, la.attribute_value
FROM spt_link l
JOIN spt_link_attribute la ON l.id = la.link_id
JOIN spt_application a ON l.application_id = a.id
WHERE l.identity_id = (SELECT id FROM spt_identity WHERE name = 'john.smith')
  AND a.name = '[Your AD Application Name]'
  AND la.attribute_name = 'memberOf'
  AND la.attribute_value LIKE '%Finance-ReadOnly%';
-- If NO rows: Link is already updated or was never updated
-- If rows exist: Link still shows the entitlement (aggregation not yet run)
```

**The resolution decision:**

If John genuinely does not have Finance-ReadOnly in AD: the certifier should Revoke it in the certification. IIQ will attempt to provision the revocation. The provisioning will attempt to remove Finance-ReadOnly from his AD account. Since he does not have it, the provisioning should either succeed silently (no-op) or fail with "attribute value not found." Either way the certification item is resolved.

What you should NOT do: re-aggregate John and then tell certifiers to trust the updated data — because the certification snapshot is immutable. The certifier still sees Finance-ReadOnly. You need a process for handling stale certification items, which in IIQ means: document the out-of-sync condition, have certifiers revoke phantom entitlements with a comment explaining the situation, and implement more frequent aggregation before future certification launches.

---

## The DERIVE Loop for IIQ Troubleshooting — The Meta-Pattern

Every IIQ production issue follows the same five-round structure with one critical addition specific to IIQ:

**Round 1 (D):** Describe the exact observed behavior with specific numbers — accounts scanned counts, workflow status values, exact log message text. The generic IIQ problem has a dozen possible causes. Your specific numbers eliminate most of them.

**Round 2 (E + R):** Paste the actual log messages. Do not paraphrase them. The exact exception class name, the exact error message text, these are what allow the model to pattern-match against real IIQ knowledge rather than generate plausible-sounding IIQ language.

**Round 3 (I):** Force the model to walk through the IIQ data model before recommending a fix. IIQ has many interconnected objects (Identity, Link, ManagedAttribute, ProvisioningTransaction, CertificationItem) and a fix that solves one object's stale data while ignoring another causes new problems.

**Round 4 (V):** This is mandatory in IIQ, not optional. Every specific claim about IIQ behavior must be verified against Compass documentation or tested in non-production before production action. The model's IIQ knowledge is real but patchy and version-sensitive.

**Round 5 (E — Expand):** Every IIQ issue has a downstream consequence you discover after the fix. The aggregation fix creates a duplicate link question. The provisioning fix reveals a thread pool sizing question. The certification fix reveals a process question about certification timing relative to aggregation. Round 5 surfaces these before they become the next incident.

**The IIQ-specific addition — always ask about data model implications before fixing:**

IIQ's data is highly interconnected. A fix that updates one table without updating the related table creates a new inconsistency. Before any fix, always ask: "Which IIQ tables or objects contain this data? If I fix the primary object, which related objects will now be out of sync?"

The production knowledge you need for IIQ troubleshooting is partially in the LLM and partially in SailPoint Compass, your specific logs, and your specific database. DERIVE gets you the diagnostic framework and the IIQ data model reasoning. Your logs and Compass provide the verification layer that makes the diagnosis safe to act on.


---------

Before I answer, let me be honest about what "improve" means here.

There are two completely different things you might mean:

**Meaning 1:** How do I get better at applying DERIVE to extract better responses from LLMs for IIQ troubleshooting?

**Meaning 2:** How do I improve my actual IIQ troubleshooting capability beyond what any LLM framework can provide?

The honest answer is that if you only pursue Meaning 1, you will hit a ceiling quickly. Because the fundamental limitation we identified in the previous response is real — IIQ knowledge in LLM training data is sparse, proprietary, and version-sensitive. Better prompting extracts better knowledge from a limited pool. It does not expand the pool.

The real improvement path requires pursuing both simultaneously. Let me show you exactly how.

---

## The Honest Diagnosis of the Current Approach

When you applied DERIVE to IIQ troubleshooting in the previous response, three things were happening simultaneously that you need to separate:

**Thing 1 — DERIVE was helping you think more clearly**
The act of constructing a DERIVE prompt forced you to organize your evidence, state what you had already ruled out, and identify the specific mechanism you did not understand. This is valuable regardless of whether the LLM gives you a good answer. The diagnostic discipline itself is the value.

**Thing 2 — DERIVE was reaching real IIQ knowledge**
The aggregation skip mechanism, the HikariCP thread pool interaction with provisioning, the certification snapshot data model — these are documented patterns that exist in Compass, in public blog posts, in conference presentations. The LLM reached this knowledge better with DERIVE than without it.

**Thing 3 — DERIVE was generating plausible IIQ-sounding responses**
Some specific SQL queries, some specific Beanshell syntax, some specific configuration property names — these may be partially fabricated. They sound right because the model knows the IIQ data model well enough to generate plausible-looking specifics. The risk is acting on fabricated specifics without verification.

Improving means: amplify Thing 1 and Thing 2, eliminate the damage from Thing 3.

---

## Layer 1: Improving How You Apply DERIVE to IIQ

### Improvement 1 — Feed the LLM Your Actual Artifacts, Not Descriptions of Them

The single biggest improvement you can make immediately.

**Current approach:**
"I have a provisioning workflow that triggers on identity create. After approval, requests are stuck in Queued state."

**Improved approach:**
Paste the actual workflow XML, the actual log excerpt, the actual task result XML, the actual database query output. Not descriptions. The actual content.

IIQ stores almost everything as XML in the database. You can export these artifacts directly:

```sql
-- Export the stuck provisioning transaction details
SELECT id, identity_name, status, plan, result, created, modified
FROM spt_provisioning_transaction
WHERE status = 'Queued'
ORDER BY created ASC
LIMIT 10;

-- Export the workflow definition
SELECT name, type, description
FROM spt_process  
WHERE name = '[Your Workflow Name]';
```

When you paste actual IIQ XML into a DERIVE prompt, the model is working with real data instead of your description of real data. Your description introduces your interpretation and your blind spots. The raw XML does not.

**Why this works at the fundamental level:**

Truth 2 says the model fills unknowns with averages. Every word in your description that is not the actual artifact is an unknown the model fills with its average assumption about IIQ. Paste the actual artifact and those unknowns disappear.

---

### Improvement 2 — Build a Personal IIQ Evidence Library

The model's IIQ training data is sparse. You can partially compensate by becoming the source of IIQ-specific knowledge in the context window.

Over time, build a personal document containing:

**Your environment's specific patterns:**
```
Our IIQ version: 8.3 patch 4
Our AD connector version: 8.3.0.2
Our correlation configuration: sAMAccountName primary, email secondary
Our provisioning thread pool size: [check yours]
Our custom rules: [list names and purposes]
Our known issues: [document issues you have encountered and resolved]
```

**Real log patterns from your environment:**
When you solve an IIQ issue, copy the exact log pattern that identified the root cause into this document. When you have a new issue, paste the relevant section of your evidence library into your DERIVE prompt.

```
KNOWN PATTERN - AD Aggregation Skip:
Symptom: Accounts missing from aggregation statistics, no error in task result
Log signature: "WARN Aggregator - Failed to process ResourceObject"
Root cause categories: schema attribute length, correlation collision, permission error
Resolution history: [document what you did]
```

**Why this works:**

You are building the IIQ-specific long-tail knowledge that is missing from training data and injecting it directly into the context window. Truth 5 says the context window is the only interface. You are populating it with real, verified, environment-specific knowledge that the model cannot hallucinate because you are providing it.

---

### Improvement 3 — Use DERIVE to Generate Diagnostic Hypotheses, Not Answers

This is a subtle but important shift in how you use the framework.

**Current mindset:** Use DERIVE to get the answer to the IIQ problem.

**Improved mindset:** Use DERIVE to generate a ranked list of hypotheses, then verify each hypothesis yourself using IIQ's built-in diagnostic tools.

**The improved DERIVE prompt structure for IIQ:**

```
[Your specific symptom with evidence]

Do not give me a solution. Give me:
1. The 5 most likely root causes for this specific symptom in IIQ, 
   ranked by probability given my evidence
2. For each root cause: the specific IIQ diagnostic step that would 
   confirm or eliminate it (specific UI navigation, specific log grep, 
   specific database query)
3. The specific order I should check them in (fastest to verify first)
4. Flag which of your hypotheses you are uncertain about and why
```

This approach uses the LLM for what it is reliable at (generating a hypothesis space, structuring diagnostic logic) and reserves the verification work for you using real IIQ data. The LLM's hypothesis may be wrong. Your verification against real data cannot be wrong.

---

### Improvement 4 — The Differential Diagnosis Pattern

Borrowed from medicine. Before DERIVE became a framework, the best doctors were already applying its principles intuitively. The differential diagnosis is the medical version of DERIVE's I principle applied specifically to troubleshooting.

**Apply it to IIQ like this:**

After your Round 1 DERIVE prompt gets you a list of possible causes, send this Round 2 prompt:

"You have given me 4 possible causes. For each one: what is the single fastest diagnostic test that would definitively include or exclude it? Order the tests so the one that eliminates the most hypotheses simultaneously comes first. The goal is to reach a confirmed root cause in the minimum number of diagnostic steps."

**Why this is an improvement:**

Without this, you might spend 2 hours investigating the least likely cause because the LLM listed it first. The differential diagnosis pattern forces prioritization by diagnostic efficiency, not by the order the LLM happened to generate hypotheses.

---

## Layer 2: Building Real IIQ Expertise That Goes Beyond LLM

This is where the genuine long-term improvement happens. DERIVE is a tool for extracting knowledge. Building your own knowledge makes you less dependent on the tool.

### Building Block 1 — Learn the IIQ Data Model Deeply

Every IIQ troubleshooting problem eventually comes down to: which tables contain the stale, incorrect, or missing data? The engineer who knows the IIQ data model can look at a symptom and immediately know which database table to query to confirm the hypothesis.

The most important IIQ tables to know:

```sql
-- Identity and account data
spt_identity              -- Core identity attributes
spt_link                  -- Application account (one row per account per identity)
spt_link_attribute        -- Account attribute values (memberOf, etc.)
spt_identity_entitlement  -- IIQ's view of what entitlements identity has

-- Provisioning
spt_provisioning_transaction  -- Provisioning requests and their status
spt_identity_request          -- Access requests (LCM)
spt_identity_request_item     -- Individual items within a request

-- Certifications
spt_certification             -- Certification campaign metadata
spt_certification_entity      -- Each identity in a certification
spt_certification_item        -- Each entitlement/role in a certification

-- Workflow
spt_process                   -- Workflow definitions
spt_process_log               -- Workflow execution history (your best friend)
spt_work_item                 -- Approval work items

-- Entitlement catalog
spt_managed_attribute         -- Known entitlements/groups
spt_managed_attribute_value   -- Values associated with managed attributes

-- Roles and access
spt_bundle                    -- Roles (IT roles and business roles)
spt_profile                   -- Role profiles (what entitlements a role grants)
```

**The practical exercise that builds this knowledge:**

Pick one IIQ issue you have already solved. After solving it, write down: which tables contained the evidence that identified the root cause? Which tables were modified by the fix? This post-mortem practice builds a mental map of the data model through real experience, not through reading documentation.

---

### Building Block 2 — Master IIQ's Built-In Diagnostic Tools

Most IIQ engineers underuse these. They are more reliable than any LLM output.

**The IIQ Debug page:**

```
IIQ UI → Debug → Object Browser

This lets you inspect any IIQ object (Identity, Application, 
Certification, Workflow Instance) in its raw XML form.

When troubleshooting:
- Inspect a stuck workflow instance: see its exact current state, 
  which step it is on, what variables it holds
- Inspect an identity: see all links, all roles, all entitlements 
  as IIQ currently knows them
- Inspect a provisioning plan: see exactly what IIQ is trying to provision
```

**The Process Instrumentation Log:**

```
IIQ UI → Setup → System Configuration → 
Enable Workflow Logging → set to DEBUG for specific workflows

This produces a step-by-step log of every workflow execution:
- Which step executed
- What variables were set at each step
- What approvals were generated
- What provisioning was attempted
- Where the workflow stopped and why

This is the single most useful IIQ diagnostic tool and 
most engineers do not know it exists or how to read it.
```

**Log4j configuration for targeted debugging:**

```xml
<!-- In log4j.properties or log4j2.xml -->
<!-- Enable DEBUG for specific IIQ components without flooding the log -->

<!-- For aggregation debugging: -->
log4j.logger.sailpoint.connector=DEBUG
log4j.logger.sailpoint.reporting.datasource.AggregationDataSource=DEBUG

<!-- For provisioning debugging: -->
log4j.logger.sailpoint.provisioning=DEBUG
log4j.logger.sailpoint.connector.ConnectorFactory=DEBUG

<!-- For workflow debugging: -->
log4j.logger.sailpoint.workflow=DEBUG
log4j.logger.sailpoint.server.TaskManager=DEBUG

<!-- For certification debugging: -->
log4j.logger.sailpoint.certification=DEBUG
```

The key discipline: enable targeted debug logging for the specific component you are investigating, reproduce the issue, capture the log, then disable debug logging. Never leave debug logging enabled in production — it floods the log and can cause performance issues.

---

### Building Block 3 — Build a Personal IIQ Runbook

Every time you solve a production IIQ issue, document it in a personal runbook with this structure:

```
ISSUE: [Symptom description]
IIQ VERSION: [specific version]
ENVIRONMENT: [connector type, configuration specifics]

INITIAL EVIDENCE:
- [What the UI showed]
- [What the logs showed]
- [What the database showed]

HYPOTHESES CONSIDERED:
1. [Hypothesis] → ELIMINATED by [specific check]
2. [Hypothesis] → ELIMINATED by [specific check]
3. [Hypothesis] → CONFIRMED by [specific check]

ROOT CAUSE:
[Specific mechanism explanation]

FIX APPLIED:
[Specific steps taken]

VERIFICATION:
[How you confirmed the fix worked]

PREVENTION:
[What monitoring or configuration change prevents recurrence]

TIME TO RESOLVE: [how long this took]
NEXT TIME: [what you would check first next time]
```

After 20 of these entries, you have something more valuable than any LLM can provide: a searchable knowledge base of issues you have personally verified in your specific environment. When a new issue appears, you search your runbook first. The LLM is the fallback, not the primary resource.

---

### Building Block 4 — Understand the IIQ Threading and Execution Model

This is the knowledge that unlocks the hardest production issues because most of them are concurrency and resource exhaustion problems, not configuration problems.

**The key threads to understand:**

```
Request Thread Pool (Tomcat)
├── Handles HTTP requests from users
├── Default: 200 threads
└── Held during synchronous operations including some IIQ operations

Quartz Scheduler Threads
├── Executes scheduled tasks (aggregation, certification generation)
├── Configurable pool size
└── Task starvation occurs when pool exhausted by long-running tasks

Provisioning Threads
├── Executes connector provisioning operations
├── Default pool: small (varies by version)
└── Thread exhaustion causes provisioning queue buildup

Request Handler Threads (PersistentRequestHandler)
├── Processes items in spt_request table
├── Background service
└── Failure here causes silent queue buildup with no UI error
```

**Why this matters for DERIVE:**

When you construct a DERIVE prompt for a production IIQ issue, knowing the threading model lets you immediately ask about the right component. Instead of "my provisioning is stuck," you can ask "my provisioning thread pool appears exhausted based on these symptoms — what specific IIQ configuration controls the provisioning thread pool size and where does that configuration live?"

That specificity activates better LLM responses because it targets the narrow portion of IIQ knowledge where the answer lives.

---

### Building Block 5 — Build a Relationship With SailPoint Compass

The single most important non-LLM resource. SailPoint Compass is the official community and documentation platform. It contains:

- Official product documentation for every IIQ version
- Known issues database with version-specific bug reports
- Community posts from IIQ engineers at other companies
- SailPoint employee responses to common issues
- Source code access for authorized partners

**How to use Compass to verify LLM responses:**

Every specific claim the LLM makes about IIQ behavior should be verifiable on Compass. If you cannot find it on Compass, treat it as potentially hallucinated.

**The search pattern that finds the most useful Compass content:**

Instead of searching for the problem, search for the error message. The exact exception class name from your log is a better Compass search term than a description of your problem. Other engineers paste their exact error messages into Compass posts. Matching your exact error to their exact error is more reliable than matching your symptom description to their symptom description.

---

## Layer 3: Building a System That Gets Better Over Time

This is the layer that most engineers never reach because they are always in reactive troubleshooting mode. The highest leverage improvement is building a system that prevents issues from becoming incidents and makes future troubleshooting faster.

### System Improvement 1 — Proactive Monitoring That Catches IIQ Issues Before Users Report Them

Most IIQ environments are monitored reactively — someone calls the help desk, you log in and investigate. The following monitoring surfaces issues before they become user-visible:

**Aggregation health monitoring:**

```sql
-- Run this as a scheduled report or monitoring query
-- Flags aggregations that completed with significantly fewer accounts than last run
-- Catches silent skip issues before they affect certifications

SELECT 
    t.name as task_name,
    t.completed,
    t.messages,
    -- Extract accounts scanned from task result
    -- (actual query depends on IIQ version and result structure)
    ABS(current_scan_count - previous_scan_count) as account_delta
FROM spt_task_result t
WHERE t.type = 'Aggregation'
  AND t.completed > NOW() - INTERVAL '25 hours'
  AND ABS(current_scan_count - previous_scan_count) > 
      (previous_scan_count * 0.05)  -- Alert if >5% change
ORDER BY account_delta DESC;
```

**Provisioning queue depth monitoring:**

```sql
-- Alert if provisioning queue depth exceeds threshold
-- Catches thread exhaustion before it becomes a user-visible backlog

SELECT 
    COUNT(*) as queued_count,
    MIN(created) as oldest_queued,
    DATEDIFF(NOW(), MIN(created)) as hours_in_queue
FROM spt_provisioning_transaction
WHERE status = 'Queued';

-- Alert threshold: queued_count > 10 OR hours_in_queue > 1
```

**Workflow stuck detection:**

```sql
-- Detects workflows that have been running longer than expected
-- Catches hung threads before they affect the whole provisioning queue

SELECT 
    pi.id,
    pi.name,
    pi.launched,
    DATEDIFF(NOW(), pi.launched) as hours_running
FROM spt_process_log pi
WHERE pi.completed IS NULL
  AND pi.launched < NOW() - INTERVAL '2 hours';

-- Any workflow running for more than 2 hours without completing
-- warrants investigation
```

**Why this matters for DERIVE:**

When you have proactive monitoring, you catch issues during off-hours before users are affected. You have more time to construct careful DERIVE prompts and verify responses before acting. Reactive troubleshooting under pressure produces worse DERIVE prompts and less careful verification. The monitoring buys you the time that good diagnostic practice requires.

---

### System Improvement 2 — A Pre-Deployment Checklist That Prevents the Most Common IIQ Issues

Looking at the three scenarios in the previous response, all three were caused by changes:

Scenario 1 — AD domain migration caused schema mismatch
Scenario 2 — New custom connector deployment caused thread exhaustion
Scenario 3 — Certification launched without fresh aggregation

A pre-deployment checklist that prevents these specifically:

```
BEFORE deploying a new or modified connector:
□ Does the connector have explicit connection timeouts configured?
  (No timeout = potential thread exhaustion)
□ Does the connector have explicit read timeouts configured?
□ Has the connector been load tested against the target system?
□ What is the connector's behavior when the target system is slow?
□ Does the connector handle partial failures gracefully?

BEFORE running a certification campaign:
□ When did the last aggregation run for each application in scope?
□ Is the aggregation count within 2% of last run? (flags silent skips)
□ Are there any pending identity changes that should be reflected first?
□ Is there a pending schema change that could affect entitlement visibility?

BEFORE making changes to application schema:
□ What is the current column size in spt_link_attribute for this attribute?
□ Does the new maxLength exceed the database column size?
□ What IIQ configurations (roles, policies, certifications, reports) 
  reference this attribute?
□ Has the change been tested in non-production with real data?
```

Each checklist item directly corresponds to a class of production issue. The checklist is derived from your troubleshooting history. The more issues you document in your runbook, the more specific your checklist becomes.

---

### System Improvement 3 — The Feedback Loop That Compounds Over Time

This is the improvement that makes every subsequent troubleshooting session faster than the last one.

**The loop:**

```
Production Issue Occurs
        ↓
Apply DERIVE to generate hypotheses
        ↓
Verify hypotheses using IIQ diagnostic tools
        ↓
Identify root cause
        ↓
[Document in personal runbook]  ← most engineers skip this
        ↓
[Update DERIVE evidence library] ← almost no engineers do this
        ↓
[Add prevention check to deployment checklist] ← rare
        ↓
Next similar issue: check runbook first
        → Found in runbook: 10-minute resolution
        → Not found: apply DERIVE with richer evidence library
```

The compounding effect: your first IIQ production issue takes 4 hours to resolve. Your tenth similar issue takes 20 minutes. Not because you are using DERIVE better — because your runbook has 9 prior entries that make the hypothesis space immediate.

The LLM cannot compound this way. Every conversation starts fresh. Your runbook does not.

---

## The Honest Improvement Priority Order

If you want to genuinely improve IIQ troubleshooting capability, here is the order that produces the most impact per unit of effort:

**Highest impact, immediate:**
Start your personal runbook today. Document the next IIQ issue you solve. The discipline of writing it down is itself the improvement — it forces you to understand the root cause deeply enough to explain it, which is different from having fixed it.

**High impact, builds over weeks:**
Learn the 15 most important IIQ database tables and what they contain. Not memorization — understanding. When you see a symptom, you should immediately know which table to query to confirm or eliminate the hypothesis.

**High impact, medium term:**
Build your DERIVE evidence library. After every solved issue, add the log signature, the root cause, and the resolution to a document you paste into future DERIVE prompts.

**Medium impact, ongoing:**
Improve your DERIVE prompts by pasting actual IIQ artifacts (log excerpts, XML, SQL output) instead of descriptions of them. This is a habit change, not a learning investment.

**Medium impact, prevents future issues:**
Build the pre-deployment checklist from your documented issues. Each entry on the checklist is a future incident that will not happen.

**Lower impact but important:**
Engage with SailPoint Compass regularly — not just when you have a problem. Reading how other engineers solved IIQ issues builds the pattern recognition that makes your DERIVE prompts more targeted.

---

## The Single Most Important Insight

DERIVE is a framework for extracting knowledge from an LLM. It works best when the knowledge exists in the LLM's training data and needs to be reached with specificity.

For IIQ, a significant portion of the knowledge you need does not exist in the LLM's training data in a reliable form. It exists in three places: SailPoint Compass, your production logs, and your personal experience.

The genuine improvement path is to build a system where your personal experience and your production log evidence flow into every DERIVE prompt — making you the knowledge source that the LLM organizes and structures, rather than making the LLM the knowledge source you are trying to extract from.

When you reach that point, the combination of your IIQ-specific knowledge and the LLM's diagnostic reasoning and structuring capability is more powerful than either alone. That is the ceiling DERIVE can actually reach — and it is significantly higher than what most IIQ engineers achieve with any approach.


-------

Let me be upfront about something critical before we start.

---

## The Honest Assessment First

SailPoint IIQ workflows are built on:

**Beanshell** — a Java-like scripting language that is itself sparsely documented outside of IIQ context. Training data is limited.

**IIQ's internal Java API** — proprietary classes like `SailPointContext`, `WorkflowContext`, `ProvisioningPlan`, `Identity`. These are documented on Compass but the documentation is sparse in public training data.

**XML workflow definitions** — IIQ stores workflows as XML in the database. The schema is proprietary.

**Version-specific behavior** — workflow execution changed significantly between IIQ 7.x, 8.0, 8.1, 8.2, 8.3.

This means:

The LLM has good knowledge of: workflow concepts, general architecture, publicly documented patterns, common workflow structures shared in public blogs and Compass posts.

The LLM has moderate knowledge of: Beanshell patterns for common IIQ operations, standard workflow variables, provisioning plan construction.

The LLM has high hallucination risk for: specific API method signatures, version-specific behavior, edge cases in workflow execution, your specific environment's custom configurations.

Every code example in this response should be treated as a starting point verified against your IIQ version's Compass documentation before production use.

With that said, let me build the deepest possible understanding of IIQ workflows using DERIVE.

---

## Part 1: Understanding How IIQ Workflows Actually Work

Before building anything, you need a precise mental model. Most IIQ engineers have a surface model that breaks when they try to build complex workflows.

### The Mental Model Most Engineers Have (And Why It Is Wrong)

Most engineers think of an IIQ workflow like this:

```
Trigger → Step 1 → Step 2 → Step 3 → Done
```

Linear. Sequential. Like a flowchart. This mental model explains simple workflows but breaks completely when you encounter: approval escalation, subprocess calls, background execution, waitFor conditions, and variable scoping. Let me replace this with the correct mental model.

---

### The Correct Mental Model

An IIQ workflow is a **persistent state machine** stored in the database.

Every word in that phrase matters:

**Persistent** — the workflow's entire state (all variables, current step, pending approvals, execution history) is serialized to XML and stored in `spt_process_log` in the database after every step. If the IIQ server restarts mid-workflow, the workflow resumes exactly where it left off from the database state.

**State machine** — the workflow moves between states (steps) based on transitions. Each transition has a condition. The workflow engine evaluates conditions to determine which step to execute next.

**Stored in the database** — the workflow definition is XML in `spt_process`. The workflow instance (currently executing workflow) is XML in `spt_process_log`. Both are inspectable in real-time.

**Why this matters for troubleshooting:**

When a workflow is "stuck," it is sitting in a specific state in `spt_process_log` waiting for a condition to become true. The condition might be: waiting for an approver to respond, waiting for a background step to complete, waiting for a manual work item to be resolved. You can read the exact state from the database.

```sql
-- See every currently running workflow instance and exactly 
-- what step it is waiting on
SELECT 
    pl.id,
    pl.process_name,
    pl.case_name,
    pl.launched,
    pl.current_step,
    pl.pending_actions,
    pl.variables  -- This XML contains ALL workflow variables current values
FROM spt_process_log pl
WHERE pl.completed IS NULL
ORDER BY pl.launched ASC;
```

---

### The Four Fundamental Workflow Concepts

Before building anything, these four concepts must be precise in your mind:

**Concept 1 — The Workflow Case**

Every executing workflow is a "case." The case has:
- A unique ID
- A name (usually derived from the identity being processed)
- A set of variables (the workflow's memory)
- A current step (where execution is right now)
- A case owner (who is responsible for the workflow)

The case is the unit of workflow execution. When you look at a stuck workflow in the UI, you are looking at a case.

**Concept 2 — Variables and Scope**

Workflow variables are like a shared memory space for the entire workflow execution. They persist across steps. Any step can read and write variables. This is both powerful and dangerous — a variable set in Step 2 can be read in Step 15, but if Step 2 sets the wrong value, Step 15 does the wrong thing and you may not discover the error until then.

The critical variable scoping rule: variables defined in the workflow definition are workflow-level variables. Variables defined in a step's local scope disappear when the step completes. Most Beanshell bugs in IIQ workflows are caused by engineers assuming a locally-scoped variable is workflow-scoped.

**Concept 3 — Transitions**

Transitions are the arrows between steps. Every step can have multiple transitions, each with a condition. The workflow engine evaluates transitions in order and follows the first one whose condition is true.

The default transition (no condition) is always followed if no conditional transition matches. If you forget a default transition, your workflow hangs with no error because no transition is valid.

This is one of the most common workflow development mistakes. Always add a default transition as a safety net.

**Concept 4 — The Approval and Work Item Model**

When a workflow generates an approval, it creates a `WorkItem` in the database and then enters a waiting state. The workflow does not block a thread while waiting — it serializes its state to the database and releases all resources. This is why IIQ can have thousands of simultaneous approvals without exhausting its thread pool.

When the approver completes the work item, the workflow engine wakes the corresponding workflow case from the database, loads its state, and continues execution from the step after the approval.

---

### DERIVE Applied to Understanding the Execution Model

**Round 1 — D (Describe current mental model and expose where it breaks)**

"I am learning IIQ workflow internals to build custom workflows. My current mental model:

A workflow is a sequence of steps defined in XML. When triggered, IIQ executes steps in order. Each step can run Beanshell code, call a subprocess, or generate an approval. The workflow maintains variables that persist between steps.

Where my mental model breaks:

1. I do not understand what actually happens at the thread level when a workflow hits an approval step. Does a thread sit and wait for the approver? If not, how does the workflow resume when the approver responds — how does IIQ know which workflow to wake up?

2. I have seen workflows with 'background' attribute set to true on steps. I do not understand what this means mechanically — is it a different thread? A different JVM? What is the difference in execution context?

3. I do not understand the relationship between the workflow definition (the XML template) and a running workflow instance. Are they the same object? Can I change the definition while instances are running?

4. I have seen variables passed into workflows via the 'launcher' and via 'input' arguments but I cannot explain the difference between them or when to use each.

For each: correct my mental model with the exact mechanism. Then give me one diagnostic SQL query or IIQ debug action that makes the mechanism visible to me directly."

**What DERIVE extracts:**

**Concept 1 — Thread behavior at approval steps:**

When a workflow hits an approval step, this exact sequence occurs:

```
Step N (pre-approval processing) completes
↓
Workflow engine creates WorkItem records in spt_work_item
↓
Workflow engine serializes entire workflow state to XML
↓
XML written to spt_process_log (current_step updated)
↓
Thread is RELEASED back to the thread pool
↓
[No thread is held. Zero threads consumed while waiting for approval.]
↓
Approver logs in, opens work item, clicks Approve
↓
HTTP request handled by a NEW thread
↓
That thread reads the completed work item
↓
Thread queries spt_process_log for workflows waiting on this work item
↓
Thread deserializes the workflow state from XML
↓
Thread continues workflow execution from the next step
↓
Thread released when workflow hits next wait point or completes
```

The diagnostic that makes this visible:

```sql
-- Find the work item associated with a stuck workflow approval
-- and see exactly which workflow case it belongs to
SELECT 
    wi.id as work_item_id,
    wi.name,
    wi.owner,
    wi.type,
    wi.created,
    wi.target_class,
    wi.target_id  -- This links back to the process_log entry
FROM spt_work_item wi
WHERE wi.completed IS NULL
  AND wi.type = 'Approval'
ORDER BY wi.created ASC;
```

**Concept 2 — Background steps:**

A background step does NOT mean a different thread in the traditional sense. In IIQ, marking a step as background means: when the workflow reaches this step, instead of executing it immediately in the current thread, it creates a `Request` object in `spt_request` and releases the current thread. The `PersistentRequestHandler` (a separate background service) picks up the request later and executes the step in its own thread.

Why would you use this? For long-running connector operations. If a provisioning step to a slow application takes 5 minutes, executing it in the current thread holds resources. Background execution releases the thread immediately and lets the slow operation run in the background service.

The risk: background steps execute in a different security context with no session. Any IIQ operation that requires a session (like loading full identity objects with lazy-loaded attributes) may fail in background context.

```sql
-- See pending background step executions
SELECT 
    r.id,
    r.name,
    r.type,
    r.phase,
    r.next_launch,
    r.completed
FROM spt_request r
WHERE r.completed IS NULL
ORDER BY r.next_launch ASC;
```

**Concept 3 — Definition vs instance:**

The workflow definition in `spt_process` is a template. Each running workflow instance in `spt_process_log` contains a copy of the definition XML at the time the workflow was launched, embedded in the instance. This means:

If you change the workflow definition while instances are running, running instances continue using the OLD definition embedded in their instance record. New launches use the NEW definition. You can have simultaneously running workflows with different versions of the same definition.

This has a critical implication: fixing a bug in a workflow definition does not fix already-running instances. Stuck or broken instances may need to be manually completed or terminated and restarted.

**Concept 4 — Launcher vs input arguments:**

```xml
<!-- The workflow definition shows the difference -->
<Workflow name="MyWorkflow">
  
  <!-- Input arguments: explicitly declared parameters -->
  <!-- These appear in the workflow UI when you manually launch -->
  <!-- They are strongly typed and documented -->
  <Variable name="identityName" input="true">
    <Description>The identity to process</Description>
  </Variable>
  
  <!-- Launcher: implicit context variable -->
  <!-- Contains the WorkflowLaunchParameters object -->
  <!-- Available in every workflow without declaration -->
  <!-- Contains: who launched it, when, with what parameters -->
  
</Workflow>
```

The practical difference: `identityName` as an input variable is the explicit, documented, typed interface of your workflow. The launcher provides the execution context (who launched it, from what trigger). Use input variables for all data your workflow needs to operate. Use the launcher only for execution metadata.

---

## Part 2: The IIQ Workflow Anatomy — Every Component Explained

Now let's dissect a complete workflow definition so every XML element is understood precisely.

### The Complete Anatomy of a Workflow Step

```xml
<Step icon="Default" name="Process Identity" posX="100" posY="100">
  
  <!-- PART 1: SCRIPT -->
  <!-- Beanshell code that executes when this step runs -->
  <!-- 'wfc' is the WorkflowContext - your gateway to everything -->
  <Script>
    <Source>
      import sailpoint.object.Identity;
      import sailpoint.object.ProvisioningPlan;
      import org.apache.log4j.Logger;
      
      Logger log = Logger.getLogger("com.company.workflows.MyWorkflow");
      
      // Get workflow variables using wfc
      String identityName = (String) wfc.get("identityName");
      
      // Get the SailPointContext for database operations
      // 'context' is automatically available in workflow scripts
      Identity identity = context.getObjectByName(
          Identity.class, identityName
      );
      
      if (identity == null) {
          log.error("Identity not found: " + identityName);
          wfc.put("errorMessage", "Identity not found: " + identityName);
          wfc.put("hasError", Boolean.TRUE);
      } else {
          log.info("Processing identity: " + identity.getDisplayName());
          wfc.put("identity", identity);
          wfc.put("hasError", Boolean.FALSE);
      }
      
      // Return value of script becomes step's 'return' variable
      return identity;
    </Source>
  </Script>
  
  <!-- PART 2: RETURN -->
  <!-- Maps the script return value to a workflow variable -->
  <Return merge="false" name="identity" to="identity"/>
  
  <!-- PART 3: TRANSITIONS -->
  <!-- Evaluated in order, first matching condition is followed -->
  <Transition to="Handle Error" when="script:(Boolean)wfc.get(&quot;hasError&quot;)"/>
  <Transition to="Build Provisioning Plan"/>  <!-- Default: no condition -->
  
</Step>
```

**Every element you need to understand:**

`posX/posY` — Visual position in the workflow designer. Purely cosmetic. Has no effect on execution.

`Script/Source` — Beanshell code. Executes synchronously in the current thread unless the step has `background="true"`. The implicit variables available in every script: `wfc` (WorkflowContext), `context` (SailPointContext), `workflow` (the Workflow object itself).

`Return` — Maps the script's return value to a named workflow variable. `merge="false"` replaces the variable's value. `merge="true"` merges (used for maps and lists).

`Transition when` — A Beanshell expression that returns boolean. The `&quot;` is XML-escaped double quote. When the condition is true, the workflow follows this transition. Order matters — first true condition wins.

`Transition to` — The name of the step to execute next. Must exactly match a step name in the workflow.

---

### The Approval Step Anatomy

The approval step is the most complex and most important to understand precisely:

```xml
<Step icon="Approval" name="Manager Approval" 
      type="approval" 
      posX="300" posY="100">
  
  <!-- PART 1: APPROVAL DEFINITION -->
  <Approval mode="serial" owner="script:getApprovalOwner(wfc)">
    
    <!-- The form shown to the approver -->
    <Form>
      <Section>
        <Field name="comments" type="string" 
               displayName="Approval Comments"
               required="false"/>
      </Section>
    </Form>
    
    <!-- What the approver can do -->
    <!-- These become buttons in the work item UI -->
    <ApprovalSet>
      <ApprovalItem 
          id="script:wfc.get(&quot;identityName&quot;)"
          displayName="script:getDisplayName(wfc)"
          value="Approve"/>
    </ApprovalSet>
    
  </Approval>
  
  <!-- PART 2: APPROVAL ACTION (what happens after decision) -->
  <Arg name="approvalSummary" value="ref:approvalSummary"/>
  
  <!-- PART 3: TRANSITIONS -->
  <!-- wfc.getApprovalSet().isApproved() - true if all items approved -->
  <Transition to="Provision" 
              when="script:wfc.getApprovalSet().isApproved()"/>
  <Transition to="Send Rejection Notification"/>
  
</Step>
```

**The approval mode attribute — this is critical:**

`mode="serial"` — Each approver in the list must approve before the next is asked. If the first approver rejects, the workflow continues immediately to the rejection path without asking subsequent approvers.

`mode="parallel"` — All approvers are asked simultaneously. Workflow waits for ALL to respond. One rejection from any approver rejects the whole request.

`mode="parallelPoll"` — All approvers asked simultaneously. Workflow continues as soon as ANY ONE responds. First response wins.

`mode="any"` — Workflow continues as soon as any single approver approves. Used for "first come first served" approvals.

---

### The Call Step Anatomy — Calling Subprocesses and Rules

```xml
<!-- Calling a Rule -->
<Step icon="Default" name="Call Rule" posX="100" posY="200">
  <Script>
    <Source>
      import sailpoint.object.Rule;
      
      // Call a Rule by name
      // Rules are reusable Beanshell scripts stored as IIQ objects
      Rule rule = context.getObjectByName(Rule.class, "MyCustomRule");
      
      // Pass arguments to the rule
      Map ruleArgs = new HashMap();
      ruleArgs.put("identity", wfc.get("identity"));
      ruleArgs.put("plan", wfc.get("plan"));
      
      // Execute the rule
      Object result = context.runRule(rule, ruleArgs);
      
      wfc.put("ruleResult", result);
    </Source>
  </Script>
  <Transition to="Next Step"/>
</Step>

<!-- Calling a Subprocess workflow -->
<Step icon="Default" name="Call Subprocess" 
      type="subProcess"
      posX="100" posY="300">
  
  <!-- The name of the workflow to call as a subprocess -->
  <WorkflowRef>
    <Reference class="sailpoint.object.Workflow" 
               name="MySubprocess"/>
  </WorkflowRef>
  
  <!-- Arguments passed INTO the subprocess -->
  <Arg name="identityName" value="ref:identityName"/>
  <Arg name="plan" value="ref:plan"/>
  
  <!-- Values returned FROM the subprocess -->
  <Return name="subprocessResult" to="subprocessResult"/>
  
  <Transition to="Handle Subprocess Result"/>
</Step>
```

---

## Part 3: Input and Output — The Complete Data Flow

This is where most engineers have the biggest gaps. Let me trace data through an entire workflow precisely.

### How Data Gets INTO a Workflow

Data enters a workflow through three mechanisms. Understanding all three is essential.

**Mechanism 1 — Trigger Variables (from the event that launched the workflow)**

When IIQ detects a lifecycle event (identity created, identity changed, access requested), it creates a `WorkflowLaunchParameters` object containing:

```java
// What IIQ populates automatically in trigger-launched workflows
Map<String, Object> triggerVariables = {
    "identityName": "john.smith",           // The identity involved
    "identityDisplayName": "John Smith",    // Human-readable name
    "trigger": "Create",                    // What triggered the workflow
    "event": lifecycleEvent,                // The full lifecycle event object
    "launcher": "spadmin",                  // Who/what launched it
    "identityModel": identityObject,        // The full identity object AT TRIGGER TIME
    "nativeIdentity": null,                 // For connector events
    "application": null                     // For connector events
};
```

These are available in your workflow as variables without any explicit declaration. The key insight: `identityModel` is a snapshot of the identity at the moment the trigger fired. If the identity changes after the trigger fires but before your workflow processes it, `identityModel` is stale. Always re-fetch the identity from the database in the first workflow step for current state.

**Mechanism 2 — Declared Input Variables (from manual launch or API)**

```xml
<!-- Declared in workflow definition -->
<Variable name="identityName" input="true" required="true">
  <Description>Name of the identity to process</Description>
</Variable>

<Variable name="requestType" input="true" required="true">
  <Description>Type of request: joiner, mover, leaver</Description>
</Variable>

<Variable name="requestedApplications" input="true" required="false">
  <Description>List of applications to provision</Description>
</Variable>
```

These appear in the IIQ UI when you launch the workflow manually. When called from another workflow, these are the subprocess arguments. When called via API, these are the parameters you pass.

**Mechanism 3 — Computed Variables (set during execution)**

```xml
<!-- Computed variables declared in workflow definition -->
<!-- These are set by steps during execution, not passed in -->
<Variable name="plan" output="false">
  <Description>The provisioning plan built during execution</Description>
</Variable>

<Variable name="approvalDecision" output="false">
  <Description>Result of the approval step</Description>
</Variable>

<Variable name="errors" output="false" type="List">
  <Description>Accumulated errors during processing</Description>
</Variable>
```

---

### How Data Flows BETWEEN Steps

This is the most important thing to understand precisely. Variables are the only way data flows between steps. There is no return value chaining like in regular programming.

```xml
<!-- Step 1: Set a variable -->
<Step name="Step One">
  <Script>
    <Source>
      // This sets a WORKFLOW-LEVEL variable
      // Persists for the entire workflow lifecycle
      wfc.put("computedValue", "hello");
      
      // This sets a LOCAL variable
      // Disappears when this step completes
      String localVar = "world";  // LOST after step completes
      
      // To pass localVar to next step, you MUST put it in wfc
      wfc.put("localVarPassedOn", localVar);
    </Source>
  </Script>
  <Return name="returnValue" to="workflowVariable"/>
  <!-- returnValue = the script's return statement value -->
  <!-- workflowVariable = the wfc variable name to store it in -->
  <Transition to="Step Two"/>
</Step>

<!-- Step 2: Read the variable -->
<Step name="Step Two">
  <Script>
    <Source>
      // Read variable set by Step 1
      String value = (String) wfc.get("computedValue");
      // value = "hello"
      
      // This CANNOT read localVar from Step 1
      // It is gone
      
      // BUT it CAN read localVarPassedOn
      String passed = (String) wfc.get("localVarPassedOn");
      // passed = "world"
    </Source>
  </Script>
  <Transition to="Step Three"/>
</Step>
```

---

### How Data Gets OUT of a Workflow

Data leaves a workflow through two mechanisms:

**Mechanism 1 — Output variables (for subprocess workflows)**

```xml
<!-- Declared as output in workflow definition -->
<Variable name="provisioningResult" output="true">
  <Description>Result returned to calling workflow</Description>
</Variable>
```

When this workflow is called as a subprocess from another workflow, the calling workflow can read these output variables via its `<Return>` declarations.

**Mechanism 2 — Side effects (for top-level workflows)**

Top-level workflows (not subprocesses) return results by creating side effects in IIQ:
- Provisioning results written to `spt_provisioning_transaction`
- Identity changes committed via `context.saveObject(identity)`
- Work items created via `WorkflowContext.sendApproval()`
- Notifications sent via rule or email template
- Audit events logged via `Auditor.log()`

---

## Part 4: Building Custom Workflows — Step by Step With DERIVE

Now let's build real custom workflows using DERIVE to get the right architecture before writing any code.

### Custom Workflow 1: New Employee Joiner Workflow

**DERIVE Prompt — Round 1:**

"I am building a joiner workflow in SailPoint IIQ 8.3 for new employee provisioning. The business requirements:

When a new employee is created in IIQ (triggered by HR feed aggregation):
1. Detect if the employee is a contractor or full-time employee
2. For full-time employees: provision Active Directory account, Email account, VPN access
3. For contractors: provision Active Directory account only (no email, no VPN)
4. Manager must approve the provisioning before it executes
5. If manager does not approve within 48 hours, escalate to HR Manager
6. After provisioning completes, send welcome email to new employee's personal email
7. If any provisioning fails, notify the IT helpdesk

Technical context: we use a custom HR attribute 'employeeType' on the identity. Values: 'FTE' for full-time, 'CONTRACTOR' for contractors. Manager is stored on the identity object as identity.getManager(). HR Manager is stored as a system configuration attribute.

Do NOT give me a complete workflow XML immediately. 

Step 1: Tell me what the correct workflow trigger configuration is for 'identity created from aggregation' specifically — what event and what configuration do I need?

Step 2: Tell me the correct way to detect contractor vs FTE inside a workflow step — what is the exact Beanshell to read a custom identity attribute?

Step 3: Tell me the correct data structure for a ProvisioningPlan that provisions an AD account — what are the minimum required components?

Step 4: Tell me the correct way to implement the 48-hour escalation — what is the IIQ mechanism for timed approval escalation?

After all four, give me the complete workflow structure."

**What DERIVE extracts:**

**Step 1 — Correct trigger configuration:**

The trigger for "identity created from aggregation" in IIQ is a Lifecycle Event. Configure it at:

```
IIQ UI → Setup → Lifecycle Events → New
Event Type: Create
Condition: (leave blank for all creates, or add a rule to filter)
Workflow: [Your joiner workflow name]
```

Critical detail: the lifecycle event fires for EVERY identity creation including test identities, service accounts, and system identities. You need a condition rule or a first step in your workflow that filters out non-human identities. Otherwise your joiner workflow runs for every service account created during aggregation.

```java
// First step: validate this is a real employee
// Check for the HR attribute that marks real employees
String employeeType = (String) identity.getAttribute("employeeType");

if (employeeType == null || 
    (!employeeType.equals("FTE") && !employeeType.equals("CONTRACTOR"))) {
    // Not a real employee - terminate workflow gracefully
    wfc.put("skipProcessing", Boolean.TRUE);
    wfc.put("skipReason", "Identity is not FTE or CONTRACTOR: " + employeeType);
}
```

**Step 2 — Reading custom identity attributes:**

```java
// Correct way to read a custom identity attribute in Beanshell
import sailpoint.object.Identity;

// Get the identity from workflow variable (set earlier in workflow)
Identity identity = (Identity) wfc.get("identity");

// Read custom attribute - returns Object, must cast
String employeeType = (String) identity.getAttribute("employeeType");

// Safe null check pattern
if ("FTE".equals(employeeType)) {
    wfc.put("isFTE", Boolean.TRUE);
    wfc.put("isContractor", Boolean.FALSE);
} else if ("CONTRACTOR".equals(employeeType)) {
    wfc.put("isFTE", Boolean.FALSE);
    wfc.put("isContractor", Boolean.TRUE);
} else {
    // Unknown type - log and handle
    log.warn("Unknown employeeType: " + employeeType + 
             " for identity: " + identity.getName());
    wfc.put("isFTE", Boolean.FALSE);
    wfc.put("isContractor", Boolean.FALSE);
}
```

**Step 3 — ProvisioningPlan minimum structure:**

```java
// Minimum required ProvisioningPlan for AD account creation
import sailpoint.object.ProvisioningPlan;
import sailpoint.object.ProvisioningPlan.AccountRequest;
import sailpoint.object.ProvisioningPlan.AttributeRequest;

// Create the plan
ProvisioningPlan plan = new ProvisioningPlan();

// Set the identity the plan is for
plan.setIdentity(identity);

// Create an account request for AD
AccountRequest accountRequest = new AccountRequest();
accountRequest.setApplication("Active Directory");  // Must match exact app name in IIQ
accountRequest.setOperation(AccountRequest.Operation.Create);

// Required attributes for AD account creation
// Exact attribute names must match your AD connector schema
accountRequest.add(new AttributeRequest("sAMAccountName", 
    ProvisioningPlan.Operation.Set, 
    generateSAMAccountName(identity)));  // Your naming convention logic

accountRequest.add(new AttributeRequest("distinguishedName",
    ProvisioningPlan.Operation.Set,
    generateDistinguishedName(identity)));  // Target OU path

accountRequest.add(new AttributeRequest("userPrincipalName",
    ProvisioningPlan.Operation.Set,
    identity.getAttribute("email")));

accountRequest.add(new AttributeRequest("givenName",
    ProvisioningPlan.Operation.Set,
    identity.getFirstname()));

accountRequest.add(new AttributeRequest("sn",
    ProvisioningPlan.Operation.Set,
    identity.getLastname()));

// Password - typically set via IIQ password policy
accountRequest.add(new AttributeRequest("password",
    ProvisioningPlan.Operation.Set,
    generateTemporaryPassword()));

plan.add(accountRequest);
wfc.put("plan", plan);
```

**Step 4 — 48-hour escalation mechanism:**

IIQ implements approval escalation via the `Escalation` element in the approval step:

```xml
<Approval mode="serial" owner="script:getManager(wfc)">
  
  <!-- Escalation configuration -->
  <Escalation 
      escalationAction="script:escalateToHRManager(wfc)"
      escalationRule="script:shouldEscalate(wfc)"
      escalationStyle="reminder"
      maxReminders="2"
      reminderFrequency="24"       <!-- Hours between reminders -->
      hoursBetweenReminders="24"
      escalationFrequency="48">    <!-- Hours before escalation fires -->
  </Escalation>
  
  <Form>
    <!-- approval form fields -->
  </Form>
  
</Approval>
```

The escalation fires after 48 hours if the approver has not responded. `escalationAction` is a Beanshell script that changes the work item owner to the HR Manager and notifies them.

---

### The Complete Joiner Workflow XML

Now that DERIVE has established the correct approach for each component, here is the complete workflow:

```xml
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE Workflow PUBLIC 
    "sailpoint.dtd" "sailpoint.dtd">

<Workflow explicitTransitions="true" 
          libraries="Identity,Role,PolicyViolation,LCM,IdentityRequest"
          name="Custom Joiner Workflow" 
          type="IdentityLifecycle">

  <!-- ============================================ -->
  <!-- VARIABLE DECLARATIONS                        -->
  <!-- ============================================ -->
  
  <!-- Input variables - populated by trigger or manual launch -->
  <Variable input="true" name="identityName" required="true">
    <Description>Name of the new identity to process</Description>
  </Variable>
  
  <Variable input="true" name="trigger" required="false">
    <Description>The lifecycle trigger that launched this workflow</Description>
  </Variable>
  
  <!-- Computed variables - set during execution -->
  <Variable name="identity">
    <Description>The identity object being processed</Description>
  </Variable>
  
  <Variable name="employeeType">
    <Description>FTE or CONTRACTOR</Description>
  </Variable>
  
  <Variable name="plan">
    <Description>The provisioning plan to execute</Description>
  </Variable>
  
  <Variable name="approvalDecision">
    <Description>Result of manager approval</Description>
  </Variable>
  
  <Variable name="provisioningErrors" type="List">
    <Description>Errors accumulated during provisioning</Description>
  </Variable>
  
  <Variable name="hasError" type="Boolean">
    <Description>Whether any errors occurred</Description>
  </Variable>

  <!-- ============================================ -->
  <!-- STEP 1: INITIALIZE AND VALIDATE              -->
  <!-- ============================================ -->
  
  <Step icon="Start" name="Start" posX="50" posY="100">
    <Transition to="Fetch Identity"/>
  </Step>
  
  <Step icon="Default" name="Fetch Identity" posX="150" posY="100">
    <Script>
      <Source>
        import sailpoint.object.Identity;
        import org.apache.log4j.Logger;
        
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        
        String identityName = (String) wfc.get("identityName");
        log.info("Joiner workflow starting for: " + identityName);
        
        // ALWAYS re-fetch from database - trigger snapshot may be stale
        Identity identity = context.getObjectByName(
            Identity.class, identityName
        );
        
        if (identity == null) {
            log.error("Identity not found: " + identityName);
            wfc.put("hasError", Boolean.TRUE);
            wfc.put("errorMessage", 
                "Identity not found: " + identityName);
            return null;
        }
        
        // Validate employee type
        String employeeType = (String) identity.getAttribute("employeeType");
        
        if (employeeType == null) {
            log.warn("No employeeType for: " + identityName + 
                     " - skipping joiner workflow");
            wfc.put("hasError", Boolean.TRUE);
            wfc.put("errorMessage", "No employeeType attribute");
            return null;
        }
        
        if (!"FTE".equals(employeeType) && 
            !"CONTRACTOR".equals(employeeType)) {
            log.info("Non-employee identity: " + identityName + 
                     " type: " + employeeType + " - skipping");
            wfc.put("hasError", Boolean.TRUE);
            wfc.put("errorMessage", "Not a human employee");
            return null;
        }
        
        // All valid - set workflow variables
        wfc.put("identity", identity);
        wfc.put("employeeType", employeeType);
        wfc.put("hasError", Boolean.FALSE);
        wfc.put("provisioningErrors", new ArrayList());
        
        log.info("Identity validated: " + identityName + 
                 " type: " + employeeType);
        return identity;
      </Source>
    </Script>
    <Return name="identity" to="identity"/>
    
    <Transition to="End - Skip" 
                when="script:(Boolean)wfc.get(&quot;hasError&quot;)"/>
    <Transition to="Build Provisioning Plan"/>
  </Step>

  <!-- ============================================ -->
  <!-- STEP 2: BUILD PROVISIONING PLAN              -->
  <!-- ============================================ -->
  
  <Step icon="Default" name="Build Provisioning Plan" 
        posX="250" posY="100">
    <Script>
      <Source>
        import sailpoint.object.Identity;
        import sailpoint.object.ProvisioningPlan;
        import sailpoint.object.ProvisioningPlan.AccountRequest;
        import sailpoint.object.ProvisioningPlan.AttributeRequest;
        import org.apache.log4j.Logger;
        
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        
        Identity identity = (Identity) wfc.get("identity");
        String employeeType = (String) wfc.get("employeeType");
        
        ProvisioningPlan plan = new ProvisioningPlan();
        plan.setIdentity(identity);
        
        // ALL employees get AD account
        AccountRequest adRequest = new AccountRequest();
        adRequest.setApplication("Active Directory");
        adRequest.setOperation(AccountRequest.Operation.Create);
        
        // Build sAMAccountName: firstname.lastname
        String firstName = identity.getFirstname() != null ? 
            identity.getFirstname().toLowerCase() : "";
        String lastName = identity.getLastname() != null ? 
            identity.getLastname().toLowerCase() : "";
        String samAccountName = firstName + "." + lastName;
        
        adRequest.add(new AttributeRequest(
            "sAMAccountName", 
            ProvisioningPlan.Operation.Set, 
            samAccountName
        ));
        
        // Target OU based on employee type
        String targetOU = "FTE".equals(employeeType) ?
            "OU=Employees,OU=Users,DC=company,DC=com" :
            "OU=Contractors,OU=Users,DC=company,DC=com";
            
        adRequest.add(new AttributeRequest(
            "distinguishedName",
            ProvisioningPlan.Operation.Set,
            "CN=" + firstName + " " + lastName + "," + targetOU
        ));
        
        adRequest.add(new AttributeRequest(
            "givenName", ProvisioningPlan.Operation.Set, 
            identity.getFirstname()
        ));
        adRequest.add(new AttributeRequest(
            "sn", ProvisioningPlan.Operation.Set,
            identity.getLastname()
        ));
        
        plan.add(adRequest);
        
        // FTE employees also get Email and VPN
        if ("FTE".equals(employeeType)) {
            
            // Email account
            AccountRequest emailRequest = new AccountRequest();
            emailRequest.setApplication("Exchange");
            emailRequest.setOperation(AccountRequest.Operation.Create);
            emailRequest.add(new AttributeRequest(
                "mailbox", 
                ProvisioningPlan.Operation.Set,
                samAccountName + "@company.com"
            ));
            plan.add(emailRequest);
            
            // VPN access
            AccountRequest vpnRequest = new AccountRequest();
            vpnRequest.setApplication("Cisco VPN");
            vpnRequest.setOperation(AccountRequest.Operation.Create);
            vpnRequest.add(new AttributeRequest(
                "userId",
                ProvisioningPlan.Operation.Set,
                samAccountName
            ));
            plan.add(vpnRequest);
            
            log.info("Built FTE plan with AD + Email + VPN for: " + 
                     identity.getName());
        } else {
            log.info("Built CONTRACTOR plan with AD only for: " + 
                     identity.getName());
        }
        
        wfc.put("plan", plan);
        return plan;
      </Source>
    </Script>
    <Return name="plan" to="plan"/>
    <Transition to="Manager Approval"/>
  </Step>

  <!-- ============================================ -->
  <!-- STEP 3: MANAGER APPROVAL WITH ESCALATION     -->
  <!-- ============================================ -->
  
  <Step icon="Approval" name="Manager Approval" 
        type="approval" posX="350" posY="100">
    <Approval mode="serial" 
              owner="script:getManagerName(wfc)">
      
      <!-- Escalation: remind after 24h, escalate after 48h -->
      <Escalation 
          escalationAction="script:escalateToHRManager(wfc)"
          escalationRule="script:shouldEscalate(wfc)"
          escalationStyle="both"
          maxReminders="2"
          reminderFrequency="24"
          escalationFrequency="48"/>
      
      <Form>
        <Section label="New Employee Access Request">
          <Field displayName="Employee Name" 
                 name="employeeDisplay"
                 readOnly="true"
                 value="script:getEmployeeDisplayInfo(wfc)"/>
          <Field displayName="Access Being Requested"
                 name="accessSummary"
                 readOnly="true"
                 value="script:getPlanSummary(wfc)"/>
          <Field displayName="Your Decision Comments"
                 name="approvalComments"
                 type="string"
                 required="false"/>
        </Section>
      </Form>
      
    </Approval>
    
    <!-- After approval decision, capture the result -->
    <Script>
      <Source>
        import org.apache.log4j.Logger;
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        
        boolean approved = wfc.getApprovalSet().isApproved();
        log.info("Manager approval decision: " + approved + 
                 " for: " + wfc.get("identityName"));
        wfc.put("approvalDecision", approved ? "approved" : "rejected");
      </Source>
    </Script>
    
    <Transition to="Provision Access" 
                when="script:&quot;approved&quot;.equals(wfc.get(&quot;approvalDecision&quot;))"/>
    <Transition to="Send Rejection Notification"/>
  </Step>

  <!-- ============================================ -->
  <!-- STEP 4: PROVISION ACCESS                     -->
  <!-- ============================================ -->
  
  <Step icon="Default" name="Provision Access" posX="450" posY="100">
    <Script>
      <Source>
        import sailpoint.object.ProvisioningPlan;
        import sailpoint.object.ProvisioningResult;
        import sailpoint.api.Provisioner;
        import org.apache.log4j.Logger;
        
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        
        ProvisioningPlan plan = (ProvisioningPlan) wfc.get("plan");
        List errors = (List) wfc.get("provisioningErrors");
        
        try {
            // Execute the provisioning plan
            Provisioner provisioner = new Provisioner(context);
            provisioner.execute(plan);
            
            // Check results for each account request
            List accountRequests = plan.getAccountRequests();
            if (accountRequests != null) {
                for (Object reqObj : accountRequests) {
                    ProvisioningPlan.AccountRequest req = 
                        (ProvisioningPlan.AccountRequest) reqObj;
                    
                    ProvisioningResult result = req.getResult();
                    if (result != null && 
                        ProvisioningResult.STATUS_FAILED.equals(
                            result.getStatus())) {
                        String errorMsg = req.getApplication() + 
                            " provisioning failed: " + 
                            result.getErrors();
                        errors.add(errorMsg);
                        log.error(errorMsg);
                    }
                }
            }
            
            wfc.put("provisioningErrors", errors);
            wfc.put("hasProvisioningErrors", 
                errors != null && errors.size() > 0);
                
        } catch (Exception e) {
            log.error("Provisioning exception for: " + 
                      wfc.get("identityName"), e);
            errors.add("Provisioning exception: " + e.getMessage());
            wfc.put("provisioningErrors", errors);
            wfc.put("hasProvisioningErrors", Boolean.TRUE);
        }
      </Source>
    </Script>
    
    <Transition to="Notify Helpdesk of Errors"
                when="script:(Boolean)wfc.get(&quot;hasProvisioningErrors&quot;)"/>
    <Transition to="Send Welcome Email"/>
  </Step>

  <!-- ============================================ -->
  <!-- STEP 5: NOTIFICATIONS                        -->
  <!-- ============================================ -->
  
  <Step icon="Default" name="Send Welcome Email" posX="550" posY="100">
    <Script>
      <Source>
        import sailpoint.object.Identity;
        import sailpoint.object.EmailOptions;
        import sailpoint.object.EmailTemplate;
        import org.apache.log4j.Logger;
        
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        
        Identity identity = (Identity) wfc.get("identity");
        
        // Get personal email - NOT the corporate email (may not exist yet)
        String personalEmail = (String) identity.getAttribute(
            "personalEmail"
        );
        
        if (personalEmail == null || personalEmail.trim().isEmpty()) {
            log.warn("No personal email for: " + identity.getName() + 
                     " - cannot send welcome email");
            return;
        }
        
        // Load email template
        EmailTemplate template = context.getObjectByName(
            EmailTemplate.class, "New Employee Welcome Email"
        );
        
        if (template == null) {
            log.error("Email template 'New Employee Welcome Email' not found");
            return;
        }
        
        // Build email variables
        Map vars = new HashMap();
        vars.put("identity", identity);
        vars.put("employeeType", wfc.get("employeeType"));
        vars.put("firstName", identity.getFirstname());
        
        EmailOptions options = new EmailOptions(
            personalEmail, vars
        );
        
        context.sendEmailNotification(template, options);
        log.info("Welcome email sent to: " + personalEmail);
      </Source>
    </Script>
    <Transition to="End - Success"/>
  </Step>
  
  <Step icon="Default" name="Send Rejection Notification" 
        posX="350" posY="250">
    <Script>
      <Source>
        import sailpoint.object.Identity;
        import org.apache.log4j.Logger;
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        
        log.info("Provisioning rejected for: " + wfc.get("identityName"));
        // Send notification to HR that provisioning was rejected
        // Implementation: load HR email template, send to HR group
        // Similar pattern to welcome email above
      </Source>
    </Script>
    <Transition to="End - Rejected"/>
  </Step>
  
  <Step icon="Default" name="Notify Helpdesk of Errors" 
        posX="550" posY="250">
    <Script>
      <Source>
        import org.apache.log4j.Logger;
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        
        List errors = (List) wfc.get("provisioningErrors");
        String identityName = (String) wfc.get("identityName");
        
        log.error("Provisioning errors for " + identityName + 
                  ": " + errors);
        
        // Send helpdesk notification
        // Load helpdesk email template
        // Include error list in notification
        // Create helpdesk ticket if integration exists
      </Source>
    </Script>
    <Transition to="Send Welcome Email"/>
    <!-- Still send welcome email - partial provisioning still happened -->
  </Step>

  <!-- ============================================ -->
  <!-- END STEPS                                    -->
  <!-- ============================================ -->
  
  <Step icon="Stop" name="End - Success" posX="650" posY="100">
    <Script>
      <Source>
        import org.apache.log4j.Logger;
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        log.info("Joiner workflow completed successfully for: " + 
                 wfc.get("identityName"));
      </Source>
    </Script>
  </Step>
  
  <Step icon="Stop" name="End - Rejected" posX="450" posY="250">
    <Script>
      <Source>
        import org.apache.log4j.Logger;
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        log.info("Joiner workflow ended - rejected for: " + 
                 wfc.get("identityName"));
      </Source>
    </Script>
  </Step>
  
  <Step icon="Stop" name="End - Skip" posX="150" posY="250">
    <Script>
      <Source>
        import org.apache.log4j.Logger;
        Logger log = Logger.getLogger(
            "com.company.workflows.JoinerWorkflow"
        );
        log.info("Joiner workflow skipped for: " + 
                 wfc.get("identityName") + 
                 " reason: " + wfc.get("errorMessage"));
      </Source>
    </Script>
  </Step>

</Workflow>
```

---

### Custom Workflow 2: Leaver Workflow With Certification Trigger

**DERIVE Prompt — Round 1:**

"I am building a leaver workflow in IIQ 8.3. Requirements:

When an employee's status changes to 'Terminated' in the HR attribute:
1. Immediately disable all application accounts (do not delete — disable)
2. Transfer the employee's manager role to their manager
3. Generate a self-service certification for the manager to review the terminated employee's access
4. After certification completes, revoke any entitlements the manager certified for removal
5. 30 days after termination, delete (not just disable) all application accounts

The 30-day delayed deletion is the part I don't understand architecturally. A workflow step that waits 30 days — how does this work in IIQ? Does the workflow stay open for 30 days? Does IIQ support scheduled future actions from within a workflow?

Walk through the IIQ mechanism for delayed execution before I try to implement it."

**What DERIVE extracts on delayed execution:**

IIQ has two mechanisms for delayed execution from within a workflow:

**Mechanism 1 — The waitFor condition with a timer:**

```xml
<Step name="Wait For 30 Days" type="approval">
  <!-- Use an approval step with a timer as a workaround -->
  <!-- This keeps the workflow case open but requires no human action -->
  <!-- The workflow serializes to database and consumes no threads -->
  
  <Approval owner="spadmin" 
            description="30-day hold before account deletion">
    
    <!-- Auto-approve after 30 days using escalation mechanism -->
    <Escalation
        escalationAction="script:autoApprove(wfc)"
        escalationFrequency="720"/>  <!-- 720 hours = 30 days -->
    
  </Approval>
  
  <Transition to="Delete Accounts"/>
</Step>
```

**Mechanism 2 — The Request/Task approach (cleaner for long delays):**

```java
// Better approach for 30-day delay:
// Schedule a Task to run 30 days from now
// The current workflow completes immediately
// The scheduled task handles the deletion

import sailpoint.object.TaskSchedule;
import sailpoint.object.TaskDefinition;
import java.util.Calendar;
import java.util.Date;

// Calculate 30 days from now
Calendar cal = Calendar.getInstance();
cal.add(Calendar.DAY_OF_MONTH, 30);
Date deletionDate = cal.getTime();

// Create a scheduled task for the deletion
TaskDefinition taskDef = context.getObjectByName(
    TaskDefinition.class, "Account Deletion Task"
);

TaskSchedule schedule = new TaskSchedule();
schedule.setName("Delete accounts for " + identityName + 
                 " on " + deletionDate);
schedule.setTaskDefinition(taskDef);
schedule.setNextExecution(deletionDate);
schedule.setArgument("identityName", identityName);

context.saveObject(schedule);
context.commitTransaction();

log.info("Scheduled account deletion for: " + identityName + 
         " on: " + deletionDate);
```

Mechanism 2 is preferred for delays longer than a few hours because: the workflow case closes immediately (cleaner), the scheduled task is visible in the IIQ task scheduler UI, and the task can be cancelled if the employee is re-hired before the 30 days expire.

---

## The DERIVE Loop for IIQ Workflow Development — Summary

The meta-pattern that makes workflow development reliable:

**Round 1 (D):** State the business requirements precisely with the specific technical question you cannot answer from documentation alone. Do not ask for a complete implementation — ask for the mechanism behind the specific part you do not understand.

**Round 2 (E + R):** Take the mechanism explanation and expose the assumption it made about your environment. Eliminate that assumption. Ask for the failure mode when the mechanism is used incorrectly — what does the workflow log show when this breaks?

**Round 3 (I):** Before writing any workflow XML, force step-by-step reasoning about variable scoping, thread context, and error handling. These are the three areas where IIQ workflow bugs most commonly appear and they are invisible in the happy path.

**Round 4 (V):** Before deploying to production, verify each Beanshell API call against your IIQ version's Javadoc on Compass. Specifically: method signatures, null return behaviors, exception types thrown. These are the areas with highest hallucination risk.

**Round 5 (E — Expand):** Every workflow you build reveals a pattern you will reuse. After building the joiner workflow, extract the approval escalation pattern, the error accumulation pattern, and the identity validation pattern into standalone Rules in IIQ. The next workflow calls these rules instead of duplicating the logic. This is how your IIQ development compound over time — each workflow makes the next one faster.

The workflow code and IIQ API usage in this response should be verified against your specific IIQ version's Compass documentation and tested in a non-production environment before production deployment. The patterns and mechanisms are correct. The specific method signatures and configuration property names carry version-specific hallucination risk that only Compass and testing can eliminate.
