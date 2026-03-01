Before the prompt, lock in why this is the right question to ask.

---

## Why A Reusable Prompt Is The Right Tool

Every time you ask a question without structure, you get the tutorial answer. Every time you apply DERIVE manually, you spend 10 minutes constructing the prompt before getting value. A reusable prompt collapses that 10 minutes to zero while preserving all the depth.

But a reusable prompt has one failure mode: it becomes a ritual you follow without understanding. So before the prompt, the one rule:

```
The prompt is a forcing function, not a magic spell.
It works because each section mechanically destroys 
a different category of generic response.
If you understand why each section exists,
you can modify it for any situation.
If you just copy-paste it, it will eventually 
produce outputs you can't evaluate.
```

---

## The Master Prompt

```
═══════════════════════════════════════════════════════════
DERIVE + 10 INVARIANTS — MASTER PROMPT
═══════════════════════════════════════════════════════════

[CONTEXT — destroy the default path]
I am working on: [specific system, not abstract problem]
Stack: [exact technologies + versions]
Scale: [real numbers — req/sec, users, data volume, team size]
Current state: [what exists now, not what you want to build]
Already tried: [what failed and why — forces model past basics]

The specific problem I am solving:
[one sentence, concrete, not "I want to understand X"]

The contradiction I cannot resolve with standard explanations:
[something that doesn't fit the tutorial answer]

───────────────────────────────────────────────────────────
[ASSUMPTIONS — eliminate wrong defaults]
Do NOT assume: [list what the generic answer assumes 
                that is wrong for your situation]

It is NOT the case that: [explicit negations of 
                          standard assumptions]

My hard constraints: [things that cannot change —
                      budget, team size, existing systems,
                      compliance requirements]

───────────────────────────────────────────────────────────
[LONG TAIL — activate operational knowledge]
You are a senior engineer who has been paged because 
[specific failure related to my problem].

You have seen this exact failure mode in production.
You know what the metrics look like 10 minutes before 
anyone understands what is wrong.

Do not give me the tutorial answer.
Give me the post-mortem answer:
— what broke
— what the failure signature looked like
— what the code pattern caused it
— what the fix was
— what you wish you had known before it happened

───────────────────────────────────────────────────────────
[REASONING — force chain before conclusion]
Before any recommendation, work through these steps 
explicitly. Do not skip steps. Do not combine steps.

Step 1 — FLOW: How does data/requests move through 
         this system? Where can it back up, block, 
         or cascade? What is the backpressure mechanism?

Step 2 — CONSISTENCY: If this operation fails halfway, 
         what state is the system in? Is recovery 
         automatic or manual?

Step 3 — CAPACITY: What resource exhausts first under 
         load? What is the failure signature before 
         exhaustion? What is the first metric that moves?

Step 4 — LATENCY: What is the P99 impact, not the 
         average? Where does tail latency come from 
         in this design? What amplifies it?

Step 5 — TRUST: Who can do what? What is the 
         object-level authorization check? 
         What data leaves the system boundary?

Step 6 — OBSERVABILITY: If this breaks at 3am, 
         what does the on-call engineer look at first? 
         What metric, what log, what dashboard? 
         Is there a runbook?

Step 7 — ECONOMICS: What is the tradeoff being made? 
         What does this cost in complexity, 
         operational burden, and money? 
         What is the cheaper alternative and 
         why is it insufficient?

Step 8 — CHANGE: Can this deploy without downtime? 
         If state changes, is it backward compatible 
         during the transition window? 
         How fast is rollback?

Step 9 — DEPENDENCY: What external systems does this 
         touch? What is the availability math? 
         What is the fallback when each one fails?

Step 10 — OPERABILITY: If this enters a bad state, 
          what does a human do? Is there an admin 
          endpoint? A kill switch? A runbook? 
          An audit trail?

After all 10 steps: give your recommendation.
Flag which steps reveal genuine risk for my situation.
Flag which steps are low risk and why.

───────────────────────────────────────────────────────────
[VERIFICATION — before I accept the answer]
After your recommendation:

1. Rate your confidence: high / medium / low
   For each major claim: is it documented behavior, 
   common practice, or reasoning about my situation?

2. What is the strongest argument that your 
   recommendation is wrong for my specific constraints?

3. What is the specific test, benchmark, or load test 
   I should run to verify the most critical claim 
   before going to production?

4. What is the one thing about this problem I didn't 
   ask about that will surprise me in production?

───────────────────────────────────────────────────────────
[EXPAND — what to ask next]
At the end of your response, give me:

The ONE follow-up question that would most change 
your recommendation if the answer were different 
from what you assumed.

The ONE invariant from the 10 above that is most 
at risk in my specific situation and why.
═══════════════════════════════════════════════════════════
```

---

## How To Use It — Three Modes

---

### Mode 1: Full Prompt (New System or Major Decision)

Use when: designing something new, making an irreversible architectural decision, evaluating a major technology choice.

Fill every section. The 10-invariant reasoning chain in Step 1-10 is the most important part. Don't skip invariants that seem irrelevant — the ones that seem irrelevant are often where the real risk hides.

**Example filled for our e-commerce project:**

