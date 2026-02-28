Yes. And in some ways DERIVE works better here than in technical domains.

Here is why: technical domains have right answers that can be verified against documentation or test runs. Product management and consulting deal with judgment calls under uncertainty where the quality of the thinking process matters more than any specific answer. DERIVE forces rigorous thinking before conclusion — which is exactly what separates mediocre product decisions from great ones.

But there is an important difference in how you apply it.

---

## The Critical Difference for Non-Technical Domains

In Spring Boot troubleshooting, DERIVE's value is reaching specific technical knowledge buried in training data. In product management and consulting, the value is different:

**The LLM's training data contains:** thousands of product management frameworks, consulting methodologies, case studies, business school cases, strategy papers, market analyses, and management books. This is actually dense, high-quality training data.

**What generic prompts produce:** MBA textbook answers. Jobs-to-be-Done, Porter's Five Forces, MECE frameworks, North Star metrics. Every product manager and consultant has read these. They produce the appearance of analysis without actual insight.

**What DERIVE produces:** the application of those frameworks to your specific situation with the failure modes, the contradictions, and the uncomfortable truths that generic analysis never surfaces.

The hallucination risk here is different too. The LLM rarely fabricates frameworks. It fabricates specific market data, specific competitor details, specific financial benchmarks. The verification discipline shifts accordingly.

---

## Part 1: Product Management

### The Mental Model for Using DERIVE in PM

Product management has three core activities where LLMs give generic answers that DERIVE can improve dramatically:

**Discovery** — understanding what problem to solve
**Prioritization** — deciding what to build first
**Strategy** — deciding where to play and how to win

Generic PM prompts get you frameworks applied to imaginary companies. DERIVE gets you frameworks applied to your actual product, your actual users, your actual constraints.

---

## Scenario 1: Product Discovery — Finding the Real Problem

### Round 1 — D (Destroy the Default Path)

**Generic prompt:**
"How do I do product discovery for a B2B SaaS product?"

**Generic response:**
```
Conduct user interviews.
Build empathy maps.
Use Jobs-to-be-Done framework.
Identify pain points.
Validate with prototypes.
```

This is the product management curriculum at any bootcamp. It tells you nothing about your specific situation.

**DERIVE Round 1:**

"I am the product manager for a B2B project management tool. Our product is used by construction companies to manage subcontractor coordination. 200 customers, average 50 seats per customer. Annual contract value $15,000-$40,000. 

Our current problem: our 12-month retention is 71%. Industry benchmark for B2B SaaS at our price point is 85%. We are losing 29 customers per cohort year.

What we already know from exit surveys: 40% say 'found a better solution,' 30% say 'too complicated,' 20% say 'not enough ROI,' 10% say 'other.'

What we tried: we simplified the UI based on the 'too complicated' feedback. Redesigned onboarding. Retention improved by 2 percentage points over 6 months.

What we have NOT done: talked to the 40% who said 'found a better solution.' We assumed they went to a direct competitor.

Do NOT give me generic discovery advice. For the specific exit survey distribution I have described: what is the most likely real reason hiding behind each response category? Exit survey responses are known to be unreliable — what are the specific ways each category understates or misrepresents the actual reason for leaving?"

**What DERIVE extracts:**

**The real story behind each exit survey category:**

**"Found a better solution" (40%) — almost never means what it says:**

Exit survey respondents saying this are usually giving you the socially acceptable answer that requires no confrontation. The real reasons hidden inside this category:

They found a solution that does less but does the critical thing perfectly. Construction companies often leave broad platforms for a narrow tool that solves the one workflow their foreman actually uses every day. "Better solution" means "different scope" not "more features." Your product may be trying to solve 10 problems at C+ level when the churned customer only needed one problem solved at A level.

They were never actually using your product at the organizational level. A champion inside the company bought your product, the champion left, nobody else knew how to use it, the renewal came up and the new manager cancelled and said "we found something simpler" — which means "we found something our team actually uses" or "we found nothing and just cancelled."

The specific discovery question to ask these churned customers: "On the day you decided to cancel, what was the last thing you tried to do in our product that didn't work the way you expected?" Not "why did you leave" — that gets you the rehearsed answer. The specific last failure moment tells you the actual trigger.

**"Too complicated" (30%) — rarely means the UI is confusing:**

In B2B construction software, "too complicated" almost always means: the person who bought it is not the person who uses it. Your buyer is a construction project manager or owner who evaluated the product in a demo. The users are site supervisors and subcontractor coordinators who were handed login credentials with no context.

The UI simplification you did treated a user adoption problem as a design problem. That is why you only got 2 percentage points of improvement — you solved the symptom not the cause.

The real question: what is the ratio of seats purchased to seats with more than 5 logins in the first 90 days? If you are selling 50 seats and 8 people are actually using it, "too complicated" means "we couldn't get our team to adopt it" not "the buttons are confusing."

**"Not enough ROI" (20%) — means they never measured ROI:**

Customers who can articulate a specific ROI number do not churn saying "not enough ROI." They either stay because the number is positive or they negotiate a lower price. "Not enough ROI" as an exit reason means the customer never had a success metric defined when they bought the product. They bought on hope, couldn't measure value after a year, and cancelled.

The question this raises: what does your sales process look like for ROI definition? Do you help customers define what success looks like before they sign? Do you have a QBR (Quarterly Business Review) process that measures value delivered? If not, every customer renewal is a fresh justification of value that customers have to figure out themselves.

---

**Round 2 — E + R (Eliminate assumptions, reach long tail)**

**DERIVE Round 2:**

"Your analysis suggested the 'found a better solution' category is hiding a champion departure problem. You assumed we can identify which churned accounts had a champion departure.

We checked. Of the 40% who said 'found a better solution,' we can identify 8 accounts where the original buyer left the company before cancellation. That is approximately 14% of that category — not the majority.

You also suggested an adoption measurement: seats purchased vs seats with 5+ logins in 90 days. We ran that query. Results: across all 200 customers, average active seats are 31% of purchased seats. Among churned customers in the last 12 months, average active seats were 19% of purchased seats at the time of cancellation.

Do NOT revise your original analysis with these numbers — that is confirmation bias. Instead:

You are a SaaS product consultant who has done retention analysis for B2B tools in field-intensive industries (construction, manufacturing, field services) and has seen the specific retention failure patterns that are unique to these industries versus pure software B2B companies. What does 31% average seat activation and 19% at churn tell you specifically? What does the construction industry context change about the interpretation?"

**What Round 2 extracts:**

**The field industry adoption problem:**

31% active seats is not a product problem in field-intensive industries — it is a structural reality. Construction project managers, site supervisors, and subcontractors are not knowledge workers who sit at computers all day. They are on job sites. Many do not have company email addresses. Some share devices. The 69% "inactive" seats may include:

Workers who access the product via a shared tablet on site — their logins appear as a single "device user" not individual seat users. Your 31% may actually represent higher real usage than the number shows.

Subcontractors who were added to the platform for one project, completed the project, and are now legitimately inactive. They are not churned — they are between projects.

The 19% at churn number is the critical one. The gap between 31% average and 19% at churn is not a 12-percentage-point difference in seat activation — it is likely a signal that the project the software was managing completed and was not replaced by a new project in your system.

**The real hypothesis this unlocks:**

Your retention problem may not be a product quality problem. It may be a project lifecycle problem. Customers buy your product for a specific large project. The project takes 18 months. The product is genuinely useful for those 18 months. The project completes. The customer does not have an immediate next large project that requires the same coordination complexity. They cancel.

If this hypothesis is correct, no amount of UI improvement, discovery research, or feature building fixes your retention. The fix is: a commercial model that matches the project lifecycle (project-based pricing instead of annual subscription), or a land-and-expand motion that ensures customers start new projects in your system before the first project closes.

The specific test for this hypothesis: of your churned customers, what percentage cancelled within 6 months of completing a major project that was being managed in your system?

---

**Round 3 — I (Insert reasoning before conclusion)**

**DERIVE Round 3:**

"The project lifecycle hypothesis is compelling. Before we pursue it:

Step 1: Walk through the alternative hypotheses that could also explain 19% active seats at churn. List every competing explanation and what evidence would distinguish each from the project lifecycle hypothesis.

Step 2: Walk through what the pricing and commercial model would need to look like if the project lifecycle hypothesis is confirmed. What are the specific risks of moving to project-based pricing for a company at our stage (200 customers, $15-40k ACV)?

Step 3: Walk through what the product changes would need to look like if the project lifecycle hypothesis is confirmed — this is not a problem you can fix by building more features.

Step 4: Only after all three — tell me what the single highest-leverage action is for the next 90 days."

**What Round 3 extracts:**

**Step 1 — Competing hypotheses:**

Hypothesis A (project lifecycle): customers churn because their project completed. Evidence needed: cancellation timing relative to project completion dates in your system.

Hypothesis B (value realization gap): customers churn because they cannot measure what the product did for them. Evidence: do churned customers have defined success metrics in your system? Do they have a named CSM? Did they have QBRs?

Hypothesis C (competitive displacement): customers churn because a competitor specifically targets your renewal conversations. Evidence: do churned customers appear as competitor case studies within 6 months of cancellation?

Hypothesis D (economic buyer disengagement): the person who manages the renewal is not the person who uses the product. The renewal arrives, nobody champions it internally, it gets cut as a line item. Evidence: who signed the original contract vs who was the designated admin in the product?

These hypotheses are not mutually exclusive. Your 29% churn may be: 40% project lifecycle, 30% value realization gap, 20% economic buyer disengagement, 10% genuine competitive displacement. Each requires a different intervention.

**Step 2 — Project-based pricing risks:**

Revenue predictability collapses. Annual subscriptions produce predictable ARR. Project-based pricing produces lumpy revenue that correlates with construction project starts — which are cyclical and economic-cycle sensitive. Your investors and your financial planning both depend on predictable ARR. Moving to project pricing at 200 customers is a bet that TAM expansion from new customers compensates for revenue lumpiness from existing customers.

Upsell complexity increases. With annual subscriptions, upsell is "more seats." With project pricing, upsell is "more projects" — which requires your sales team to track customer project pipelines, which they do not currently do.

The middle path: keep annual subscriptions but add a "project pause" feature that allows customers to pause their subscription between projects at a reduced rate (25-30% of full price) rather than cancelling. This retains the customer relationship and ARR predictability while acknowledging the project lifecycle reality.

**Step 3 — Product changes:**

If project lifecycle is confirmed, the product needs: a project health score that tells CSMs when a managed project is approaching completion (so they can proactively start a conversation about the next project before the current one closes), a project handoff workflow that makes it easy to archive one project and start another without feeling like you are starting over, and a customer success trigger that fires when seat activation drops below 25% for more than 30 days.

**Step 4 — Highest leverage action in 90 days:**

Run the project lifecycle test. Pull the completion dates of major projects managed in your product for all 58 churned customers in the last 12 months. Map cancellation dates against project completion dates. If more than 60% of churned customers cancelled within 90 days of their last project completing, the hypothesis is confirmed and every subsequent decision follows from that data. This takes 2 weeks of analysis and costs nothing. It either confirms the hypothesis with high confidence or eliminates it, directing your attention to the other hypotheses. No other action has this ratio of insight to effort.

---

## Scenario 2: Prioritization — The Real Reason Your Roadmap Is Wrong

**Full DERIVE Prompt:**

"I am the PM for a FinTech payments API used by developers at e-commerce companies. Our API processes payments, refunds, and disputes. 500 enterprise customers, $8M ARR.

Our roadmap currently has 47 items in the backlog. Our prioritization framework is RICE (Reach, Impact, Confidence, Effort). We run quarterly planning. Every quarter we commit to 12 items and deliver 7.

Our specific problems:
1. Engineering says our estimates are consistently wrong by 2-3x
2. Sales says the roadmap never has what prospects ask for
3. Customer success says customers are churning over features we deprioritized 6 months ago
4. Our CEO asks why we are building X when competitor Y just launched Z

We already tried: switching from RICE to a different prioritization framework (ICE). Same results. Switching frameworks did not change outcomes.

Do NOT give me another prioritization framework. Tell me specifically what is wrong with our prioritization process that switching frameworks cannot fix. Then tell me what is structurally broken in how we are gathering input into the prioritization process."

**What DERIVE extracts:**

**What is structurally broken — four specific problems:**

**Problem 1 — You are prioritizing items, not outcomes:**

47 items in a backlog is a feature list, not a product strategy. RICE and ICE are item-level scoring frameworks. They answer "which item is more valuable than another item." They do not answer "what outcome are we trying to achieve this quarter and which items contribute to that outcome."

When every item competes against every other item with a score, the highest-scoring item wins regardless of whether it fits a coherent strategy. You end up with a roadmap that is a collection of individually justified features that do not add up to a product that moves a specific metric.

The structural fix: before scoring any item, define 2-3 outcome goals for the quarter (reduce churn by X%, increase expansion revenue by Y%, reduce time-to-first-payment for new customers by Z%). Items that do not contribute to these outcomes are automatically deprioritized regardless of their RICE score.

**Problem 2 — Your estimates are wrong because nobody owns the estimate:**

Engineering says estimates are wrong by 2-3x. This is not an engineering problem. It is a process problem. In most product teams, estimates are generated under social pressure — a PM asks "how long will this take" in a planning meeting, an engineer gives a number in the moment, that number becomes the plan. Nobody is accountable for the accuracy of the estimate because nobody did the work of breaking the item into discrete engineering tasks before estimating.

The structural fix: no item enters the sprint without a technical spike that results in a written breakdown of tasks and their individual estimates. The PM's job is to create the conditions for accurate estimation, not to accept estimates generated under social pressure.

**Problem 3 — Sales input is a distortion, not a signal:**

"Sales says the roadmap never has what prospects ask for." Sales brings you the features that lost the last 3 deals. This is recency-biased, sample-biased, and systematically skewed toward feature parity with the competitor that beat you most recently. It does not represent what your best customers want. It does not represent what would make your product defensible long-term. It represents what your sales team needs to overcome their most recent objection.

The structural fix: separate "competitive table-stakes features" (things you need to not lose deals) from "differentiation features" (things that make you win deals and retain customers). Sales input belongs in the first category only. Customer success input, retention data, and usage analysis belong in the second category.

**Problem 4 — Your CEO's question reveals a strategy vacuum:**

"Why are we building X when competitor Y just launched Z" is the question that gets asked when there is no articulated product strategy that the CEO has bought into. If everyone agreed on the strategy — the specific customer segment you are serving, the specific outcome you are optimizing for, the specific moat you are building — the answer to the CEO's question would be obvious: "Because Z serves a different customer than we are targeting" or "because Z is table-stakes and we are already building it for Q3."

The CEO is asking the strategy question in the disguise of a prioritization question. No prioritization framework answers it. Only an articulated, agreed-upon product strategy answers it.

---

## Part 2: Management Consulting

### The Mental Model for Using DERIVE in Consulting

Consulting work has three phases where generic LLM answers are particularly useless:

**Problem structuring** — what is the actual question we are being asked?
**Hypothesis development** — what do we think is true before we look at data?
**Recommendation development** — what should the client do?

Generic consulting prompts produce McKinsey-slide-sounding analysis that could apply to any company in any industry. DERIVE produces analysis that grapples with the specific situation.

---

## Scenario 3: Structuring a Cost Reduction Engagement

**Round 1 — D (Destroy the Default Path)**

**Generic prompt:**
"How do I structure a cost reduction consulting engagement?"

**Generic response:**
```
Conduct spend analysis.
Benchmark against industry.
Identify quick wins and strategic initiatives.
Build business case for each initiative.
Create implementation roadmap.
```

This is the table of contents for every cost reduction engagement ever run. It tells you nothing.

**DERIVE Round 1:**

"I am a management consultant brought in by the CFO of a regional hospital system. 3 hospitals, 8,000 employees, $1.2B annual revenue, currently losing $45M per year. The CFO wants $80M in annual cost reduction identified within 8 weeks.

Context that makes this engagement different from generic cost reduction:
- The CEO is opposed to the engagement. The CFO hired us over the CEO's objection.
- The clinical staff (60% of employees) are represented by two unions whose contracts expire in 14 months
- The last consultant who came through 3 years ago recommended workforce reductions. 3 of those recommendations were implemented. Staff morale is measured as lowest in the region.
- The $80M target was given to the CFO by the board. The CFO does not know how the board arrived at that number.
- We have 8 weeks and a team of 4 consultants.

Do NOT give me a generic cost reduction framework. Tell me specifically: what are the political and structural landmines in this specific engagement that will derail the work before the analysis is complete? What does the CEO's opposition mean for how I should structure my first two weeks?"

**What DERIVE extracts:**

**Landmine 1 — The $80M number has no foundation:**

The board gave the CFO a number without a diagnostic. This is one of the most dangerous engagement setups in consulting. When a target precedes the analysis, two things happen: the team unconsciously shapes the analysis to reach the pre-determined number, and when the analysis finds that $80M is not achievable without destroying clinical capacity, nobody has the political standing to say so because the board has already anchored to the number.

Your first deliverable should be a hypothesis about where the $80M could come from before you do the analysis. If your quick scan of the cost structure suggests $40-50M is achievable without clinical cuts and $80M requires clinical cuts, you need to surface this to the CFO in week 2 — not week 8. Finding this out in week 8 means your recommendations arrive when there is no time to reset expectations.

**Landmine 2 — CEO opposition transforms the engagement from consulting to politics:**

A CEO who opposed the engagement will not implement recommendations that come from it. This is not about the quality of the analysis. It is about organizational authority. If the recommendations require operational changes — and cost reduction always requires operational changes — the COO, CMO, and operational leaders will take their cues from the CEO, not the CFO. Your recommendations will be "studied" indefinitely.

Before structuring the engagement, you need to understand why the CEO is opposed. Three possibilities: the CEO thinks the $80M target is wrong and this engagement will produce unfair cuts. The CEO has a competing internal initiative. The CEO and CFO have a power struggle that this engagement has become a proxy for.

Your week 1 should include a conversation with the CEO that is not framed as "we need your buy-in." It should be framed as "we want to understand your perspective on the challenges facing the system before we begin analysis." If the CEO's concerns are analytical (the target is wrong), you can address them. If they are political (this is a power struggle), you need to tell the CFO that the engagement has a structural problem that must be resolved before week 3 or the recommendations will not be implemented.