```
[CONTEXT]
I am working on: order placement flow for e-commerce platform
Stack: Spring Boot 3.2 + Go 1.22 + PostgreSQL 15 + Redis 7
Scale: 50k DAU, 800 req/sec peak catalog, 50 orders/minute peak,
       team of 4 engineers, 18-month timeline
Current state: monolith with plain Java wiring, no framework
Already tried: nothing yet — designing before building

The specific problem I am solving:
Should the order placement flow be synchronous 
(caller waits for confirmation) or async 
(caller gets queued, result via webhook/polling)?

The contradiction I cannot resolve:
Synchronous gives better UX (immediate confirmation)
but creates tight coupling to inventory and payment services.
Async decouples services but makes the client experience 
complex and the failure handling harder to reason about.
Standard advice says "it depends" without resolving 
which factor dominates for our specific constraints.

[ASSUMPTIONS]
Do NOT assume we have Kubernetes — we are on AWS ECS.
Do NOT assume our team has distributed systems experience.
Do NOT assume we can tolerate eventual consistency 
for payment confirmation — it must be strongly consistent.
Do NOT assume infinite engineering time — 
we need this decision in 2 days, implementation in 2 weeks.

[HARD CONSTRAINTS]
Cannot add new AWS services without approval (2-week process).
Cannot exceed $2,000/month additional infrastructure cost.
Payment confirmation must be strongly consistent.
Team has zero experience with message queues in production.

[LONG TAIL]
You are a senior engineer who has been paged because
an async order flow produced duplicate charges when 
the webhook retry system fired twice for the same order.
You know what idempotency failures look like at 2am.
```

---

### Mode 2: Targeted Prompt (Specific Invariant Under Pressure)

Use when: debugging an incident, reviewing a PR for a specific concern, going deep on one invariant.

Keep Context, Assumptions, and Long Tail. Replace the 10-step reasoning chain with a single invariant focus:

```
[CONTEXT]
[same as above]

[TARGETED REASONING — CAPACITY INVARIANT ONLY]
My system is showing elevated P99 at 3x normal load.
CPU: 15%. Memory: normal. Database CPU: 8%.

Before giving any recommendation:

Step 1: List every resource that could be exhausted 
        given these symptoms. Rank by probability.

Step 2: For the most probable resource: 
        what does the metric signature look like 
        60 seconds before total failure?

Step 3: What is the immediate mitigation that 
        requires no deploy?

Step 4: What is the architectural fix that 
        prevents recurrence?

Only then: your recommendation.

[VERIFICATION — same as master prompt]
```

---

### Mode 3: Iterative Expansion (Multi-Round Investigation)

Use when: you've gotten a first answer and want to go deeper on the most interesting claim.

After each response, use this follow-up template:

```
Your previous response assumed [newly revealed assumption].
That is wrong for my situation because [specific reason].

Given that correction:

1. Which parts of your recommendation change?

2. Go one level deeper on [most interesting specific claim 
   from previous response].

3. Re-evaluate [the invariant most affected by the correction]
   against my actual constraints.

4. What does the code look like for the corrected approach?
   Give me the minimal working implementation, 
   not the tutorial version.
```

---

## The Cheat Sheet — When To Apply Each Invariant

Not every question needs all 10. Use this to know which 2-3 to focus on:

```
IF YOU ARE...                    FOCUS ON THESE INVARIANTS

Designing a new API endpoint     Flow, Trust, Latency
Designing a database schema      Consistency, Change, Operability
Choosing a caching strategy      Capacity, Consistency, Economics
Adding a new external API call   Dependency, Latency, Operability
Handling money or orders         Consistency, Trust, Operability
Deploying a schema change        Change, Operability, Observability
Debugging high latency           Capacity, Latency, Observability
Debugging data inconsistency     Consistency, Operability, Change
Reviewing a security feature     Trust, Observability, Operability
Planning a new microservice      Dependency, Economics, Change
Debugging a production incident  Observability, Operability, [whichever failed]
Preparing for an interview       All 10 — map yourself against the rubric
```

---

## The Self-Evaluation Rubric

After any answer you get from the prompt, evaluate it against this:

```
MECHANISM LEVEL (junior signal)
The answer describes tools and syntax.
"Use @Transactional"
"Add a circuit breaker"
"Use Redis for caching"

INVARIANT LEVEL (mid-senior signal)
The answer describes what property is being protected
and what breaks when it isn't.
"@Transactional protects consistency within one service —
 it cannot protect consistency across services,
 which requires saga with compensation"

ECONOMIC LEVEL (senior-staff signal)
The answer makes the tradeoff explicit with real costs.
"Saga adds 2 weeks of implementation and ongoing
 operational complexity. For 50 orders/minute,
 the simpler synchronous approach with retry handles
 99.9% of cases. The saga is justified only when
 order volume exceeds X or when compensation
 failures exceed Y per month in cost."

TARGET: Every important claim at invariant level minimum.
        Every architectural decision at economic level.
        If the answer is all mechanism: re-run with 
        more specific assumptions elimination.
```

---

## The One-Line Version

When you don't have time for the full prompt, this single addition to any question recovers most of the value:

```
Before answering, walk through which of these 
10 properties my design violates or risks:
Flow, Consistency, Capacity, Latency, Trust, 
Observability, Economics, Change, Dependency, Operability.

For each violation: what breaks and what fixes it.
Only then give your recommendation.
```

It's not as deep as the full prompt. But it's 10x better than asking the question naked — because it forces the model to check your design against the failure modes that matter before generating the confident tutorial answer that ignores all of them.