**Landmine 3 — The union contract timeline changes everything:**

14 months until contract expiration. Any recommendation that touches nursing staffing ratios, shift structures, or compensation will be interpreted by union leadership as a position being taken before contract negotiations. This is not irrational — it is accurate. Cost reduction recommendations about clinical staffing ARE positions in labor negotiations whether you intend them to be or not.

This does not mean you avoid clinical staffing. It means: anything you recommend about clinical staffing needs to be scoped as a discussion about contract negotiation strategy, not presented as an operational recommendation that happens to affect contract terms. The CFO needs to make a decision about what to do with clinical staffing recommendations before you put them in a deliverable.

**Landmine 4 — Prior consultant trauma changes how every finding will be received:**

When you present a finding that touches workforce, every person in the room will remember what happened 3 years ago. Your analysis will be filtered through "they are recommending layoffs again." This is not a communication problem — it is not solved by better slide design or more empathetic language.

The structural response: your engagement team should include someone who talks to frontline staff, not just senior leadership. The analysis should distinguish between "reducing waste that burdens clinical staff" and "reducing clinical staff." If your cost reduction includes genuinely efficiency-improving changes that frontline nurses and staff would actually want (reducing administrative burden, eliminating redundant documentation requirements), leading with those builds credibility for the harder recommendations.

---

**Round 2 — R (Reach the long tail on healthcare-specific cost structure)**

**DERIVE Round 2:**

"Your Round 1 response addressed the political landmines well. Now I need to understand the healthcare-specific cost structure before I begin the analysis.

You are a consultant who has done hospital cost reduction engagements specifically — not generic cost reduction, not manufacturing cost reduction, hospital systems specifically. You have been in the room when clinical leadership pushed back on recommendations with 'you don't understand how hospitals work.' You know which cost reduction levers are genuinely available in a regional hospital system and which ones sound good in a slide but cannot be implemented.

Walk through: in a $1.2B hospital system losing $45M, where do the real cost reduction opportunities exist that are actually implementable? Not the textbook categories — the specific levers that have been successfully pulled in comparable systems. And tell me which commonly recommended levers look good on paper but consistently fail in implementation."

**What Round 2 extracts:**

**Real levers that work in comparable systems:**

**Supply chain and purchased services (typically 15-20% of total cost):**

This is where hospitals consistently leave money. A regional 3-hospital system almost certainly does not have centralized GPO (Group Purchasing Organization) utilization. Different hospitals in the system are likely on different contracts for the same items — sometimes the same vendor at different prices. Supply chain standardization and contract renegotiation is the highest-ROI, lowest-political-resistance lever in hospital cost reduction. Typical achievable savings: 8-12% of supply chain spend. On a $1.2B system where supply chain is typically 25-30% of cost ($300-360M), this is $24-43M. Achievable in 12-18 months without touching clinical staff.

**Revenue cycle and denial management:**

Hospitals lose 3-5% of potential revenue to claim denials that could be prevented or overturned. For a $1.2B system, 3% is $36M. This is not a cost reduction — it is revenue recovery. But it shows up as margin improvement identical to cost reduction. The implementation requires: hiring or contracting denial management specialists, investing in coding accuracy, and building a payer relations function that appeals denied claims systematically. This typically requires 6-12 month runway before cash impact is visible.

**Physician alignment and OR efficiency:**

Operating room inefficiency is the most expensive operational problem in most hospitals. Common issues: surgeon preference cards (the specific supplies each surgeon requests for each procedure) are not standardized, leading to massive supply variation costs. OR block scheduling is managed politically rather than by utilization data. First case on-time starts are below 70% at most systems, which cascades into overtime and case cancellations.

The catch: OR efficiency improvements require physician buy-in that cannot be mandated. Surgeons have medical staff privileges, not employment relationships in most systems. You can present data but you cannot force standardization. This is the lever that sounds great in a slide and stalls in implementation if physician leadership is not engaged as partners.

**Levers that consistently fail:**

**Length of stay reduction presented as cost reduction:**

Every hospital cost reduction engagement recommends reducing average length of stay. It is almost never implemented as recommended because: reducing LOS requires care coordination, case management, and post-acute placement capacity. If the post-acute system in the region is at capacity (common in regional markets), patients cannot be discharged earlier regardless of clinical readiness. And shorter LOS with the same fixed cost base does not reduce costs — it reduces revenue if beds are not backfilled.

**Workforce reduction in clinical areas:**

You already have data on this from 3 years ago. The structural problem: hospital nursing is already operating at staffing ratios that make further reduction a patient safety issue in many units. The legal exposure from patient safety incidents dwarfs any labor cost savings. Union contract complications aside, clinical workforce reduction in a 3-hospital system that is already below regional staffing benchmarks is a recommendation that will not survive contact with the CMO, CNO, or risk management.

---

**Round 3 — I (Force reasoning on the 8-week deliverable)**

**DERIVE Round 3:**

"Before structuring the 8-week workplan:

Step 1: Walk through what is actually achievable in 8 weeks of analysis with 4 consultants given the political complexity you described. What can be done to enough depth to recommend with confidence and what will be directional only?

Step 2: Walk through how I handle the situation where my analysis finds $50M in achievable savings but the board target is $80M. What is the specific recommendation structure that is intellectually honest about this gap while not destroying the CFO's position with the board?

Step 3: Walk through the CEO opposition problem specifically — at what point in the 8 weeks do I need a resolved CEO buy-in or do I need to tell the CFO that the engagement should pause?

Step 4: Give me the week-by-week workplan only after addressing all three."

**What Round 3 extracts:**

**Step 1 — What 4 consultants can actually do in 8 weeks:**

With 4 consultants across 3 hospitals, you cannot do primary research at the depth required to make precise recommendations in every category. You will have to be explicit about what is analysis-supported and what is hypothesis-requiring-validation.

What you can do with confidence: supply chain spend analysis (data is in ERP, 2 consultants can complete this in 3 weeks), revenue cycle analysis (billing data is accessible, denial rates can be calculated precisely), OR utilization analysis (schedule data exists). These three areas cover the highest-ROI levers and are data-intensive enough for confident recommendations.

What will be directional only: workforce productivity benchmarking (you can benchmark against published data but without detailed operational observation you cannot make specific workforce recommendations), capital and purchased services beyond the major categories (requires vendor-by-vendor analysis that 8 weeks cannot support).

This means your deliverable should explicitly separate "recommendations ready for implementation planning" from "hypotheses requiring a 90-day operational assessment before implementation."

**Step 2 — The $50M vs $80M gap:**

Never present this as "we found less than the target." Present it as a portfolio with time horizons: "$47M in high-confidence opportunities across supply chain, revenue cycle, and OR efficiency implementable in 12-18 months with no clinical staff impact. An additional $28M in opportunities in workforce productivity and purchased services that require a 90-day operational assessment to scope precisely and a 24-36 month implementation timeline given union contract considerations. Total portfolio: $75M, phased over 36 months, with $47M fundable in the first 18 months."

This reframes the gap from "we couldn't find $80M" to "here is how $75M gets implemented successfully rather than how $80M gets announced and then stalls." The board cares about implementation more than announcement. The CFO needs a way to present this that does not look like the engagement underdelivered.

**Step 3 — CEO opposition resolution timeline:**

Week 2 is the latest you can have the CEO conversation without risking the engagement. Here is why: by week 3 you are interviewing operational leaders. Those leaders will ask the CEO whether they should cooperate. If the CEO is still opposed, you will get surface-level access to operations — data but not insight, facts but not context. Your analysis will be technically complete and operationally shallow.

If the week 2 CEO conversation does not produce at minimum neutrality — the CEO agrees to let the engagement proceed without active opposition even if they are skeptical — you tell the CFO: "We have a structural problem. The recommendations will be accurate but unimplementable. We recommend the board convene a session with both the CEO and CFO to align on the mandate before we proceed to recommendations."

This conversation is the hardest one in consulting. Most consultants avoid it and deliver recommendations that sit on a shelf. The ones who have it are the ones who get called back.

---

## Scenario 4: Strategy Consulting — Market Entry Decision

**Full DERIVE Prompt:**

"I am advising a $200M revenue industrial software company that makes ERP software for mid-market manufacturing companies. They are evaluating entering the food and beverage manufacturing vertical. Currently they serve discrete manufacturing (automotive parts, electronics assembly, industrial equipment).

The CEO's hypothesis: food and beverage is adjacent to their current market, uses similar ERP capabilities, and is growing faster than discrete manufacturing.

What we know:
- Food and beverage has FDA compliance requirements (21 CFR Part 11, FSMA) that their current software does not support
- Their 3 largest competitors are already in food and beverage
- A private equity-backed competitor raised $150M last year, partly for food and beverage expansion
- The company has $40M in cash on the balance sheet
- Their current NPS is 67 — strong in discrete manufacturing

Do NOT give me a market entry framework. Tell me specifically:

1. What the CEO's 'adjacent market' hypothesis gets wrong about how ERP switching decisions work in food and beverage specifically

2. What the FDA compliance gap means commercially — is this a 6-month engineering problem or something more fundamental?

3. What the competitive timing signal (3 competitors already there, 1 just raised $150M) tells us about whether this is still a good time to enter

4. What the right first question to answer is before committing to this strategy

Walk through each before giving any recommendation."

**What DERIVE extracts:**

**Point 1 — What the adjacent market hypothesis gets wrong:**

Adjacency in product capabilities does not equal adjacency in buying behavior. Discrete manufacturing ERP is bought by operations managers and manufacturing engineers. The key pain points are: production scheduling, shop floor control, inventory management.

Food and beverage ERP is bought by a different buyer with different pain points. The primary buyer is often the VP of Quality or VP of Operations with a strong compliance orientation. Their primary concerns are: lot traceability (the ability to recall specific product batches), shelf life management, catch weight processing, and regulatory audit readiness. These are not "similar ERP capabilities" — they are fundamentally different data models and workflows.

The adjacency trap: because both markets use the word "ERP" and both involve manufacturing, the CEO is treating them as similar. The actual similarity is like saying "we make great tools for building houses, let's enter the boat building market because both use wood." The material is similar. The craft, the buyer, the requirements, and the competitive dynamics are different.

**Point 2 — What the FDA compliance gap means commercially:**

21 CFR Part 11 is not a feature you add to existing software. It is a set of requirements about how the software manages electronic records and signatures that touches the software architecture at a fundamental level: audit trails must be tamper-evident, user authentication must meet specific standards, the software must be validated using FDA-defined methodology.

The validation requirement is the commercial problem. Food and beverage manufacturers do not just buy software that has these features — they buy software that has been validated in their facility using FDA-approved validation protocols. This process takes 3-6 months per customer implementation and requires specialized consultants. This means even after you build the compliance features (12-18 months minimum), you need to build the implementation methodology and partner network that can execute FDA validation. The total time from decision to first reference customer is 24-36 months minimum.

FSMA (Food Safety Modernization Act) adds supply chain traceability requirements that went into full effect in 2026 and are creating significant compliance pressure in the industry right now. This is either an opportunity (customers are actively looking for solutions) or a threat (if you enter now, you are entering a market where customers are making ERP decisions under regulatory pressure on a timeline you cannot influence).

**Point 3 — What competitive timing tells you:**

Three competitors already there means the market is not being created — it is being competed for. The $150M PE raise is a more specific signal: PE-backed companies raise to accelerate growth, not to enter new markets. That company is already in food and beverage and is using the capital to expand sales capacity and acquire customers at below-rational pricing. You are entering a market where a well-capitalized competitor is willing to win deals at negative gross margin to build market share.

The timing signal says: the window for entering as a fast follower with a differentiated product closed approximately 18-24 months ago. The current window requires either: a differentiated approach that the three incumbents have not taken (possible — are they all serving the same tier of the market?), or an acquisition of a small existing player with installed customer base and FDA-validated software (bypasses the 24-36 month build timeline), or a partnership/OEM arrangement with an existing food and beverage compliance software company.

**Point 4 — The right first question:**

The right first question is not "should we enter food and beverage?" It is: "why are we losing to food and beverage-native competitors in deals with customers who have both discrete manufacturing and food and beverage divisions?"

If you serve automotive parts manufacturers, some of them also have food packaging divisions. Do those customers buy your ERP for both divisions or a different ERP for food and beverage? If they buy a different ERP for food and beverage, that is your real intelligence source. Those are customers who already trust you, already have your sales team relationships, and chose a competitor for a different division. The reasons they give you for that choice tell you more than any market analysis about what it would actually take to compete in food and beverage.

If that pattern does not exist — if none of your current customers have food and beverage divisions — then the adjacency is weaker than the CEO believes and the market entry requires more fundamental justification than "it seems similar."

---

## The DERIVE Loop for PM and Consulting — The Meta-Pattern

The pattern across all four scenarios:

**Round 1 (D):** State your specific situation with the numbers that make it specific, what you already tried, and what the generic answer would miss about your context. For PM: your actual retention rate, your actual survey data, your actual team size. For consulting: your actual client situation, the political context, the constraints that make generic frameworks inapplicable.

**Round 2 (E + R):** Take the most important assumption from Round 1 and test it against real data you have. Eliminate the assumption explicitly. Give the model the role of someone who has been in this exact industry situation — not a generic business expert, a specialist who has seen this specific failure pattern.

**Round 3 (I):** For PM: force the model to walk through competing hypotheses before concluding. For consulting: force the model to walk through what is achievable given constraints before recommending. The conclusion that comes after forced reasoning is qualitatively different from the conclusion that comes from pattern matching to the most common answer.

**Round 4 (V):** In PM and consulting the verification discipline is different. You cannot look up the answer in documentation. You verify by: checking the logic against your specific data, stress-testing the recommendation against the stakeholders who will push back, and asking what evidence would change the recommendation.

**Round 5 (E — Expand):** Every PM and consulting engagement reveals a deeper question under the surface question. The retention problem revealed a project lifecycle problem. The cost reduction engagement revealed a CEO opposition problem that precedes the cost analysis. Round 5 probes the deeper question that the analysis revealed but the original mandate did not include.

The fundamental difference from technical domains: in PM and consulting, the LLM's value is not retrieving specialized technical knowledge from the long tail. It is forcing rigorous thinking before conclusion in domains where social pressure, stakeholder management, and cognitive biases produce premature conclusions constantly. DERIVE is the structure that prevents the most common PM and consulting failure: answering the question that was asked instead of the question that matters.

---------

Let me build this as the most comprehensive before/after demonstration of DERIVE across both domains. Every example follows the same structure: the generic prompt, the generic response, the full DERIVE prompt, and what DERIVE actually extracts — showing the gap explicitly.

---

## Part 1: Product Management

---

### Scenario 1: You Are a PM at a B2B HR Tech Company Facing Sudden Churn

**The Situation:**

You are the PM for an HR analytics platform. Your product helps HR teams track employee engagement, turnover risk, and performance trends. 300 customers, $12M ARR, Series B startup. Your CEO just told you that three enterprise customers (combined $1.8M ARR) have sent cancellation notices in the same week. The board meeting is in 6 days.

---

**BEFORE DERIVE**

**Prompt:**
"Three of our enterprise customers just cancelled. How do I do a churn analysis and figure out what went wrong?"

**Response:**
```
Conduct exit interviews with churned customers.
Review product usage data to identify drop-off points.
Analyze support ticket history for recurring issues.
Compare churned customers to retained customers.
Build a health score model to predict future churn.
Use NPS scores to identify at-risk accounts.
Create a retention playbook for customer success.
```

This is the churn analysis curriculum from every SaaS playbook ever written. It is a list of activities with no connection to your specific situation. You could have read this in a blog post in 2018. It tells you nothing about why three enterprise customers cancelled in the same week, which is a very specific pattern with a very specific set of possible causes.

---

**AFTER DERIVE**

**Round 1 — D (Destroy the Default Path):**

"I am the PM for an HR analytics platform. Three enterprise customers cancelled in the same week. Combined ARR: $1.8M. This is not a gradual churn problem — three simultaneous cancellations is a specific event pattern.

What I know about the three accounts:
- Customer A: 2,400 employees, manufacturing company, been a customer 26 months, primary user is CHRO, cancellation letter cited 'strategic restructuring'
- Customer B: 800 employees, financial services firm, been a customer 8 months, primary user is VP People Analytics, cancellation letter cited 'consolidating our HR tech stack'
- Customer C: 3,100 employees, retail chain, been a customer 31 months, primary user is HR Director, no reason given in cancellation letter

What I know about recent product and company events:
- We shipped a major UI redesign 6 weeks ago
- Our pricing changed 4 months ago — enterprise tier increased 22%
- One of our large competitors announced a significant product update 8 weeks ago
- Our VP of Customer Success left the company 10 weeks ago and we have not replaced her

Do NOT give me generic churn analysis steps. For the specific pattern of three simultaneous enterprise cancellations with these specific characteristics, tell me: what are the three most likely root causes ranked by probability? For each root cause, tell me what specific evidence I would find in the next 48 hours that confirms or eliminates it. I have 6 days before a board meeting."

---

**What DERIVE extracts:**

**Root Cause 1 (Highest probability) — CS leadership departure created an invisible relationship gap:**

The VP of CS left 10 weeks ago. Enterprise customers at the $400k-$800k ACV level are not retained by the product alone — they are retained by the relationship. When a CS leader leaves, three things happen simultaneously that take 10-12 weeks to become visible as churn:

The regular executive business reviews stop or become lower quality. Nobody is proactively reaching out to the CHRO at Customer A or the VP People Analytics at Customer B with strategic conversation. The relationship shifts from partner to vendor.

The institutional knowledge about each account's strategic priorities leaves with the CS leader. The team member covering the account after the departure does not know that Customer B's VP People Analytics has been trying to justify the platform spend to a CFO who is skeptical. They do not know that Customer C's HR Director had three items on a wishlist that were being informally tracked.

New customer issues that previously would have been resolved through a phone call from the CS VP to the CHRO now sit in a support queue. The executive experience degrades without the executive-level relationship to absorb it.

**Evidence to find in 48 hours:** Pull the Salesforce/Gainsight activity logs for all three accounts. Find the date of the last executive-level touchpoint (call or meeting with C-level or VP-level contact, not just an end-user support ticket). If the last executive touchpoint for any of these accounts is more than 8 weeks ago, this is likely the primary cause.

**Root Cause 2 (Second probability) — The 22% price increase hit renewal conversations without adequate value justification:**

Customer A is 26 months old and Customer C is 31 months old. Both have gone through at least one renewal since your pricing change 4 months ago or are approaching renewal now. A 22% price increase at renewal requires a proactive value conversation before the renewal invoice arrives — the customer needs to be shown what they are getting for the additional cost before they see the number. Without the CS VP, that proactive value conversation likely did not happen.

Customer B at 8 months is not at renewal yet, which makes their "consolidating HR tech stack" reason more significant — they are leaving before their first renewal, which means they likely decided early in the customer lifecycle that the product was not delivering enough value to justify the price they signed at.

**Evidence to find in 48 hours:** Pull the renewal dates and the renewal conversation history for Customers A and C. Find when they first saw the new pricing. Find whether there was a value business review before they saw the new number. If the new pricing arrived in an invoice before anyone had a conversation about value, this is a contributing cause.

**Root Cause 3 (Third probability) — The UI redesign disrupted power users at a critical moment:**

Your most sophisticated enterprise users — the VP People Analytics at Customer B, the HR Director at Customer C — built their workflows around the old UI. A major UI redesign 6 weeks ago means they had to relearn the product at the same time they were evaluating whether to continue. Even if the new UI is objectively better, the disruption cost during an already uncertain evaluation period can tip a borderline account to cancellation.

"Consolidating HR tech stack" is frequently the language used when someone wanted to cancel anyway and needed a neutral, non-confrontational reason to give. The UI redesign may have been the proximate trigger for a latent dissatisfaction.

**Evidence to find in 48 hours:** Pull the product usage data for all three accounts specifically in the 6 weeks since the UI redesign. Look for: drop in daily/weekly active users, increase in support tickets about navigation, decrease in use of specific features that moved location in the redesign. If usage dropped measurably after the redesign, this is a contributing cause.

**The board meeting framing:**

You do not have enough information in 48 hours to give the board a definitive root cause. What you give them is: "We have identified three probable causes. We have preliminary evidence supporting [whichever cause your 48-hour investigation most supports]. We are running customer conversations this week and will have a confirmed root cause and retention plan within 3 weeks. Here is what we are doing immediately: [specific action for each probable cause]."

The board does not expect you to have solved it in 6 days. They expect you to demonstrate rigorous thinking and a credible path to understanding. DERIVE gives you that rigorous thinking.

---

**Round 2 — E + R:**

**Prompt:**
"Your Round 1 analysis identified CS leadership departure as the highest probability cause. I checked the activity logs. Last executive touchpoint for Customer A was 11 weeks ago. Customer B was 9 weeks ago. Customer C was 13 weeks ago.

You assumed we could recover these accounts if we act immediately. That assumption needs testing.

You are a SaaS customer success consultant who has managed enterprise account recovery specifically — not general CS strategy, actual recovery of accounts that have sent cancellation notices. You have been in the room when a recovery attempt succeeded and when it failed. What specifically determines whether a cancellation notice at this stage is recoverable versus final? What does the conversation need to look like in the next 72 hours to have any chance?"

---

**What Round 2 extracts:**

**The recovery determinants — what separates recoverable from final:**

A cancellation is recoverable when the decision was made by one person who was frustrated and the economic buyer has not yet been informed. In B2B HR tech, the primary user (CHRO, VP People Analytics) often sends the cancellation without full organizational alignment. If the CFO who approved the original contract does not know yet, there is a window.

A cancellation is effectively final when: the economic buyer has already approved it, procurement has been notified, and a replacement has been selected. At that point you are not recovering an account — you are negotiating a wind-down.

**The 72-hour conversation structure:**

Do not call the person who sent the cancellation notice first. Call one level above them. The person who sent the cancellation notice has already made their decision and defending it is now part of their professional identity. Calling them first puts them in a position where agreeing to stay means admitting their cancellation was wrong.

Call the CHRO at Customer A before calling whoever sent the letter. The conversation is not "we want to retain you." It is: "I want to make sure I understand what we could have done better and whether there is anything we should explore before this is finalized." This gives the executive an opportunity to either confirm the decision is final (which tells you it is truly final) or reveal that they were not fully aware of it (which gives you a recovery path).

The specific thing you are listening for in that first 5 minutes: does the executive know the specific cancellation was sent? Do they use the word "we" when discussing the decision or do they use a name? "We decided" means organizational alignment. "Sarah sent that because..." means there is a gap between the user decision and the executive decision.

---

**Round 3 — I:**

**Prompt:**
"Before giving me a retention action plan:

Step 1: Walk through what immediate retention action for these three accounts risks if done wrong. What is the failure mode of an aggressive recovery attempt?

Step 2: Walk through what systemic fix for the CS leadership gap looks like in the next 90 days — not the recovery, the prevention of the next three.

Step 3: Walk through what I tell the board versus what I tell the CEO. Are those the same conversation?

Step 4: Only after all three — give me the specific action plan with owners and timelines."

---

**What Round 3 extracts:**

**Step 1 — Risk of aggressive recovery:**

The failure mode of aggressive recovery in B2B SaaS is: you successfully convince the customer to stay through a combination of pricing concession and executive attention. Six months later they cancel again because the underlying issue (no CS relationship, unclear value, product gaps) was not fixed — it was papered over with a discount. The second cancellation is permanent and you have lost the discount too.

Aggressive recovery also sends a market signal. Enterprise customers talk to each other. If Customer A learns that Customer C got 30% off renewal by sending a cancellation notice, your entire enterprise renewal process is compromised. Every renewal negotiation from this point forward will include a trial cancellation notice as a negotiating tactic.

Recovery with integrity looks like: fixing the actual problem (assigning a dedicated senior CS resource to each account, scheduling an executive business review within 30 days, addressing specific product gaps that were raised), and offering pricing flexibility only if you can honestly say the price increase was not justified by value delivered.

**Step 2 — Systemic fix for CS gap:**

The 90-day priority is not hiring a new VP of CS. It is: identifying your top 20 accounts by ARR, assigning a named senior resource to each one this week, and running an emergency account health review for each one using the same 48-hour evidence gathering you just did for these three. You do not have a three-account problem. You have a CS coverage gap that affects every enterprise account. The three cancellations are the visible manifestation of an invisible risk across your entire enterprise base.

**Step 3 — Board vs CEO conversation:**

These are not the same conversation and conflating them is a common PM mistake.

The board conversation is about: financial exposure ($1.8M ARR at risk, what does full enterprise churn look like as a worst case), root cause hypothesis, and confidence level in the recovery and prevention plan. The board is a governance body. They need to know the exposure is bounded and the management team has a credible plan.

The CEO conversation is about: what specifically went wrong in the organization (the CS leadership gap, the pricing change execution, the UI redesign without enterprise user communication), who is accountable for fixing each thing, and what decisions need the CEO's involvement (interim CS leadership, pricing concession authority, escalation calls with enterprise executives). The CEO needs to own the org-level fixes. You can own the product-level fixes.

Going to the board with the same level of operational detail you give the CEO is a mistake — it suggests you do not understand governance versus operations. Going to the CEO with the same high-level framing you use for the board is a mistake — it suggests you do not understand your own execution gaps.

---

### Scenario 2: Roadmap Prioritization — The Feature Request That Could Change Everything

**The Situation:**

You are PM for a project management SaaS tool. Your largest customer (8% of ARR) has requested an AI-powered resource allocation feature. Your engineering team says it will take 4 months. Three other customers have mentioned wanting something similar. Your CEO wants it built immediately because "AI is the future." Your Head of Engineering says it will require significant architectural changes. You have a quarterly planning meeting tomorrow.

---

**BEFORE DERIVE**

**Prompt:**
"Our largest customer wants an AI resource allocation feature. Should we build it? How do I decide?"

**Response:**
```
Validate the request with multiple customers before building.
Assess strategic alignment with product vision.
Estimate engineering effort and opportunity cost.
Consider build vs buy vs partner options.
Use a prioritization framework like RICE or weighted scoring.
Evaluate competitive landscape for this feature.
```

You already knew all of this. None of it tells you what to actually do tomorrow in the planning meeting.

---

**AFTER DERIVE**

**Full DERIVE Prompt:**

"I am PM for a project management SaaS tool. $15M ARR, 450 customers. Our largest customer, Meridian Corp ($1.2M ARR, 8% of total ARR), has formally requested an AI-powered resource allocation feature. They submitted this 3 weeks ago. Their Head of PMO said in a call last week: 'This is not a nice-to-have. If we don't see this on the roadmap by Q3 we will evaluate alternatives.'

Additional context:
Three other customers have mentioned 'better resource management' in NPS comments in the last 6 months — but none have asked for AI specifically and none have threatened to leave.
Our CEO said in the leadership meeting: 'We need to be an AI company. Build it.'
Our Head of Engineering estimate: 4 months of a 3-person team, requires architectural changes to our data model that will affect 2 other planned features.
Our Head of Sales said: 'I've lost 2 deals in the last quarter where the prospect asked about AI capabilities.'
Current quarterly plan: 6 features committed, 0 delivered yet this quarter (we are 3 weeks in).

Do NOT give me a prioritization framework. Answer these specific questions:

1. What does 'we will evaluate alternatives' from a customer representing 8% of ARR actually mean in B2B SaaS — is this a negotiating tactic, a genuine threat, or something else? How do I find out which it is before tomorrow's meeting?

2. The CEO said 'build it' and the Head of Engineering said it breaks other things. What is the specific conversation I need to have with each of them before the planning meeting and what is the decision I am asking each of them to make?

3. The 3 NPS comments about 'better resource management' and the 2 lost sales deals — are these signal or noise for this specific feature request?

4. What is the worst decision I could make in tomorrow's planning meeting and why?"

---

**What DERIVE extracts:**

**Question 1 — What 'evaluate alternatives' really means:**

In B2B SaaS, "evaluate alternatives" from a customer at renewal distance means one of three things and you need to know which before tomorrow:

**Negotiating tactic (most common):** The Head of PMO needs to show their CEO they pushed for a feature commitment before signing the renewal. They will renew regardless, but they want something to point to. Indicator: Meridian Corp is not currently in an active evaluation with a competitor, and the feature request came with a specific deadline rather than a specific alternative named.

**Genuine evaluation beginning (serious but recoverable):** Meridian has started talking to competitors but has not made a decision. The "evaluate alternatives" language is a warning shot. Indicator: they asked for the feature 3 weeks ago and have not followed up since. If they were genuinely evaluating, you would expect more urgency.

**Decision already made (potentially unrecoverable):** They have already talked to a competitor and are in a demo or proof of concept. The "evaluate alternatives" language is courtesy notice. Indicator: the Head of PMO's language has shifted from "we want this" to "we need this" with a hard deadline attached.

**How to find out before tomorrow:** Call the Head of PMO today — not email. The question is not "are you looking at competitors?" It is: "I want to make sure I understand the full scope of your resource management challenge before we finalize our planning. Can you walk me through what a perfect solution looks like for you?" If they describe a specific competitor's feature, they are in evaluation. If they describe a business problem, they are still in the negotiating tactic phase. This one 20-minute call changes how you present everything tomorrow.

**Question 2 — The CEO and Engineering Head conversations:**

The CEO conversation is not about whether to build it. The CEO said build it because they are responding to a pattern (AI competitive pressure, large customer threat) not a specific analysis. The decision you are asking the CEO to make is: "Given that building this feature now delays these two other committed features by 6-8 weeks and costs us $X in engineering opportunity cost, do you want me to make that tradeoff explicitly or find an alternative path?" You are not overriding the CEO — you are forcing a tradeoff decision that the CEO needs to own, not you.

The Engineering Head conversation is about scope, not timeline. The architectural change concern needs to be specific: which other planned features are affected, what is the nature of the dependency, and is there a version of this feature that delivers value to Meridian without the architectural change (a "paved path" version that uses existing data model and gets 60% of the value in 6 weeks instead of 4 months at full capability). Engineering leads often present architectural concerns as binary when they are actually a spectrum of implementation approaches.

**Question 3 — Signal or noise from NPS and lost deals:**

The 3 NPS comments about "better resource management" are weak signal for this specific feature. They validate the problem space (resource management is painful) not the solution (AI-powered allocation). Building an AI feature to satisfy customers who said "better resource management" risks building the wrong solution for a real problem. Before this becomes a roadmap argument, someone needs to talk to those 3 NPS respondents and find out what "better" means to them specifically.

The 2 lost sales deals are more concerning but the evidence is ambiguous. Sales lost deals where "prospect asked about AI capabilities" could mean: the prospect was specifically evaluating AI resource allocation (relevant to this feature), or the prospect asked a general "do you have AI?" question during due diligence and your sales team did not have a good answer (not necessarily about this feature at all). Ask your Head of Sales: what specific AI capability did each prospect ask about, and did they go to a competitor who had it?

**Question 4 — The worst decision tomorrow:**

The worst decision is committing to build the full 4-month feature in Q3 in the planning meeting tomorrow under pressure from CEO urgency and customer threat without verifying whether the threat is real and without exploring the smaller faster alternative.

Why it is the worst decision: you commit to 4 months of a 3-person team, you defer 2 other features that affect a broader set of customers, you create an expectation with Meridian that a 4-month feature will be delivered (and when it slips to 5-6 months, you have not retained them — you have given them a new reason to leave), and you have traded certain near-term cost for uncertain benefit.

The second worst decision is saying "we won't build it" in the planning meeting without an alternative that preserves the Meridian relationship and addresses the competitive gap.

The right decision tomorrow is not a final roadmap commitment. It is: "We are doing a 2-week discovery sprint on this feature before committing. We will interview Meridian and the 3 NPS respondents, assess the technical path options, and present a recommendation at the week-3 check-in." This gives the CEO action, gives Meridian responsiveness, and gives Engineering the space to find a smaller faster path.

---

## Part 2: Management Consulting

---

### Scenario 3: Strategy Engagement — Retail Client Facing E-Commerce Disruption

**The Situation:**

You are a management consultant engaged by the CEO of a mid-size regional grocery chain. 45 stores, $2.1B revenue, family-owned for three generations. The third-generation CEO hired you because Amazon Fresh opened 6 locations in their market 18 months ago, and same-store sales have declined 3.1% in the last 12 months. The engagement scope is: "develop a strategy to respond to e-commerce disruption."

---

**BEFORE DERIVE**

**Prompt:**
"How do I develop a strategy for a regional grocery chain facing e-commerce disruption from Amazon Fresh?"

**Response:**
```
Invest in your own e-commerce and delivery capabilities.
Differentiate on local products and community connection.
Optimize store operations for efficiency.
Develop a loyalty program to increase customer retention.
Consider click-and-collect as a lower-cost alternative to delivery.
Focus on fresh and prepared foods where Amazon is weaker.
Partner with Instacart for delivery infrastructure.
```

This is the grocery industry strategy consensus from every retail consulting firm's published thought leadership. The CEO has already read all of this. It is the answer to a different company's problem.

---

**AFTER DERIVE**

**Round 1 — D:**

"I am a management consultant engaged by the CEO of a regional grocery chain. 45 stores, $2.1B revenue, family-owned third generation. Amazon Fresh opened 6 locations in their market 18 months ago. Same-store sales declined 3.1% in the last 12 months.

What I know about the specific situation:
Their 45 stores are in three geographic clusters: 22 stores in suburban markets, 15 stores in smaller towns (populations 15,000-40,000), and 8 stores in urban neighborhoods.
The 3.1% same-store sales decline is not uniform: suburban stores declined 5.8%, small town stores declined 0.9%, urban stores declined 4.2%.
Amazon Fresh's 6 locations are all in the suburban cluster.
Gross margin has held steady — the sales decline is volume, not price erosion.
Their private label penetration is 14% (industry average for regional chains is 22%).
They have 280,000 loyalty card members but the loyalty program has not been updated since 2011.

The CEO's framing: 'We need to beat Amazon at e-commerce.'

Do NOT give me a generic omnichannel strategy. Answer these specific questions:

1. What does the store-cluster performance data tell me about whether Amazon Fresh is actually the problem? What alternative explanations does this data pattern suggest?

2. What does 'beat Amazon at e-commerce' as a strategic frame tell me about what the CEO is worried about versus what the actual threat is?

3. The 14% private label penetration versus 22% industry average — is this relevant to the Amazon threat or is it a separate issue?

4. Before I develop any recommendations, what is the single most important thing I do not yet know that could completely change the strategy?"

---

**What DERIVE extracts:**

**Question 1 — What the cluster data actually says:**

The data pattern is highly specific: suburban stores declined 5.8% and Amazon Fresh is in the suburban cluster. Small town stores declined only 0.9% and Amazon Fresh has no small town locations. This is consistent with Amazon Fresh being the cause of suburban decline.

But here is what the data also shows: urban stores declined 4.2% with no Amazon Fresh competition. Urban store decline without an Amazon competitor suggests a second cause of decline that is not Amazon-related. The urban decline could be: demographic shift in their urban trade areas, increased competition from specialty grocers (Trader Joe's, Whole Foods, local independents), reduced foot traffic from remote work reducing downtown lunch/dinner shopping, or a quality/assortment issue specific to their urban stores.

The strategic implication is significant: if you design a strategy entirely around "beating Amazon," you fix the suburban problem (maybe) while doing nothing about the urban problem. After 18 months of strategy execution you have improved suburban stores and urban stores are still declining. The CEO concludes the strategy failed. The strategy did not fail — it solved the wrong problem.

**Question 2 — What 'beat Amazon at e-commerce' reveals:**

"Beat Amazon at e-commerce" is a fear response dressed up as a strategic frame. No regional grocery chain with $2.1B revenue will beat Amazon at e-commerce. Amazon's fulfillment infrastructure, technology investment, and Prime membership base are structural advantages that cannot be overcome by a 45-store regional chain regardless of strategy.

What the CEO is actually worried about is not losing the e-commerce battle. They are worried about losing their customers — specifically, the question is whether the customers who went to Amazon Fresh are gone permanently or whether they are shopping Amazon Fresh for specific categories while still shopping the regional chain for others.

This distinction completely changes the strategy. If Amazon Fresh took customers entirely (customers switched their primary grocery shopping), the problem is share-of-stomach and the strategy is about winning back primary grocery shoppers. If Amazon Fresh took customers partially (customers now buy certain categories — packaged goods, household supplies — at Amazon while still buying fresh and local at the regional chain), the problem is basket size and the strategy is about strengthening the categories Amazon cannot replicate.

The CEO needs to be asked: "When you say beat Amazon, what does winning look like in 3 years? What number changes?" If the answer is "reverse the same-store sales decline," that is a different strategy than "build an e-commerce platform." The question exposes whether the CEO is solving for fear or for a specific business outcome.

**Question 3 — Private label penetration relevance:**

14% versus 22% industry average is directly relevant to the Amazon threat and here is why: Amazon Fresh's weakness is private label. Amazon's grocery private label is national-scale commodity products. A regional chain's private label can be locally sourced, regionally specific products that Amazon structurally cannot replicate.

Private label is also the highest-gross-margin category in grocery — typically 8-12 points higher margin than national brands. Going from 14% to 22% private label penetration on $2.1B revenue is approximately $25-35M in additional annual gross profit at current revenue levels. This is a margin opportunity that exists independent of the Amazon competitive response and that accelerates the financial position to invest in whatever the strategy turns out to be.

The private label gap is not a separate issue — it is likely a root cause of why the competitive response options feel constrained. A chain with 22% private label has more gross margin to invest in customer experience, price competitiveness, and technology than a chain at 14%.

**Question 4 — The single most important unknown:**

The most important unknown is: what do the loyalty card members who reduced their purchase frequency in the last 18 months look like compared to those who did not? The 280,000 loyalty card members contain the answer to whether Amazon is taking full customer relationships or partial customer relationships.

Specifically: of members who shopped 40+ times per year two years ago, how many now shop fewer than 25 times per year? And for those customers, what categories disappeared from their baskets? If packaged goods disappeared but fresh produce stayed, Amazon is taking the commodity trip and the regional chain is holding the fresh trip. If the entire basket shrank proportionally, Amazon is taking the full customer relationship.

This one analysis from the loyalty data changes the entire strategic framing. It is 2 days of data work. It should happen before any strategy recommendation is presented.

---

**Round 2 — E + R:**

"We ran the loyalty data analysis. Results:

Of 280,000 loyalty members, 41,000 members who were 40+ trip customers 2 years ago are now 18-24 trip customers. That is 14.6% of loyalty members who significantly reduced frequency.

For those 41,000 members: their basket category breakdown shows packaged goods purchases dropped 67%, household/cleaning dropped 71%, personal care dropped 58%. Fresh produce purchases dropped only 12%, fresh meat dropped 9%, prepared foods dropped 8%, deli dropped 6%.

Interpretation: these customers are still buying fresh and prepared foods from this chain but buying packaged and household goods from Amazon or another channel.

You assumed this is primarily an Amazon Fresh problem. This data pattern actually matches Amazon Prime/online shopping behavior more than Amazon Fresh store behavior. Amazon Fresh stores compete on fresh. Amazon.com competes on packaged goods and household.

You are a retail strategy consultant who has analyzed grocery consumer behavior shifts specifically in markets where both Amazon Fresh physical stores AND Amazon Prime delivery competed simultaneously with regional chains. What does this specific basket fragmentation pattern tell you about which Amazon product is actually causing the damage, and what does that change about the strategic response?"

---

**What Round 2 extracts:**

**The actual competitive threat is not Amazon Fresh stores:**

The basket data is diagnostic. Fresh produce down only 12% while packaged goods down 67% is the signature of Amazon Prime grocery delivery taking the stock-up trip, not Amazon Fresh taking the daily fresh shopping trip. Amazon Fresh physical stores compete most directly on fresh, produce, prepared foods, and the in-store experience. Amazon Prime delivery competes most directly on packaged goods, household products, and anything that is shelf-stable and predictable.

The strategic implication: the CEO is worried about the 6 Amazon Fresh stores. The real threat is the Amazon Prime delivery behavior that existed before and accelerated during the pandemic and is structurally embedded in their customers' shopping habits. Amazon Fresh is the visible competitor. Amazon Prime delivery is the invisible competitor that is taking more volume.

**What this changes about the strategy:**

Fighting Amazon Fresh with an e-commerce platform is fighting the visible competitor with the wrong weapon. The customers who reduced frequency are still coming in for fresh, produce, deli, and prepared foods. They are choosing the regional chain for the categories where physical store experience, freshness, and local knowledge matter. They are choosing Amazon for the categories where price, convenience, and selection matter.

The right strategy is not to beat Amazon at packaged goods delivery. It is to: deepen the advantage in the categories where customers are already choosing the regional chain (fresh, prepared, local, deli), reduce investment in the categories where Amazon has a structural advantage (packaged goods, household), and use the private label opportunity to replace margin lost from the packaged goods volume decline.

The practical implication: stop competing on packaged goods price with Amazon. Reduce shelf space for national brand packaged goods (a low-margin category you are losing anyway), expand fresh, prepared foods, and local/regional products (high-margin categories where you are winning), and invest the saved space in an in-store experience that Amazon Fresh cannot replicate (cooking demonstrations, local farmer relationships, prepared meal programs).

---

**Round 3 — I:**

"Before recommending this strategy to the CEO:

Step 1: Walk through how I tell a third-generation family business CEO that the strategy is 'stop competing on packaged goods' when their stores have always stocked a full range. What is the emotional and organizational reality of this recommendation?

Step 2: Walk through the financial model for reducing packaged goods shelf space and expanding fresh and prepared — what are the risks to revenue during the transition period?

Step 3: Walk through what the 15 small-town stores should do differently than the 22 suburban stores. The data shows small-town stores are largely unaffected — should the strategy be the same for both?

Step 4: Walk through what the CEO means by 'beat Amazon at e-commerce' — do I need to address this frame directly in my recommendation or can I redirect without confronting it?

Step 5: Only after all four — give me the recommendation structure for the final presentation."

---

**What Round 3 extracts:**

**Step 1 — Telling a family business to abandon a category:**

Third-generation family businesses have identity wrapped up in their operational model. "We have always been a full-service grocer" is not just a strategy — it is a family legacy statement. Telling the CEO to reduce packaged goods shelf space will be heard as "abandon what your grandfather built."

The framing that works: "Your customers are telling us through their shopping behavior where they trust you most and where they need you most. They are choosing you for fresh, local, and prepared foods at the same rate they always have. The question is whether to fight for categories where you are losing ground or to double down on the categories where your customers are actively choosing you. Every retailer who has thrived through Amazon competition did it by becoming the best in the world at what Amazon cannot do, not by trying to match what Amazon does best."

You are not asking them to abandon categories — you are asking them to allocate space and investment toward their strength. The emotional reframe matters more than the analytical argument.

**Step 2 — Financial transition risk:**

The packaged goods to fresh/prepared transition has a specific financial risk profile: packaged goods shelf space is highly efficient from a labor and shrink perspective. Expanding fresh and prepared foods increases labor cost (prepared foods requires cooks, not stockers), increases shrink (fresh product has waste that packaged goods do not), and requires capital investment in store infrastructure (expanded deli cases, food preparation equipment).

The revenue during transition is not at risk — the transition is moving space from a declining category to a growing one. The margin during transition is at risk because prepared foods at scale has lower margin initially until you optimize the program. A typical prepared foods expansion takes 12-18 months to reach target margin, during which the category runs at 2-3 margin points below target.

The mitigation: phase the transition store by store starting with the suburban stores where the Amazon Fresh competition is most acute and where the customer base has the highest income demographics that support prepared foods pricing.

**Step 3 — Small-town stores need a different strategy:**

The small-town stores declining only 0.9% with no Amazon Fresh competition are a fundamentally different competitive situation. Their threat is not Amazon Fresh — their threat is the customer who drives 30 minutes to a suburban Walmart or orders online and has groceries delivered. Their strength is convenience (they are the closest grocery) and community relationship (they know the customers by name).

Small-town stores should NOT get the same prepared foods and fresh expansion strategy. Their strategy is: maintain the full-service model that serves as the community grocery (a small town does not need a gourmet prepared foods section), invest in the digital capability that prevents the 30-minute-drive trip (a simple click-and-collect that lets customers order online and pick up at the small-town store), and deepen community relationships (local sourcing, events, donation programs that make them the community anchor not just a grocery store).

Two different strategies for two different competitive situations within the same company. The CEO needs to accept that a single enterprise-wide strategy will not be the right answer.

**Step 4 — Addressing the "beat Amazon" frame:**

You cannot ignore it and redirect. The CEO will notice that you did not answer their strategic brief. You have to address the frame directly and with respect — then redirect.

The specific language: "You asked us to develop a strategy to beat Amazon at e-commerce. After six weeks of analysis, we believe the more important question is: how do you win the customers Amazon cannot serve as well as you? The data shows your customers are still choosing you for 88% of their fresh food purchases while shifting packaged goods online. The opportunity is not to fight Amazon on their strongest terrain — it is to become irreplaceable on yours. Here is what that looks like."

This acknowledges the frame, explains why you are redirecting, and leads with what they gain rather than what they are conceding.

**Step 5 — Recommendation structure:**

Structure the final presentation in four sections:

Section 1 — "What the data tells us" (15 minutes):
The loyalty analysis showing basket fragmentation by category. The cluster analysis showing Amazon Fresh vs Amazon Prime as different threats. The private label margin opportunity. This section is entirely data and observation. No recommendations yet. The CEO needs to see the diagnosis before they will hear the prescription.

Section 2 — "Where you are winning" (10 minutes):
Fresh, prepared, deli, local. Customers are actively choosing you in these categories at the same rate as two years ago. This is your competitive foundation. This section is designed to shift the emotional frame from "we are losing to Amazon" to "we are winning where it matters."

Section 3 — "The strategic choice" (20 minutes):
Present two paths explicitly, not just one recommendation. Path A: invest in e-commerce to compete in packaged goods (the CEO's original frame). Estimated cost, estimated likelihood of success, financial model. Path B: double down on fresh/prepared/local advantage while accepting packaged goods share loss (your recommendation). Estimated cost, estimated financial outcome, financial model. Let the CEO see both. The contrast makes Path B more compelling than presenting it alone.

Section 4 — "Where to start" (15 minutes):
The 90-day actions that are true regardless of which strategic path they choose: private label expansion (beneficial in both scenarios), loyalty program modernization (necessary in both scenarios), and the suburban store pilot of fresh/prepared expansion. Starting with actions that are strategy-agnostic builds momentum without requiring full commitment to a strategic direction that the CEO may need two weeks to process.

---

### Scenario 4: Organizational Consulting — Post-Merger Integration Failure

**The Situation:**

You are called in 8 months after a merger between two mid-size professional services firms. Combined firm: 1,200 employees, $180M revenue. The integration is failing. Revenue is down 11% since the merger. Three senior partners from the acquired firm have resigned. The CEO (from the acquiring firm) says "culture isn't meshing." The CFO says "we overpaid and the synergies aren't materializing." The CHRO says "people don't know who to report to."

---

**BEFORE DERIVE**

**Prompt:**
"How do I help a merged professional services firm that is struggling with post-merger integration?"

**Response:**
```
Conduct cultural assessment of both legacy organizations.
Define the combined firm's values and culture.
Clarify reporting structures and governance.
Identify and retain key talent.
Communicate transparently and frequently.
Align on combined firm strategy and client approach.
Create integration task forces.
```

Eight months after a merger this is crisis management, not integration planning. These suggestions would have been right 8 months ago. They are the wrong prescription for where this organization actually is.

---

**AFTER DERIVE**

**Full DERIVE Prompt:**

"I am a management consultant called in 8 months after a merger of two professional services firms. Combined: 1,200 employees, $180M revenue. Revenue down 11% since merger.

Specific symptoms I have observed in the first two weeks on the ground:

Senior partners from the acquired firm have stopped bringing new clients to the combined firm. When I asked one directly, she said: 'Why would I bring a client here when I don't know if the firm will honor the commitment I make to them?'

The three senior partners who resigned in the last 3 months all resigned within 2 weeks of specific events: one resigned after being passed over for a client leadership role that went to someone from the acquiring firm with less experience in that sector. One resigned after a compensation structure change that reduced the revenue share for partners who brought clients through legacy acquired-firm relationships. One resigned after a client they had managed for 7 years was reassigned to a partner from the acquiring firm without their input.

Revenue is down 11% but the revenue decline is almost entirely in the acquired firm's legacy client relationships. The acquiring firm's client relationships are flat.

Clients of the acquired firm have started calling the managing partners directly and asking: 'Is [specific partner they work with] still going to be my relationship manager?'

Do NOT give me integration best practices. Tell me specifically:

1. What is the actual problem I am dealing with? The CEO says 'culture isn't meshing.' The CFO says 'synergies aren't materializing.' The CHRO says 'reporting structure confusion.' Which diagnosis is right?

2. The three resignation triggers are very specific — what do they reveal about how the acquiring firm has managed the integration that created this situation?

3. The senior partner who said 'I don't know if the firm will honor the commitment I make' — what does this tell me about the psychological contract between the acquired firm's partners and the combined firm?

4. Is this recoverable in 90 days or is this a managed decline situation?"

---

**What DERIVE extracts:**

**Question 1 — What is the actual problem:**

All three diagnoses (culture, synergies, structure) are symptoms. The actual problem is a breach of psychological contract with the acquired firm's partners executed through a series of specific decisions in the first 8 months.

In professional services mergers, value lives in relationships — client relationships that partners hold and partner relationships within the firm. The acquiring firm treated the merger like a product merger (integrate systems, align reporting, communicate values) instead of a relationship merger (understand the informal commitments made during due diligence, protect the relationship capital that was the basis of the acquisition price, make the acquired partners' clients feel their relationships are secure).

The 11% revenue decline concentrated in legacy acquired-firm clients is diagnostic. The clients are not leaving because the combined firm has worse capabilities. They are leaving because the people they trusted are either leaving or signaling that they are not trusted by the new organization. In professional services, the client relationship is the revenue. If the relationship holder is uncertain about their own position, the client feels that uncertainty immediately.

**Question 2 — What the resignation triggers reveal:**

The three resignation triggers form a pattern that reveals a systematic bias in integration decisions toward protecting the acquiring firm's partners at the expense of the acquired firm's partners:

A client leadership role went to a less experienced person from the acquiring firm. This is an implicit message to every acquired-firm partner: your expertise is less valued than your origin. Every partner who witnessed this recalibrated their expectations of their own future in the combined firm.

A compensation structure changed to reduce revenue share for legacy acquired-firm relationships. This is a direct financial taking. The acquired-firm partners brought client relationships as their contribution to the merger. Reducing their share of those relationships' revenue is changing the economic terms of the deal after the deal closed.

A 7-year client relationship was reassigned without input. Client relationships in professional services are the professional identity of the partner. Reassigning without input is not an organizational decision — it is a statement that the acquired-firm partner's relationship is a transferable asset owned by the firm, not a professional relationship they built and steward.

Each of these decisions was probably made by the acquiring firm's leadership as a rational integration choice. Together they constitute a systematic stripping of the acquired firm's partners of the three things that made them valuable: their expertise recognition, their economic interest in their work, and their client relationships. The remaining acquired-firm partners are watching and concluding that the same will happen to them.

**Question 3 — The psychological contract breach:**

When the senior partner said "I don't know if the firm will honor the commitment I make," she is describing the collapse of trust at the foundation of professional services client relationships. In professional services, partners sell confidence. The confidence is: "I personally commit that this firm will deliver for you." When the partner does not trust her own firm to honor her commitments, she cannot sell that confidence. She stops bringing new clients not because she does not want to — because doing so would be professionally irresponsible. She would be making promises she cannot guarantee.

The psychological contract that was implicitly offered during the merger was: "Your relationships, your compensation structure, and your standing in the firm are protected. The merger makes you part of a larger platform — it does not diminish what you built." The three resignation triggers are specific violations of that implicit contract.

**Question 4 — Recoverable or managed decline:**

This is recoverable in 90 days if and only if: the CEO of the combined firm is willing to make three specific reversals that signal the psychological contract is being honored, and if the remaining senior acquired-firm partners have not yet made the private decision to leave.

The test for whether the remaining partners have privately decided to leave: are they still in client conversations? Are they still attending internal strategy meetings? Are they still introducing junior staff to their clients? If yes, they are still invested. If they are going through the motions, managing the transition of their client relationships, and updating their LinkedIn profiles — they have decided and the timeline to departure is 2-6 months.

The three specific reversals needed:
Reinstate or compensate the partner who lost the client leadership role. Not necessarily with the original role — with a role that signals their expertise is valued.
Review and adjust the compensation structure change. The message is: we are not going to change the economic terms of what you brought to this merger.
Establish a formal protocol that no client relationship managed by an acquired-firm partner can be reassigned without that partner's explicit agreement.

These are not culture interventions. They are specific contractual corrections. Culture follows trust. Trust requires specific behavioral evidence that the implicit contract is being honored.

---

## The Explicit Before/After Summary

Looking across all four scenarios, the pattern is precise:

**Before DERIVE:** The LLM answers the category of question (churn analysis, prioritization, grocery strategy, post-merger integration) with the canonical answer for that category. The answer is complete, structured, and useless for your specific situation because it was built for a generic version of your problem.

**After DERIVE:** The LLM is forced to engage with the specific evidence you provided — the cluster data, the basket fragmentation, the resignation triggers, the specific cancellation letters. It cannot give you the generic answer because the generic answer does not fit your specifics. It has to reason about your actual situation.

The quality gap is not subtle. Before DERIVE you get a consultant's slide deck outline. After DERIVE you get the insight that takes a consultant 3 weeks of on-the-ground work to develop — because you have compressed the context that normally develops over 3 weeks of interviews into a single well-constructed prompt.

This is what DERIVE actually does at its best: it substitutes context for time. You provide the context that an expert would normally gather over weeks. The LLM provides the analytical framework that connects that context to a specific, actionable insight. Neither alone is sufficient. Together they produce something that generic prompting cannot reach.
