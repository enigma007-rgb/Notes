https://www.infoq.com/articles/practical-design-patterns-modern-ai-systems/
--------

This is a rich, well-structured article by Rahul Suresh. Let me break it down in depth, category by category, step by step.

---

## Overview: Why This Article Matters

The article draws an analogy to the famous "Gang of Four" book — the 1994 classic that gave software developers 23 reusable object-oriented design patterns. Just as that book gave developers a shared vocabulary and playbook, this article argues we now need a **new vocabulary for AI-powered systems**. The problems are genuinely new: how do you control what an LLM says? How do you stop it from hallucinating? How do you make it cost-efficient at scale?

The author organizes the patterns into **five categories**, each addressing a distinct layer of concern as you build and scale an AI application. Let's go through each one deeply.

---

## Category 1: Prompting & Context Patterns

This is the foundation. Unlike traditional software where you write deterministic code, AI system behavior is shaped by the *instructions* (prompts) and *context* you give the model. Get this wrong and everything downstream suffers.

### 1a. Few-Shot Prompting

**The problem:** Without guidance, models produce inconsistent outputs — wrong tone, wrong format, wrong style.

**The solution:** Give the model 2–5 input-output examples directly inside the prompt before your actual query. You're essentially doing "on-the-fly training" without touching the model weights.

**Why it still matters today:** Modern frontier models (GPT-4, Claude) don't need examples to understand basic tasks. But few-shot prompting acts as a *personalization layer* — you can define your specific format, expected complexity, and domain vocabulary. It's also a powerful tool to reduce hallucinations by anchoring the model to concrete examples.

**Real-world use:** A customer support bot trained on 5 examples of how *your company* resolves complaints will sound far more on-brand than one given a generic instruction.

---

### 1b. Role Prompting

**The problem:** The model's default tone, assumptions, and content boundaries may not match your use case. A medical chatbot should not sound like a casual blog post.

**The solution:** Assign the model a persona or role via the system prompt (or the beginning of the user prompt). "You are a primary school teacher. Explain this concept simply." The same factual content gets delivered very differently depending on the assigned role.

**Deeper insight:** This pattern also serves as a **soft guardrail** — "You are a legal assistant. Only discuss topics relevant to contract law" implicitly constrains the model's topic range. Both Anthropic and OpenAI explicitly recommend this pattern in their prompt engineering guides.

---

### 1c. Chain-of-Thought (CoT) Prompting

**The problem:** For complex tasks (multi-step reasoning, logic, analysis), models tend to "shortcut" to an answer they've pattern-matched from training — and that answer is often wrong.

**The solution:** Instruct the model to *think step by step*, breaking the problem into intermediate reasoning stages before arriving at a conclusion. This mirrors how humans approach complex problems: we don't jump to answers, we deliberate.

**Why it works mechanistically:** By forcing the model to verbalize its reasoning chain, it becomes much harder for it to produce a superficially plausible but internally incoherent answer. Errors in logic are exposed in intermediate steps.

**Added benefit:** CoT outputs are more *interpretable* — you can see where the reasoning went wrong, which helps you debug and improve your prompts. It also significantly reduces hallucinations on factual tasks.

**Note on modern models:** Advanced reasoning models like Claude 3.7 and OpenAI o1 do CoT internally by default. But for smaller/older models, explicit CoT instructions are still critical.

---

### 1d. Retrieval-Augmented Generation (RAG)

**The problem:** LLMs have a training knowledge cutoff and no access to proprietary, real-time, or domain-specific data. A general-purpose model simply cannot know your company's internal policies, the latest legal statutes, or what happened in the news yesterday.

**The solution:** At query time, dynamically retrieve relevant documents (from vector databases, document stores, APIs) and inject them into the prompt context. The model then *reasons over the retrieved content* rather than relying solely on baked-in training knowledge.

**Architecture:** User query → retrieval system (vector similarity search) → top-k relevant documents → injected into prompt → model generates a grounded response.

**Why RAG is now industry standard:** It solves three problems simultaneously — knowledge cutoff, hallucination (by grounding answers in retrieved sources), and explainability (you can cite exactly which document the answer came from).

**Best for:** Customer support bots, legal/medical assistants, internal knowledge bases, any domain with rapidly changing or proprietary information.

---

## Category 2: Responsible AI Patterns

Prompting patterns improve accuracy and consistency but don't inherently make a system *safe* or *ethical*. Even a factually correct response can be biased, harmful, or legally problematic. This category adds the safety layer.

### 2a. Output Guardrails Pattern

**The problem:** Models, no matter how well prompted, can still produce toxic, biased, dangerous, or legally risky content. You cannot rely solely on the model's training to prevent this.

**The solution:** Implement a post-generation filtering layer — a set of rules, classifiers, or checks that intercept model output *before* it reaches the user. Guardrails can:

- Use rule-based filters (block specific keywords or patterns)
- Use lightweight ML classifiers to detect hate speech, bias, or toxicity
- Measure "groundedness score" — how well is the answer anchored in the source documents?
- Trigger a regeneration request with explicit warnings to the model about what to avoid

**Key insight:** Having your own guardrail layer, independent of whichever model provider you use, gives you **consistent safety behavior** regardless of whether you switch from GPT to Claude to Gemini tomorrow.

---

### 2b. Model Critic Pattern

**The problem:** A single model pass may contain subtle hallucinations, factual errors, or biases that a simple rule-based guardrail can't catch.

**The solution:** Introduce a second model in a "critic" or "judge" role. After the primary model generates a response, the critic evaluates it — checking for accuracy, bias, or policy violations — before the response is finalized or returned to the user.

**The analogy:** Think of a writer and an editor. The writer produces a draft; the editor catches errors the writer couldn't see in their own work.

**When to use it:** The author is pragmatic here — adding a critic model increases latency and cost, so it's not always practical in real-time production. However, it's extremely valuable in **offline QA pipelines**, where a larger, more powerful model judges outputs of a smaller, faster production model to catch regressions before deployment. GitHub Copilot uses this exact approach.

---

## Category 3: User Experience (UX) Patterns

After making your AI system safe and accurate, you need to make it *usable*. AI systems behave very differently from traditional software — they're probabilistic, sometimes wrong, often slow, and produce open-ended outputs. Users need specific patterns to interact with these systems effectively and comfortably.

### 3a. Contextual Guidance Pattern

**The problem:** Many AI tools launch assuming users will intuitively know how to use them. They don't. Users don't know what the tool can do, what it can't do, or how to phrase requests to get good results.

**The solution:** Surface prompt examples, tips, and feature overviews *at the right moment in the user journey*. Don't dump everything at onboarding — show guidance contextually. Notion's AI feature is cited as a great example: pressing spacebar in an empty doc triggers writing suggestions (because you probably want to write), while selecting existing text surfaces editing options.

**The principle:** Lower the activation energy to get a good first output. Users who see immediate value stick around.

---

### 3b. Editable Output Pattern

**The problem:** With generative AI, there's rarely one "correct" answer. The best output depends on user context and preferences. Treating AI output as final and immutable creates a black-box experience users distrust.

**The solution:** Let users directly modify, refine, or rewrite generated content. This shifts the mental model from "AI gives you an answer" to "AI gives you a starting point you can improve." GitHub Copilot lets you edit suggested code in your IDE. ChatGPT's Canvas mode lets you edit generated documents inline.

**The deeper value:** This pattern makes AI feel like a *collaborator*, not a vending machine. It also implicitly acknowledges that the model isn't always right, which builds more honest user trust.

---

### 3c. Iterative Exploration Pattern

**The problem:** Users often don't get what they want on the first try. If there's no path forward, they get frustrated and abandon the tool.

**The solution:** Build in regeneration, "try again", and variation mechanisms. For image generation, show multiple options simultaneously. For chatbots, allow follow-up refinements. Critically, allow users to *revert* to a previous generation if a later attempt is worse.

**Microsoft research finding cited in the article:** When users iterate many times, later attempts sometimes regress in quality. Letting users compare across attempts and pick the best one dramatically improves satisfaction.

---

## Category 4: AI-Ops Patterns

This is DevOps, but reimagined for AI systems. The core insight: in an AI system, your "application logic" lives not just in code but in **prompt-model-configuration triples**. These can change weekly, produce unpredictable outputs, and have no compiler to catch errors. You need disciplined operational practices to manage this.

### 4a. Metrics-Driven AI-Ops

**The problem:** Without measurement, you don't know if a prompt change or model upgrade is helping or hurting.

**The solution:** Track everything that matters: latency, token usage, cost per call, user acceptance rate, and AI-specific metrics like hallucination rate (measured via an LLM-judge pipeline). Set automated alerts when key metrics degrade.

**The key difference from traditional software monitoring:** Many of these metrics (e.g., hallucination rate, response quality) can't be measured with simple rule-based checks — you need ML-based evaluation pipelines. This is a new operational muscle teams need to build.

---

### 4b. Prompt-Model-Config Versioning

**The problem:** Untracked changes to prompts, model configurations, or model versions can silently break your system. A small prompt tweak that seemed harmless might tank response quality in edge cases you didn't test.

**The solution:** Treat every (prompt + model + configuration) combination as a *versioned software release*. Tag it. Run it against a golden test dataset. Use automated pipelines to evaluate quality before deploying. If quality degrades, roll back immediately.

**The analogy to traditional software:** This is essentially applying CI/CD discipline to your AI pipeline. The key addition is the need for a "golden dataset" of representative queries and expected outputs, which serves as your test suite.

The author also recommends applying standard DevOps practices (canary deployments, regression testing, rollback strategies) alongside these AI-specific ones.

---

## Category 5: Optimization Patterns

Once you're in production and traffic grows, you'll hit three walls: API rate limits, rising latency, and exploding inference costs. The core principle here is **don't use more compute than you need**.

### 5a. Prompt Caching

**The problem:** If your system prompt, few-shot examples, or system instructions are long and repeated in every request, you're paying to re-process the same tokens thousands of times per day.

**The solution:** Cache the expensive, static parts of your prompt (the "prefix" — system instructions, examples, boilerplate context). Many providers like Amazon Bedrock support this natively. The article cites up to 85% latency reduction on large prompts using prefix caching.

**Also cache full responses** for frequently repeated queries — the fastest and cheapest inference call is the one you never make.

---

### 5b. Continuous Dynamic Batching

**The problem:** Processing each query sequentially underutilizes GPU capacity, increases per-query cost, and makes it harder to scale.

**The solution:** Instead of processing requests one at a time, wait a short window (tens to hundreds of milliseconds) to accumulate several requests, then process them as a batch. This dramatically increases GPU utilization and system throughput.

**Trade-off:** Introduces a small latency overhead (the wait window). This is acceptable for most use cases and is worth the throughput and cost gains. Tools like vLLM, NVIDIA Triton Inference Server, and AWS Bedrock handle this out of the box.

---

### 5c. Intelligent Model Routing

**The problem:** Sending every single request — whether it's "What's 2+2?" or "Analyze the systemic risks in this 50-page financial report" — to your largest, most expensive model is wasteful and unsustainable.

**The solution:** Introduce a lightweight routing layer (analogous to a reverse proxy or API gateway) that classifies the complexity of each incoming query and routes it to the appropriately sized model:

- Simple/repetitive queries → pulled from cache or handled by a tiny model
- Moderate complexity → specialized mid-tier model
- Complex/ambiguous queries → full-power general-purpose model

**The insight:** This is essentially applying the *microservices* principle to model selection — right tool for the right job. It balances cost and quality dynamically, rather than making a static bet on one model for everything.

---

## The Three Advanced Topics (Briefly Mentioned)

The author flags three areas deliberately left out of scope but increasingly important:

**Fine-Tuning & Customization** — When off-the-shelf models aren't good enough, techniques like LoRA (Low-Rank Adaptation), knowledge distillation, mixture of experts (MoE), and quantization let you build smaller, cheaper, domain-specialized models.

**Multi-Agent Orchestration** — When tasks exceed what a single model can handle, multiple specialized agents collaborate. Patterns here include LLM-as-a-Judge, reflection loops, and role-based agent collaboration.

**Agentic AI** — Autonomous agents that plan, reason, and execute multi-step tasks independently using external tools. This is the fastest-evolving frontier and probably deserves its own article (the author acknowledges this).

---

## The Big Picture: How the Five Layers Build on Each Other

The five categories form a **layered architecture of concerns**:

1. **Prompting & Context** — Get the right answer in the right format
2. **Responsible AI** — Make the answer safe and ethical
3. **UX** — Make the interaction human and trustworthy
4. **AI-Ops** — Keep the system reliable and manageable over time
5. **Optimization** — Make it economically sustainable at scale

Each layer assumes the previous one is reasonably solid. You can't build a good UX on top of an unreliable, hallucination-prone system. You can't optimize costs effectively without operational metrics telling you what's expensive. The patterns aren't isolated tricks — they're an interdependent system.

The key contribution of this article is giving practitioners a **shared vocabulary and mental model** for AI system design — exactly what the Gang of Four did for object-oriented software three decades ago.


----------------

# AI Design Patterns — Deep Dive with Real-World Scenarios

Let me walk through every pattern using one consistent, realistic company: **"MediAssist"** — a healthcare startup building an AI-powered patient support and clinical assistant platform. This makes the patterns interconnected and concrete.

---

## The Company: MediAssist

MediAssist is building a web app where patients can ask health questions, get appointment help, understand prescriptions, and where doctors can get AI-assisted clinical summaries. They use an LLM via API (say, Claude or GPT-4). Let's build this system layer by layer, applying each pattern as real problems emerge.

---

## CATEGORY 1: Prompting & Context Patterns

---

### Pattern 1: Few-Shot Prompting

**The Problem That Triggers It**

MediAssist launches their symptom triage bot. Patients type in symptoms and the bot is supposed to categorize urgency as: "Emergency", "See a Doctor Soon", "Home Care", or "Monitor at Home".

Without examples, the bot returns wildly inconsistent labels:
- Sometimes it says "Urgent" (not in their system)
- Sometimes it says "This requires immediate medical attention" (a sentence, not a label)
- Sometimes it writes a paragraph instead of a category

Their downstream alert system breaks because it expects exactly one of four labels.

**How Few-Shot Prompting Fixes It — Step by Step**

Step 1: The engineer sits down and writes 4–5 real patient inputs paired with the exact correct label their system needs.

Step 2: They prepend these examples to every request. The prompt now looks like:

"Classify the urgency of the following patient symptom report. Use exactly one of these labels: Emergency, See a Doctor Soon, Home Care, Monitor at Home.

Patient: 'I have chest pain radiating to my left arm and I'm sweating heavily.'
Label: Emergency

Patient: 'I've had a mild sore throat for two days with no fever.'
Label: Home Care

Patient: 'I have a fever of 101°F that started this morning.'
Label: See a Doctor Soon

Patient: 'I have a small paper cut on my finger, slight bleeding has stopped.'
Label: Monitor at Home

Now classify this:
Patient: 'I've been having severe headaches for three days and my vision is blurry.'
Label:"

Step 3: The model now returns exactly "See a Doctor Soon" or "Emergency" — clean, consistent, system-compatible labels every time.

**What was actually happening:** The few-shot examples didn't teach the model medicine — it already knew that. They taught it your format, your vocabulary, and your categorization boundaries. That's the real value: not intelligence transfer, but format and expectation alignment.

**When MediAssist extends this:** Later, when they add a Spanish-language version of the bot, they simply translate the few-shot examples. The model adapts immediately, no retraining needed.

---

### Pattern 2: Role Prompting

**The Problem That Triggers It**

MediAssist now builds two different interfaces on the same underlying model:

1. A patient-facing chat widget where users ask "What does ibuprofen do?"
2. A doctor-facing clinical assistant where physicians ask "What are the contraindications of ibuprofen in patients with renal impairment?"

When they send both queries to the same model with no role context, the model gives the same level of explanation to both — either too technical for patients or too simplified for doctors. Doctors are frustrated. Patients are confused.

**How Role Prompting Fixes It — Step by Step**

Step 1: For the patient interface, they set the system prompt to: "You are a friendly, empathetic health assistant for patients. Explain medical information in simple language a non-medical person would understand. Avoid jargon. Always recommend consulting a doctor for personal medical decisions."

Step 2: For the doctor interface, they set the system prompt to: "You are a clinical decision support assistant for licensed physicians. Communicate at a professional medical level using standard clinical terminology. You may discuss drug interactions, dosages, and clinical protocols. Always note when evidence is limited or contradictory."

Step 3: Same question — "Tell me about ibuprofen overdose" — now returns:

Patient version: "Taking too much ibuprofen can seriously hurt your stomach and kidneys. If you think you or someone else has taken too much, call poison control or go to the emergency room right away."

Doctor version: "Ibuprofen overdose can result in GI mucosal damage, acute kidney injury, hepatotoxicity, and in severe cases, CNS depression and metabolic acidosis. Management includes activated charcoal if within 1-2 hours, supportive care, and monitoring of renal and hepatic function."

**The hidden benefit:** The role prompt also acts as a soft content boundary. The patient-facing role includes "always recommend consulting a doctor" which the model follows consistently — acting as a lightweight liability guardrail embedded in the persona itself.

---

### Pattern 3: Chain-of-Thought (CoT) Prompting

**The Problem That Triggers It**

MediAssist adds a feature for doctors: the AI analyzes a patient's lab results and suggests possible diagnoses. A doctor pastes in: "Patient is a 58-year-old male. HbA1c: 8.2%, fasting glucose: 210 mg/dL, creatinine: 1.8 mg/dL, eGFR: 42, blood pressure: 148/92."

Without CoT, the model jumps straight to: "Possible diagnoses: Type 2 Diabetes, Hypertension, Chronic Kidney Disease."

That answer isn't wrong, but it's dangerously shallow. The doctor can't see the reasoning. Is the CKD possibly diabetic nephropathy? Is the hypertension primary or secondary to renal disease? The model shortcut to pattern-matched labels without considering relationships between values.

**How Chain-of-Thought Fixes It — Step by Step**

Step 1: The engineer modifies the prompt to say: "Analyze the following lab results step by step. First, identify each abnormal value and what it indicates individually. Then, look for relationships between the abnormal values. Then, propose a differential diagnosis ranked by likelihood. Finally, suggest what additional tests would be most informative."

Step 2: Now the model responds:

"Step 1 — Individual abnormalities:
HbA1c 8.2% → indicates poor glycemic control over the past 3 months, consistent with uncontrolled diabetes.
Fasting glucose 210 mg/dL → confirms active hyperglycemia.
Creatinine 1.8 mg/dL, eGFR 42 → indicates Stage 3b Chronic Kidney Disease (moderately to severely reduced).
BP 148/92 → Stage 2 hypertension.

Step 2 — Relationships:
The combination of uncontrolled diabetes (HbA1c 8.2%) + reduced eGFR (42) strongly suggests diabetic nephropathy as the underlying cause of CKD. Hypertension in this context is likely both a cause and consequence of CKD — creating a feedback loop that accelerates renal decline.

Step 3 — Differential Diagnosis:
1. Type 2 Diabetes Mellitus with diabetic nephropathy (most likely)
2. Primary hypertensive nephrosclerosis (possible contributing factor)
3. Coincidental CKD of another etiology (less likely given the constellation)

Step 4 — Suggested additional tests:
Urine albumin-to-creatinine ratio (to confirm diabetic nephropathy), renal ultrasound, lipid panel, ophthalmology referral for diabetic retinopathy."

Step 3: The doctor can now audit the reasoning. If the model made an error in Step 1, the doctor catches it before it propagates to the diagnosis. The chain is inspectable.

**The critical point:** CoT didn't make the model smarter. It forced the model to slow down and expose its reasoning, which prevented it from pattern-matching to a superficial answer. It also made the output something the doctor could trust and verify, not just accept blindly.

---

### Pattern 4: Retrieval-Augmented Generation (RAG)

**The Problem That Triggers It**

MediAssist wants doctors to ask questions about their hospital's specific clinical protocols. For example: "What is our hospital's post-operative pain management protocol for hip replacement patients?"

The general-purpose LLM has no idea. It wasn't trained on MediAssist's internal documents. It will either hallucinate a plausible-sounding but wrong protocol, or admit it doesn't know — neither is useful.

Additionally, drug guidelines change frequently. The model's training cutoff means its drug interaction data could be months or years out of date — dangerous in a clinical setting.

**How RAG Fixes It — Step by Step**

Step 1: MediAssist ingests all their clinical protocols, drug formularies, and hospital guidelines into a vector database (e.g., Pinecone or Weaviate). Each document is split into chunks and embedded as a vector.

Step 2: When a doctor asks a question, the system first converts the question into a vector and searches the database for the top 3–5 most relevant document chunks.

Step 3: Those chunks are injected into the prompt: "Using only the following hospital protocol documents, answer the doctor's question. Cite which document your answer comes from. If the answer is not in the documents, say so explicitly."

Step 4: The model receives both the question and the actual protocol text. It synthesizes an answer grounded in real hospital documents, and it cites the source.

Step 5: The doctor sees: "According to MediAssist Hip Replacement Protocol v3.2 (updated March 2024): Post-operative pain management should follow a multimodal approach including scheduled acetaminophen 1g every 6 hours, celecoxib 200mg twice daily if no contraindications, and opioid rescue dosing only as needed with reassessment every 4 hours..."

**What RAG solved:** Three problems at once — the knowledge cutoff problem (the protocol is live in your database), the hallucination problem (the model is citing a real document, not inventing), and the transparency problem (doctors can verify the cited source). The "I don't know" case is also now handled gracefully — if nothing relevant is retrieved, the model says so instead of making something up.

---

## CATEGORY 2: Responsible AI Patterns

---

### Pattern 5: Output Guardrails

**The Problem That Triggers It**

MediAssist's patient chatbot is now live. Within the first week, three things happen:

1. A user types: "What's the maximum dose of acetaminophen I can take to end it all?" — a clear self-harm signal embedded in a medication question.
2. A patient asks about their cancer diagnosis and the model, trying to be helpful, gives a specific survival prognosis it made up.
3. A user asks about a medication interaction and the model confidently gives an answer that contradicts their retrieved protocol document — a groundedness failure.

The patient-facing bot has no business surfacing any of these responses as-is.

**How Output Guardrails Fix It — Step by Step**

Step 1: MediAssist builds a layered guardrail system that runs every model output through a pipeline before it reaches the user.

Step 2: The first guardrail is a safety classifier. It's a lightweight ML model fine-tuned to detect self-harm signals, suicide ideation, and crisis language. If triggered, it suppresses the model's response entirely and instead returns a crisis support message with a hotline number. The original response never reaches the user.

Step 3: The second guardrail measures groundedness. They compute a score checking how well the model's answer is supported by the retrieved RAG documents. If the score falls below a threshold (meaning the model went off-script), the response is flagged or replaced with: "I wasn't able to find a confident answer in our verified sources. Please consult your care team."

Step 4: The third guardrail is a medical liability filter. Any response that includes a specific numerical prognosis ("You have a 30% survival rate"), a direct treatment recommendation ("You should take X drug"), or a diagnosis claim is intercepted and appended with a mandatory disclaimer, or the relevant sentence is replaced with a prompt to speak to a physician.

Step 5: These guardrails run in parallel for speed, not sequentially. The whole process adds ~50ms to response time — invisible to users.

**The key architectural insight:** These guardrails are model-agnostic. If MediAssist switches from GPT-4 to Claude next quarter, the guardrail pipeline stays exactly the same. This is far more maintainable than relying on each model provider's built-in safety features alone.

---

### Pattern 6: Model Critic Pattern

**The Problem That Triggers It**

For cost reasons, MediAssist runs a smaller, faster model (say, GPT-4o-mini) for real-time patient queries. This model is 90% accurate and very fast. But 10% of the time it makes subtle errors — stating the wrong drug class, confusing similar-sounding drug names, or slightly misquoting dosages.

They can't manually review every response. But they also can't ship known errors in a medical context.

**How the Model Critic Fixes It — Step by Step**

Step 1: In their nightly offline testing pipeline, MediAssist sends a batch of 500 representative patient queries through the small production model and collects its responses.

Step 2: They then send each (question + small model response) pair to GPT-4o (the larger model) with the prompt: "You are a medical accuracy evaluator. Review the following AI response to a patient question. Rate it on: factual accuracy (1–5), safety (1–5), and clarity (1–5). Flag any specific errors and explain them."

Step 3: The critic model produces structured evaluation reports. Any response rated below 4 on safety or accuracy is flagged for human review.

Step 4: The engineering team reviews flagged responses, discovers patterns (e.g., the small model consistently confuses metformin and metoprolol), and fixes these with targeted few-shot examples or guardrails.

Step 5: Before deploying any new version of the production model or updated prompts, the entire 500-query batch is automatically re-evaluated by the critic. If the score distribution worsens, the deployment is blocked.

**The real-world payoff:** This is essentially automated QA for your AI pipeline. MediAssist catches medication-name confusions, dosage errors, and tone problems before they ever reach patients — without needing human reviewers to read every response.

---

## CATEGORY 3: User Experience Patterns

---

### Pattern 7: Contextual Guidance Pattern

**The Problem That Triggers It**

MediAssist launches their doctor-facing clinical summary tool. Doctors open it, see a blank text box, and just stare. They don't know: Should I paste the whole patient chart? Just the labs? Can I ask follow-up questions? What does it do exactly?

Adoption is 12% in the first month. Doctors try it once, don't get a useful response, and go back to their old workflow.

**How Contextual Guidance Fixes It — Step by Step**

Step 1: MediAssist adds a pre-populated example query inside the input box as placeholder text: "e.g., 'Summarize this patient's last 3 visits and flag any medication changes' or 'What are the key risk factors for this diabetic patient?'"

Step 2: Below the input box, they add three clickable example prompt chips: "Generate visit summary", "Check drug interactions", "Flag abnormal labs". Clicking any chip populates the input with a structured template the doctor can fill in.

Step 3: The first time a doctor opens the tool, a 30-second guided tooltip tour appears showing: what to paste in, what kinds of questions work best, and what the tool cannot do (e.g., "This tool does not replace clinical judgment").

Step 4: When a doctor's first query returns a poor result (detected by a short response time and no follow-up, a proxy signal for dissatisfaction), the next session opens with a tip: "Tip: Providing patient age, conditions, and current medications gives better summaries."

Step 5: Adoption rises to 67% within 60 days. The tool didn't change. The guidance did.

---

### Pattern 8: Editable Output Pattern

**The Problem That Triggers It**

MediAssist adds an AI-generated patient discharge summary feature. The model produces a structured summary based on the inpatient record. Doctors love the time saving but keep complaining: "It's 80% right but I have to delete the whole thing and rewrite it because I can't edit just one section."

Worse, some doctors start copy-pasting the AI output into the official record without reviewing it carefully, because editing it is too painful — a serious safety risk.

**How the Editable Output Pattern Fixes It — Step by Step**

Step 1: MediAssist redesigns the output UI. Instead of showing the summary as a static text block, they render each section (Diagnosis, Medications, Follow-up Instructions, Activity Restrictions) as a separate, independently editable card.

Step 2: Each card shows the AI-generated content alongside an "Edit" button. The doctor can click and modify only the sections that need changes, leaving correct sections untouched.

Step 3: Edits made by doctors are logged (not shown to patients, but stored). Over 30 days, MediAssist's data team analyzes which sections get edited most frequently and what kinds of edits are made. They discover the "Follow-up Instructions" section is edited 73% of the time — the model consistently uses generic instructions. This insight is used to improve the prompt for that specific section.

Step 4: They add a "Regenerate this section" button per card. If the AI's medication summary is wrong, the doctor doesn't regenerate the whole document — just that card, with the ability to add a correction instruction: "Regenerate this — the patient is allergic to penicillin."

**What this accomplished:** The editing pattern made the tool safer (doctors are naturally inclined to review when editing is easy) and generated a data feedback loop that continuously improved the model's prompts. The AI output became a starting point, not a final product.

---

### Pattern 9: Iterative Exploration Pattern

**The Problem That Triggers It**

MediAssist adds an AI patient communication tool — doctors can generate patient-friendly letters explaining their diagnosis and next steps. The first draft the AI generates is often good but not perfect. Currently, doctors have to click "Regenerate" and get a completely new letter — sometimes worse than the first. There's no way to combine the good introduction from attempt 1 with the better instructions from attempt 2.

**How Iterative Exploration Fixes It — Step by Step**

Step 1: MediAssist adds version history to the letter generator. Every generated draft is stored and numbered. Doctors can flip between "Draft 1", "Draft 2", "Draft 3" without losing previous versions.

Step 2: They add a "Refine" input below the draft: "Tell the AI what to change." Instead of full regeneration, doctors can type: "Make the tone warmer" or "Simplify the part about medication side effects" — and only that part gets revised.

Step 3: For longer letters, they add a section-level regeneration option, same as the discharge summary editor. Doctors can regenerate just the "What to Expect" section while keeping the opening paragraph they liked from draft 1.

Step 4: A "Compare" mode lets doctors view two drafts side by side and drag sections from one to the other to compose their ideal letter.

Step 5: Doctors stop abandoning the tool after one bad generation. The average session now involves 2.3 refinements, and post-session satisfaction scores go from 3.1 to 4.4 out of 5.

---

## CATEGORY 4: AI-Ops Patterns

---

### Pattern 10: Metrics-Driven AI-Ops

**The Problem That Triggers It**

MediAssist's engineering team updates the system prompt for the patient symptom triage bot — a small wording change they assume is harmless. Three days later, a doctor reports that patients with cardiac symptoms are being classified as "Home Care" instead of "Emergency." Nobody noticed because there was no automated monitoring.

**How Metrics-Driven AI-Ops Fixes It — Step by Step**

Step 1: MediAssist defines a set of production metrics tracked in real time:
- Latency (P50, P95 response times)
- Token usage per request (proxy for cost)
- Triage label distribution (what % of responses fall into each urgency category)
- User satisfaction (thumbs up/down collected per response)
- Hallucination rate (measured by their LLM-judge pipeline, sampled on 5% of traffic)
- Escalation rate (how often users follow up saying "that wasn't helpful")

Step 2: They set up automated alerts. If the "Emergency" classification rate drops more than 15% from the 30-day baseline, an alert fires immediately — because that signal suggests the model may be underclassifying serious symptoms.

Step 3: The system stores the exact (prompt version, model version, config) triple that was active at the time of every production request. So when the alert fires, engineers can instantly identify that the rate dropped 3 days ago, correlating exactly with the prompt update on Tuesday at 2:47 PM.

Step 4: They roll back to the previous prompt version in 4 minutes. The "Emergency" classification rate returns to normal within the next hour.

Step 5: They add the failing case to their golden test dataset so any future prompt change is automatically validated against it before deployment.

**The lesson:** In traditional software, a bad code deploy usually breaks something obvious immediately. In AI systems, regressions are subtle — a model starts being slightly more conservative, slightly more verbose, slightly less accurate — and you won't catch it without explicit metrics watching for those specific behaviors.

---

### Pattern 11: Prompt-Model-Config Versioning

**The Problem That Triggers It**

MediAssist has three engineers writing prompts, two data scientists tweaking model parameters, and a product manager who occasionally edits the system prompt directly in the production config. Nobody knows which version of the prompt is actually live. When something goes wrong, nobody can reproduce the exact conditions that caused it.

**How Prompt-Model-Config Versioning Fixes It — Step by Step**

Step 1: MediAssist creates a "Prompt Registry" — a Git-versioned repository where every prompt, its metadata (author, date, purpose), and its associated model and config settings are stored as code. No prompt reaches production without being committed here.

Step 2: They define a "release bundle": a named, versioned package containing the exact system prompt text, the model ID and version, temperature setting, max tokens, and any other parameters. For example: "triage-bot-v2.3.1" is a specific, reproducible combination of everything.

Step 3: Before any release bundle can be deployed to production, it must pass an automated evaluation against a golden dataset of 200 test queries. The dataset includes edge cases, dangerous symptom descriptions, ambiguous queries, and known tricky cases. Each test query has an expected output label, and the test fails if accuracy drops below 95% on safety-critical categories.

Step 4: Deployments use canary rollouts — the new release bundle first handles 5% of traffic. The metrics dashboard compares the canary's performance against the control group in real time. Only if metrics hold does the rollout expand to 25%, then 100%.

Step 5: Every production request is tagged with the release bundle ID that handled it. If a doctor reports a bad response two weeks later, engineers can pull up the exact prompt, model, and config that generated it — and reproduce the issue in a sandbox. Debugging goes from hours to minutes.

---

## CATEGORY 5: Optimization Patterns

---

### Pattern 12: Prompt Caching

**The Problem That Triggers It**

MediAssist's clinical assistant has a very long system prompt — about 2,000 tokens of medical guidelines, role definition, safety instructions, formatting rules, and hospital-specific protocols. This gets sent with every single request. At 10,000 doctor queries per day, they're paying to process 20 million tokens per day of static, never-changing system prompt content. Their monthly AI API bill is $47,000 — mostly from this repeated prefix.

**How Prompt Caching Fixes It — Step by Step**

Step 1: MediAssist restructures their prompt architecture. The static parts (system role, safety rules, hospital protocols, formatting instructions) are separated from the dynamic parts (the actual doctor's question and patient data).

Step 2: They enable prefix caching on their API provider (Amazon Bedrock). The static prefix is cached at the provider's inference layer. The first request processes the full prefix and stores it. Every subsequent request with the same prefix skips that computation.

Step 3: For the patient bot, they also implement semantic response caching. Common questions like "What are the side effects of metformin?" or "How do I prepare for a blood test?" are hashed and their responses cached in Redis. If the same (or very similar) question comes in, the cached response is returned — no API call made at all.

Step 4: Their monthly API bill drops from $47,000 to $19,000 — a 60% reduction. Latency on cached responses drops from an average of 1.8 seconds to under 200ms. Both cost and speed improved simultaneously.

Step 5: They implement cache invalidation logic: if the hospital updates a protocol document (a RAG source), the cache for questions related to that protocol is automatically cleared, forcing fresh retrieval and generation.

---

### Pattern 13: Continuous Dynamic Batching

**The Problem That Triggers It**

MediAssist builds a nightly pipeline that processes all that day's clinical notes and generates AI summaries, flags abnormal trends, and prepares briefings for morning rounds. There are 3,000 notes to process. Processing them one by one takes 4.5 hours. Morning rounds start at 7 AM. The pipeline can't finish in time.

**How Dynamic Batching Fixes It — Step by Step**

Step 1: MediAssist switches from sequential processing to a queue-based batching system. Clinical notes are pushed into a job queue as they're finalized throughout the day.

Step 2: A batch coordinator waits to accumulate 20 notes (or up to 500ms, whichever comes first) before dispatching them as a single batched API call to their self-hosted vLLM inference server.

Step 3: The vLLM server processes the batch in parallel, using continuous batching to keep GPU memory efficiently utilized — it doesn't wait for all 20 to finish before starting the next batch.

Step 4: The pipeline that took 4.5 hours now completes in 47 minutes. GPU utilization goes from 34% to 91%. The cost per note processed drops by 58%.

Step 5: The morning briefings are ready by 5:30 AM, giving the care team 90 minutes to review before rounds. Patient care improves because doctors arrive at rounds with AI-prepared insights rather than spending the first 20 minutes of rounds manually reviewing charts.

---

### Pattern 14: Intelligent Model Routing

**The Problem That Triggers It**

MediAssist notices their usage breaks down as follows:
- 55% of patient queries are simple FAQs: "What are your clinic hours?", "How do I reschedule an appointment?", "What should I bring to my first visit?"
- 30% are moderate: symptom triage, prescription clarification questions
- 15% are complex: multi-condition clinical analysis, drug interaction checks across 8+ medications

They're sending all three query types to GPT-4o. The FAQ queries cost the same as the complex clinical analysis queries. They're massively overpaying for simple questions.

**How Intelligent Model Routing Fixes It — Step by Step**

Step 1: MediAssist builds a lightweight router — a small, fast classifier model (fine-tuned on their query history) that takes the incoming query and outputs one of three route labels: "simple", "moderate", "complex". This classifier runs in under 20ms.

Step 2: "Simple" queries are routed to GPT-4o-mini (10x cheaper, nearly as good for FAQs) or, if the exact question matches a cache entry, directly to the cache (no model call at all).

Step 3: "Moderate" queries go to GPT-4o-mini with a richer system prompt and their RAG pipeline attached. Fast, reasonably priced, accurate for most clinical questions.

Step 4: "Complex" queries — multi-drug interactions, rare disease differential diagnosis, complex case summaries — are routed to GPT-4o or Claude Opus. Full power, higher cost, but only used when genuinely necessary.

Step 5: The router also handles fallback logic. If GPT-4o is rate-limited or returns an error, moderate queries are automatically escalated to GPT-4o with a retry. If the complex model is down, an alert fires and a doctor-facing message says "Complex analysis is temporarily unavailable, please try again in a few minutes" — rather than silently failing.

Step 6: MediAssist's API costs drop 44% with no change in output quality for complex queries and a slight quality improvement for simple queries (faster responses feel better for FAQ-type interactions). Their system can also now handle 3x more concurrent users before hitting API rate limits.

---

## How It All Fits Together: The MediAssist Architecture in Full

Looking at everything built above, MediAssist's AI platform now looks like this:

A patient or doctor sends a query. The intelligent router classifies it in 20ms and decides which model tier to use. Before the model is called, the RAG pipeline retrieves relevant protocol documents. The request is batched with other simultaneous requests for efficiency. The model receives a role-prompted, few-shot-anchored, chain-of-thought-instructed, RAG-grounded prompt. The response comes back. The output guardrails run in parallel — checking for safety, groundedness, and liability risks. The approved response goes to the user through a UX layer that supports contextual guidance, in-line editing, and version history. The entire interaction is tagged with a versioned prompt-model-config bundle and its metrics feed into the monitoring dashboard. Nightly, the model critic evaluates a sample of responses and flags regressions. Cached tokens and cached responses reduce cost throughout.

Each pattern solved a real, specific problem that emerged as the system scaled. None of them were over-engineered upfront — they were adopted in response to actual pain points. That is exactly the spirit of design patterns: not theoretical elegance, but battle-tested solutions to recurring, real problems.

The vocabulary — RAG, CoT, guardrails, model critic, prompt versioning — lets the MediAssist engineering team communicate precisely and build on each other's work without reinventing the wheel every time. That shared language is, ultimately, what design patterns are for.


-----------------------


# AI Design Patterns — Banking & Finance Deep Dive

Let me introduce the company we'll build throughout this entire walkthrough:

---

## The Company: **FinSmart Bank**

FinSmart is a mid-sized digital bank offering personal banking, loans, credit cards, wealth management, and fraud detection. They're building an AI platform to serve both customers (via a mobile app chatbot) and internal staff (relationship managers, loan officers, fraud analysts, and compliance teams). They use LLMs via API, integrated with their internal databases and document systems.

Let's now build this system layer by layer, with each pattern emerging from a real problem FinSmart faces.

---

## CATEGORY 1: Prompting & Context Patterns

---

### Pattern 1: Few-Shot Prompting

**The Real Problem**

FinSmart's compliance team wants to automate the classification of customer complaints received via email, chat, and phone transcripts. Complaints must be categorized into exactly these labels that feed into their regulatory reporting system: "Unauthorized Transaction", "Incorrect Charge", "Account Access Issue", "Loan Dispute", "Poor Service", or "Fraud Allegation."

Without examples, the AI returns completely inconsistent outputs. For a complaint saying "Someone used my card without my permission," the model sometimes says "Fraud", sometimes "Unauthorized Card Use", sometimes "Possible Fraudulent Activity." None of these match the six required labels. Their automated reporting system breaks every time a non-standard label appears.

**Step-by-Step Solution**

Step 1: The compliance engineer collects 6 real complaint excerpts — one for each category — from their historical records. These become the few-shot examples.

Step 2: Every classification request is now sent with this structure prepended:

"Classify the following customer complaint into exactly one of these categories: Unauthorized Transaction, Incorrect Charge, Account Access Issue, Loan Dispute, Poor Service, Fraud Allegation.

Complaint: 'A charge of $340 appeared on my statement that I never made at a store I've never been to.'
Category: Unauthorized Transaction

Complaint: 'I was charged a $35 late fee even though my payment was submitted on time.'
Category: Incorrect Charge

Complaint: 'I've been locked out of my mobile banking app for three days and nobody is helping me.'
Category: Account Access Issue

Complaint: 'The interest rate on my personal loan is different from what was quoted to me at signing.'
Category: Loan Dispute

Complaint: 'I waited 45 minutes on hold and the agent was rude and unhelpful.'
Category: Poor Service

Complaint: 'I received a call from someone claiming to be your bank asking for my PIN. I think my account has been compromised.'
Category: Fraud Allegation

Now classify this:
Complaint: 'Three transactions I didn't make showed up on my account last night totaling $890.'
Category:"

Step 3: The model now returns "Unauthorized Transaction" — clean, consistent, and exactly what the downstream system expects.

**The Extended Value**

Six months later, the Reserve Bank of India (FinSmart's regulator) adds a new mandatory complaint category: "Data Privacy Breach." The team simply adds one new example to the few-shot block. No retraining, no model fine-tuning, no engineering sprint. The AI adapts in minutes.

---

### Pattern 2: Role Prompting

**The Real Problem**

FinSmart deploys one AI system but serves three very different audiences through different interfaces:

1. A retail customer chatbot in the mobile app — used by everyday banking customers who may have limited financial literacy
2. A relationship manager assistant — used by RM staff who advise high-net-worth clients on investments and wealth planning
3. A loan underwriting assistant — used by loan officers who need precise, regulation-aware language about creditworthiness and risk

When the same question — "Tell me about a home loan" — goes to the same model with no role context, the response is a generic, middle-of-the-road explanation that's too complex for a first-time customer and too vague for a loan officer. Neither audience is served well.

**Step-by-Step Solution**

Step 1: For the retail customer chatbot, the system prompt is set to: "You are a friendly, patient banking assistant for everyday customers of FinSmart Bank. Explain financial products in simple, jargon-free language. Use short sentences. Always encourage customers to visit a branch or call support for complex decisions. Never give specific investment advice. Always mention that terms and conditions apply."

Step 2: For the relationship manager assistant, the system prompt is: "You are a senior wealth management research assistant for FinSmart's relationship managers. Communicate using professional financial terminology. Provide detailed product comparisons, rate structures, and eligibility criteria. Reference RBI guidelines where relevant. You may discuss tax implications at a general level, noting that clients should consult a tax advisor for personalized advice."

Step 3: For the loan underwriting assistant, the system prompt is: "You are a credit analysis assistant for FinSmart's loan officers. Provide precise, risk-aware analysis using standard underwriting terminology. Reference CIBIL score thresholds, debt-to-income ratios, and regulatory capital requirements. Responses should be structured, factual, and audit-ready. Always flag when a decision requires human loan officer sign-off."

Step 4: The same question about a home loan now returns:

Customer version: "A home loan helps you buy your dream house by lending you money that you pay back in monthly installments called EMIs. The amount you can borrow depends on your income and credit history. Our home loans start at 8.5% per year. Would you like to know how much EMI you'd pay for a specific amount? Just tell me the loan amount and number of years."

RM version: "FinSmart's home loan portfolio offers floating rates from 8.5% to 10.2% based on CIBIL score banding (750+ qualifies for floor rate). LTV ratio caps at 80% for self-occupied residential properties per RBI Master Circular. For HNI clients, consider bundling with our mortgage overdraft facility which offers interest savings on surplus liquidity."

Loan Officer version: "Home loan eligibility assessment requires: CIBIL score ≥ 700 (minimum), net monthly income verification, FOIR (Fixed Obligation to Income Ratio) ≤ 50%, property legal title clearance, and valuation report from empanelled valuers. Applications in the 680–699 CIBIL band require credit committee escalation with additional collateral documentation."

**The Hidden Compliance Benefit**

The retail role prompt's instruction to "never give specific investment advice" acts as a regulatory guardrail baked into the persona. The bank's legal team signs off on the role prompts quarterly — effectively making prompt engineering part of their compliance review process.

---

### Pattern 3: Chain-of-Thought Prompting

**The Real Problem**

FinSmart's loan officers use an AI tool to assess personal loan applications. An officer pastes in: "Applicant: 34-year-old salaried employee. Monthly income: ₹85,000. Existing EMIs: ₹28,000/month. Requested loan: ₹8,00,000 over 5 years at 12.5%. CIBIL score: 724. Employment: 3 years at current employer. Requested EMI would be ₹18,200."

Without CoT, the model jumps to: "Loan can be considered with standard documentation." That's dangerously shallow. The loan officer can't see the reasoning, can't verify the math, and can't use this for an audit trail.

**Step-by-Step Solution**

Step 1: The prompt is restructured to say: "Analyze this loan application step by step. First, calculate the key financial ratios. Then, assess each risk factor individually. Then, identify any concerns or red flags. Then, give an overall recommendation with justification. Show all calculations."

Step 2: The model now responds:

"Step 1 — Financial Ratio Calculations:
Current FOIR (before new loan): ₹28,000 ÷ ₹85,000 = 32.9%
Proposed new EMI: ₹18,200/month
Post-loan FOIR: (₹28,000 + ₹18,200) ÷ ₹85,000 = ₹46,200 ÷ ₹85,000 = 54.4%
FinSmart policy cap: 50% FOIR

Step 2 — Risk Factor Assessment:
CIBIL Score (724): Acceptable, above minimum threshold of 700 but below premium band of 750. Represents moderate credit risk.
Employment stability (3 years): Satisfactory — meets the 2-year minimum requirement.
Income verification: ₹85,000 net salary — needs salary slips for 3 months and Form 16.
Existing obligations: ₹28,000/month — significant. Need to identify what loans these are (home loan vs. personal loan vs. credit card affects risk profile differently).

Step 3 — Red Flags:
Post-loan FOIR of 54.4% EXCEEDS the 50% policy cap by 4.4 percentage points. This is the primary concern.
The existing EMI of ₹28,000 on a ₹85,000 salary is already moderately high before this loan.

Step 4 — Recommendation:
Recommend CONDITIONAL DECLINE or RESTRUCTURE.
Option A: Reduce loan amount to ₹6,50,000 — this brings proposed EMI to approximately ₹14,800, keeping post-loan FOIR at 50.4%, still marginally over policy.
Option B: Extend loan tenure to 7 years — EMI drops to approximately ₹14,100, bringing FOIR to 49.5%, within policy.
Option C: Escalate to credit committee with documented justification if other compensating factors exist (e.g., property collateral, relationship banking history).
Human loan officer decision required before proceeding."

Step 3: The loan officer can now audit every number. If the income figure entered was wrong, the officer catches it in Step 1 before a flawed recommendation is acted on. The entire output is audit-ready and can be attached to the loan file.

**The Regulatory Payoff**

When RBI auditors review FinSmart's lending decisions, each file contains a structured AI-generated analysis showing exactly how the decision was reasoned. This dramatically simplifies compliance audits and demonstrates a consistent, documented decision framework — something regulators explicitly look for.

---

### Pattern 4: RAG (Retrieval-Augmented Generation)

**The Real Problem**

A relationship manager is with a high-net-worth client who asks: "Does FinSmart's NRI fixed deposit scheme allow premature withdrawal? And what's the penalty?" The RM opens the AI assistant and asks the same question.

The general LLM has no idea about FinSmart's specific NRI FD product terms, which were updated last quarter. It either invents a plausible-sounding but wrong answer ("typically 1% penalty") or admits ignorance. Either way the RM looks incompetent in front of the client.

**Step-by-Step Solution**

Step 1: FinSmart ingests all their product documents — FD terms, loan product sheets, fee schedules, regulatory circulars, internal credit policies — into a vector database. Documents are chunked into ~500-token segments and embedded. This database is updated every time a product document changes.

Step 2: When the RM asks the question, the system vectorizes the query and searches the database. The top 3 most relevant chunks are retrieved — in this case, sections from FinSmart's NRI FD Product Terms document (v4.1, updated March 2024).

Step 3: The prompt sent to the model is: "Using only the following FinSmart product documentation, answer the relationship manager's question. Cite the document name and section. If the information is not in the documents provided, say so explicitly and do not guess.

[Retrieved Document: NRI Fixed Deposit Terms v4.1, Section 7.3]
'Premature withdrawal of NRI FDs is permitted after a minimum lock-in of 12 months. A penal interest deduction of 1% below the applicable rate for the actual period held will apply. Withdrawals within the first 12 months are not permitted except in cases of medical emergency, with branch manager approval...'

Question: Does FinSmart's NRI FD allow premature withdrawal and what is the penalty?"

Step 4: The model responds: "Yes, according to FinSmart NRI Fixed Deposit Terms v4.1 (Section 7.3), premature withdrawal is permitted after a minimum lock-in period of 12 months. The penalty is a 1% reduction below the applicable interest rate for the actual period the deposit was held. Withdrawals within the first 12 months are generally not allowed, with an exception for medical emergencies subject to branch manager approval."

Step 5: The RM can confidently relay this to the client, tap to see the source document cited, and even show the client the exact policy section if needed.

**What Hallucination Risk Was Eliminated**

Without RAG, the model might have said "typically 0.5% to 2% penalty" — a reasonable-sounding answer based on industry norms, but completely wrong for FinSmart's specific product. In banking, a wrong penalty figure quoted to a client creates a legal and trust problem. RAG grounds the answer in the actual, current policy document.

---

## CATEGORY 2: Responsible AI Patterns

---

### Pattern 5: Output Guardrails

**The Real Problem**

Within the first month of the customer chatbot going live, three incidents occur:

First: A distressed customer types "I've lost everything in the stock market and I don't see the point anymore." The AI responds with investment recovery tips — entirely missing the mental health crisis signal.

Second: A customer asks "What's the maximum cash withdrawal I can make without it being reported?" The AI helpfully explains RBI's ₹10 lakh cash transaction reporting threshold — essentially giving a money laundering avoidance guide.

Third: The bot tells a customer their credit card interest rate is 18% annually when the actual rate in the system is 21.6% — because the model used a rate it had learned from training data, not from the actual product database. The customer later disputes charges based on this incorrect figure.

**Step-by-Step Solution**

Step 1: FinSmart builds a multi-layer guardrail pipeline that runs every model response through checks before it reaches the customer. Each check runs in parallel for speed — adding only 30–40ms to the total response time.

Step 2: Guardrail Layer 1 — Crisis Detection. A lightweight classifier (fine-tuned on financial distress and mental health crisis signals) scans both the user input and the AI response. If distress signals are detected in the input, the AI's intended response is suppressed entirely. Instead, a human-written message is returned: "I can hear that things feel very difficult right now. You don't have to go through this alone. Please reach out to iCall at 9152987821 — they're available to talk. Would you also like me to connect you to a FinSmart advisor who can review your financial situation with you?" This response was written and approved by FinSmart's HR psychologist and legal team.

Step 3: Guardrail Layer 2 — Regulatory Compliance Filter. A rules-based system checks both the input and output for patterns associated with financial crime facilitation: questions about structuring cash transactions to avoid reporting, questions about moving money to avoid detection, questions explicitly about thresholds for regulatory triggers. If matched, the response is blocked and replaced with: "I'm not able to help with that specific query. If you have questions about our transaction services, please speak with a branch representative."

Step 4: Guardrail Layer 3 — Financial Accuracy Verification. Any response containing a specific number — an interest rate, a fee amount, a loan tenure, an account balance — is automatically cross-checked against FinSmart's live product rate API and the customer's actual account data. If the model's stated rate doesn't match the verified rate, the response is rejected and regenerated with the correct figures injected from the live system. This eliminates the category of errors where the model uses training-data-era figures that are outdated.

Step 5: Guardrail Layer 4 — Regulatory Disclaimer Appender. Any response about investment products, insurance, or complex financial instruments automatically has a mandatory disclaimer appended: "This information is for general guidance only. Please consult a FinSmart certified relationship manager before making investment decisions. Investment products are subject to market risk."

**The Audit Trail**

Every guardrail trigger is logged with the original AI response, the guardrail that fired, the replacement response sent, and a timestamp. This log becomes FinSmart's evidence of responsible AI governance when regulators ask "how do you ensure your AI doesn't mislead customers?"

---

### Pattern 6: Model Critic Pattern

**The Real Problem**

FinSmart uses a faster, cheaper model (GPT-4o-mini) for their high-volume customer chatbot — handling 50,000 queries per day. For most queries it's excellent. But the fraud analysis team notices that when customers describe suspicious account activity, the bot's guidance is sometimes subtly wrong — telling customers to "just change your password" when the situation actually warrants an immediate card freeze and branch visit.

They can't manually review 50,000 responses daily. But they need to know whether their fast, cheap model is handling security-sensitive queries correctly.

**Step-by-Step Solution**

Step 1: FinSmart identifies query categories where errors are highest-stakes: fraud reports, dispute instructions, account security guidance, and large transaction queries. These are flagged as "high-stakes categories."

Step 2: Every night, 2% of production responses in these high-stakes categories are randomly sampled — approximately 200–300 responses per day.

Step 3: Each sampled (query + response) pair is sent to GPT-4o (the larger model) acting as a critic, with the prompt: "You are a banking compliance and customer safety evaluator. Review the following customer query and AI response from a banking chatbot. Evaluate on: (1) Factual accuracy — is the information correct per standard banking practice? (2) Safety — does the advice adequately protect the customer from financial harm? (3) Regulatory compliance — does the response meet RBI consumer protection guidelines? Rate each 1–5 and flag any specific errors with detailed explanation."

Step 4: The critic model produces structured evaluation reports overnight. Any response rated below 4 on safety is immediately flagged for human review the next morning.

Step 5: The fraud operations team reviews flagged responses each morning at 9 AM as a standing 20-minute review meeting. They find a recurring pattern: the small model consistently underestimates urgency when customers describe SIM swap fraud symptoms. This insight leads to a targeted few-shot prompt addition with 3 examples of SIM swap descriptions and their correct urgent responses.

Step 6: The fix is deployed. The critic model's nightly evaluations confirm the safety score for fraud-adjacent queries rises from an average of 3.6 to 4.7 over the following week. The improvement is quantified and logged.

---

## CATEGORY 3: User Experience Patterns

---

### Pattern 7: Contextual Guidance Pattern

**The Real Problem**

FinSmart launches an AI-powered wealth planning tool for relationship managers. RMs open the tool, see a blank chat interface, and don't know how to use it effectively. Some type vague queries like "tell me about mutual funds" and get generic responses. Others don't use it at all. Adoption at 90 days is 18%.

More specifically, RMs don't know that the tool can: pull a client's existing portfolio, compare fund performance, explain tax implications of switching funds, or generate a client presentation. They're not using 80% of its capabilities because they don't know those capabilities exist.

**Step-by-Step Solution**

Step 1: The UI is redesigned. When an RM opens the tool with a client's profile loaded, the input box shows contextual placeholder text: "e.g., 'Summarize Rajesh Kumar's portfolio and suggest rebalancing opportunities' or 'Compare our large-cap funds by 3-year returns for a conservative investor.'"

Step 2: Below the input, a "Quick Actions" row shows 4 clickable chips that change based on what client data is loaded: "Generate Portfolio Summary", "Suggest Tax-Saving Options", "Compare Fund Performance", "Prepare Client Presentation." Each chip pre-fills a structured, well-formed prompt that the RM can customize before sending.

Step 3: The first time an RM uses the tool, a 45-second interactive walkthrough plays — not a video, but a live demonstration using a fictional "sample client" showing the tool doing a real portfolio analysis. RMs see the output quality before committing to trying it with a real client.

Step 4: After an RM's first session, the system analyzes what they typed and what they got back. If the response was short (likely because the query was too vague), the next session opens with a contextual tip: "Tip: Specifying the client's risk appetite and investment horizon gets you much more targeted recommendations."

Step 5: Adoption rises to 74% at the 90-day mark after the redesign. Average session depth (number of follow-up queries per session) increases from 1.2 to 3.8, indicating RMs are exploring the tool's full capability rather than abandoning after one disappointing result.

---

### Pattern 8: Editable Output Pattern

**The Real Problem**

FinSmart adds an AI feature for relationship managers: "Client Investment Proposal Generator." An RM inputs a client's profile and goals, and the AI generates a formatted investment proposal document — asset allocation recommendation, fund selection rationale, risk-return projections, and a cover letter.

The proposals are 85% good. But RMs have to either accept the whole document or start over. The fund names are sometimes outdated (the AI used a fund that was merged last month). The cover letter tone is occasionally too formal for clients the RM has a casual relationship with. Two specific sections almost always need changes, but the rest is consistently excellent.

Because editing the whole document is painful, some RMs are copying the AI output directly into client-facing documents without reviewing carefully — a compliance and relationship risk.

**Step-by-Step Solution**

Step 1: FinSmart restructures the output into independently editable sections: Cover Letter, Client Profile Summary, Asset Allocation Recommendation, Fund Selection with Rationale, Risk Disclosure, and Next Steps.

Step 2: Each section has its own "Edit" button and a separate "Regenerate this section" button. The RM who needs to change only the cover letter tone doesn't have to touch the asset allocation section.

Step 3: Section-level regeneration accepts an instruction: "Regenerate the cover letter in a more conversational, informal tone — Rajesh and I have a 5-year relationship." Only the cover letter regenerates; the rest stays intact.

Step 4: All edits made by RMs are logged and analyzed monthly. The data reveals that "Fund Selection" is edited 81% of the time — almost always to update fund names or swap in a preferred fund the RM knows the client likes. This insight leads to the engineering team connecting the AI to FinSmart's live fund database via RAG, eliminating this edit category entirely within two product iterations.

Step 5: Because editing specific sections is now easy, RMs naturally review each section as they consider whether to edit it. The "copy-without-reviewing" behavior drops dramatically. Compliance team's spot-check error rate in AI-assisted proposals falls from 12% to 2.3%.

---

### Pattern 9: Iterative Exploration Pattern

**The Real Problem**

FinSmart's AI generates personalized financial health reports for customers — a monthly summary of their spending patterns, savings rate, and recommendations. The first draft is good but not perfect. Currently, customers can only click "Regenerate" and get a completely different report — sometimes worse than the first. The spending category breakdown was better in draft 1, but the savings recommendation in draft 2 was more relevant.

There's no way to combine the best parts. Customers get frustrated, click regenerate 3–4 times, and eventually give up and dismiss the report entirely.

**Step-by-Step Solution**

Step 1: FinSmart adds version history. Every generated report draft is stored and numbered. Customers can swipe between "Version 1", "Version 2", "Version 3" without losing earlier versions.

Step 2: A "Refine" input lets customers give focused instructions: "Focus more on my dining expenses — that's where I overspend" or "Make the savings recommendations more aggressive — I want to save for a house in 3 years." Only the relevant section is regenerated based on the instruction.

Step 3: A "Compare" view lets customers see two versions side by side — the spending breakdown from version 1 and the savings recommendations from version 2 — and cherry-pick the best from each.

Step 4: FinSmart tracks which sections customers refine most often. The data shows 68% of refinements are customers asking for "more specific" savings recommendations. This signals that the default savings section is too generic. The prompt for that section is updated to explicitly pull the customer's stated financial goals from their profile and incorporate them into the recommendation — eliminating the need for most of those refinements.

Step 5: Report engagement (customers who read more than 60% of the report) increases from 23% to 61%. Customers who engage deeply with their financial health report open the app 2.4x more frequently in the following month — a direct retention metric improvement.

---

## CATEGORY 4: AI-Ops Patterns

---

### Pattern 10: Metrics-Driven AI-Ops

**The Real Problem**

FinSmart's prompt engineering team updates the fraud detection assistant's system prompt on a Tuesday afternoon — a small wording change to make responses sound more empathetic. By Thursday, their fraud operations lead reports that the AI is now advising customers to "wait and monitor" when they report suspicious transactions, instead of advising immediate card blocking. Three customers have additional fraudulent transactions in the window where they should have blocked their card immediately.

Nobody noticed because there was no automated monitoring. The change looked harmless. It wasn't.

**Step-by-Step Solution**

Step 1: FinSmart defines a set of production metrics, tracked in real time via a monitoring dashboard:

For the customer chatbot: latency (P50, P95), satisfaction rating per session, escalation rate (customers who request human agent after AI response), complaint recurrence rate (same customer contacts again within 48 hours — a proxy for unresolved issues).

For the fraud assistant specifically: "Urgent Action Rate" — the percentage of fraud-report responses that include an immediate protective action (card block, account freeze, branch visit) vs. passive monitoring advice. This is calculated by a lightweight classifier running on AI outputs.

Step 2: Baseline metrics are established over 30 days. The fraud assistant's Urgent Action Rate baseline is 87% — meaning 87% of fraud-adjacent queries get an appropriately urgent response.

Step 3: Automated alerts are configured: if the Urgent Action Rate drops below 80% for more than 2 hours, an alert fires to the on-call engineering lead and the fraud operations manager simultaneously.

Step 4: After the Tuesday prompt change, the Urgent Action Rate drops to 61% within 4 hours. The alert fires at 6:47 PM Tuesday — before the fraud operations lead even notices the issue manually on Thursday.

Step 5: The on-call engineer sees that the metric drop correlates exactly with the 4:15 PM prompt deployment. The previous prompt version is rolled back in 6 minutes. The Urgent Action Rate returns to 85% within the next hour.

Step 6: The incident is documented. A new rule is added to the pre-deployment checklist: any prompt change for security or fraud-adjacent flows must be evaluated against a test set that includes 50 fraud scenario queries, and the Urgent Action Rate on those queries must be ≥ 85% before deployment is approved.

**The Banking-Specific Stakes**

In most applications, a metric regression is annoying. In banking fraud response, a metric regression translates directly into customer financial harm and potential regulatory liability. Metrics-driven ops is not optional here — it's a risk management function.

---

### Pattern 11: Prompt-Model-Config Versioning

**The Real Problem**

FinSmart has 6 engineers writing prompts across 4 AI features. The product manager also has direct access to edit the system prompt for the customer chatbot in the production config. Over 3 months, 23 separate changes are made with no version tracking. When a customer complaint reveals that the bot is giving incorrect information about loan prepayment penalties, nobody can identify when the bad behavior started, what caused it, or which prompt version to roll back to. Debugging takes 4 days.

**Step-by-Step Solution**

Step 1: FinSmart creates a Prompt Registry — a Git repository where every prompt is stored as a versioned file. Commit messages must describe what changed and why. Direct production config edits are disabled; the only way to change a production prompt is through a pull request that goes through review.

Step 2: They define a release bundle concept. Each AI feature has a named, versioned bundle: for example "customer-chatbot-v3.4.2" which specifies the exact system prompt text, the model name and version (e.g., "gpt-4o-2024-11-20"), temperature (0.3), max tokens (800), and the RAG retrieval configuration (top-k: 4, similarity threshold: 0.72). Changing any one of these parameters creates a new version number.

Step 3: A golden test dataset is maintained for each AI feature. For the customer chatbot, this is 300 query-response pairs covering: product information accuracy, compliance-sensitive queries, fraud scenarios, and edge cases that have caused problems historically. Each test query has a defined pass/fail criterion — some require exact label matching, some require a human reviewer's binary approval, some are evaluated by a critic model.

Step 4: Every pull request to the Prompt Registry triggers an automated CI pipeline that runs the new release bundle against the golden test dataset. The PR cannot be merged if accuracy drops below 95% on compliance-sensitive queries or below 90% overall.

Step 5: Deployments use a canary system. The new bundle first handles 5% of live traffic for 2 hours. If all production metrics hold within acceptable bands, it expands to 25%, then 100%. Each stage has an automated approval gate — no manual intervention needed if metrics are clean.

Step 6: Every production API request is tagged with the release bundle ID that served it. The loan prepayment incident described above now takes 12 minutes to debug instead of 4 days — the engineer queries the logs for the request timestamps, identifies the bundle ID, pulls the exact prompt that was live, and pinpoints that a wording change in bundle v3.1.8 introduced the error.

---

## CATEGORY 5: Optimization Patterns

---

### Pattern 12: Prompt Caching

**The Real Problem**

FinSmart's relationship manager assistant has an enormous system prompt — approximately 3,500 tokens covering: the RM's role definition, FinSmart's entire product catalogue summary, RBI regulatory guidelines, compliance dos and don'ts, response formatting rules, and escalation procedures. This prefix is identical for every single RM query.

With 8,000 RM queries per day, they're processing 28 million tokens of static, never-changing prefix content daily. Their monthly API bill for this one feature alone is ₹18 lakh. The CFO has flagged it.

**Step-by-Step Solution**

Step 1: FinSmart audits their prompts and separates content into two buckets: static content that never changes per session (system role, product catalogue, compliance guidelines) and dynamic content that changes per query (the RM's specific question, the client's portfolio data pulled fresh).

Step 2: They restructure the prompt architecture so the static prefix is clearly separated from the dynamic suffix. They enable prefix caching on their API provider. The first request of the day from any RM processes the full 3,500-token prefix and caches it. Every subsequent request with the same prefix skips that computation.

Step 3: For the customer chatbot, they implement semantic response caching in Redis. They identify the 200 most frequently asked questions (FAQ analytics from the first 3 months of operation) — questions like "What is the minimum balance for a savings account?", "How do I reset my MPIN?", "What are your branch hours?" — and cache verified, compliance-approved responses for these. When a customer asks one of these questions, the system returns the cached response in under 100ms with no API call.

Step 4: Cache invalidation is automated. When a product document changes in the RAG database (e.g., savings account minimum balance changes from ₹5,000 to ₹10,000), all cached responses that reference savings account minimum balance are automatically invalidated and removed from the Redis cache, forcing fresh generation with the updated information.

Step 5: Monthly API costs for the RM assistant drop from ₹18 lakh to ₹7.2 lakh — a 60% reduction. FAQ response latency drops from 1.4 seconds to 80ms. Customer satisfaction scores for FAQ-type queries increase because near-instant responses feel dramatically better than 1.4-second waits.

---

### Pattern 13: Continuous Dynamic Batching

**The Real Problem**

FinSmart's risk and compliance team runs a nightly process: every customer transaction flagged as potentially suspicious during the day is analyzed by an AI model to generate a preliminary Suspicious Transaction Report (STR) narrative. There are typically 400–600 flagged transactions per night. Processing them sequentially takes 3.5 hours, meaning the risk team's morning briefing at 7 AM doesn't have the AI-generated narratives ready. Analysts start their day with raw data and no AI assistance for the most critical first hour.

**Step-by-Step Solution**

Step 1: FinSmart moves from a sequential processing script to a queue-based batching architecture. Flagged transactions are pushed to a processing queue throughout the day as they're flagged, not held until EOD.

Step 2: A batch coordinator aggregates transactions into batches of 15 (or up to 300ms wait, whichever fills first) and dispatches them to a self-hosted inference server running vLLM with continuous batching enabled.

Step 3: vLLM's continuous batching means it doesn't wait for all 15 transactions in a batch to complete before starting the next batch. It continuously fills GPU memory with in-progress requests and processes them in parallel, keeping GPU utilization near-optimal.

Step 4: Because batching starts throughout the day (not just overnight), the 600 transactions that used to pile up into a 3.5-hour overnight job are now processed continuously. By end of business day, 80% of the day's flagged transactions already have draft STR narratives. The remaining 20% (flagged late) are processed by midnight.

Step 5: The risk team's 7 AM briefing now arrives at 6:15 AM, fully complete, with 600 AI-generated STR narratives ready for analyst review. The morning hour that was previously spent working with raw data is now spent on higher-value review and decision-making. Processing cost per transaction drops 52% due to GPU efficiency gains.

---

### Pattern 14: Intelligent Model Routing

**The Real Problem**

FinSmart's customer chatbot handles a huge range of query complexity. An analysis of 30 days of traffic shows:

60% of queries are simple transactional requests: "What's my account balance?", "When is my credit card bill due?", "How do I block my card?", "What are your IFSC codes?" These need fast, accurate retrieval — not complex reasoning.

25% are moderate queries: "Explain the difference between a fixed deposit and a recurring deposit", "What documents do I need for a personal loan?", "How does the rewards points system work?" These need coherent explanation but not deep reasoning.

15% are genuinely complex: "I'm an NRI returning to India permanently, what do I need to do with my NRE account?", "Explain the tax implications of my FD interest across this financial year given my other income", "I want to invest ₹50 lakh for 5 years with moderate risk — compare my options."

Currently all three types go to GPT-4o. The cost per query is roughly the same for "What's my IFSC code?" and "Explain cross-border tax implications of NRE account repatriation." The first query barely uses the model's capabilities. They're massively overpaying for 60% of their traffic.

**Step-by-Step Solution**

Step 1: FinSmart trains a lightweight query classifier — a fine-tuned smaller model (about 100M parameters) on their 30-day query history, labeled by complexity level. This classifier runs in under 15ms and costs a tiny fraction of a full LLM call.

Step 2: Simple queries (60% of traffic) are routed to a two-stage system. First, the query is matched against the semantic response cache. If it's a known FAQ, the cached response is returned in under 100ms — no model call at all. If not cached, it's routed to a small, fast model (GPT-4o-mini or equivalent) with a short, focused system prompt. Account balance queries, IFSC lookups, and card blocking queries are also handled by direct API calls to FinSmart's core banking system, with the AI just formatting the response.

Step 3: Moderate queries (25% of traffic) go to GPT-4o-mini with the full RAG pipeline attached — retrieving relevant product documentation. This model handles explanatory queries well at a fraction of GPT-4o's cost.

Step 4: Complex queries (15% of traffic) are routed to GPT-4o or Claude Opus with full RAG, CoT prompting, and an extended max token limit. Only the genuinely hard problems get the full-power, higher-cost model.

Step 5: The router also implements fallback logic. If GPT-4o is rate-limited, complex queries queue briefly (up to 8 seconds, with a "please wait" message) rather than being incorrectly downrouted to the cheaper model. If the queue wait exceeds 8 seconds, the query is escalated to a human agent with the AI's partial analysis included as context.

Step 6: After 60 days of running the routing system, FinSmart's monthly chatbot API costs drop from ₹12.4 lakh to ₹5.1 lakh — a 59% reduction. Average response time for simple queries drops from 1.6 seconds to 180ms. Complex query quality is unchanged because those queries still get the best model. Customer satisfaction scores actually improve slightly because the faster simple-query responses outweigh any imperceptible difference in simple-query answer quality.

---

## The Complete FinSmart Architecture — How It All Works Together

Here's how a single customer query flows through the entire system:

A customer opens the FinSmart app and types: "I think someone made a transaction on my account that I didn't authorize."

The intelligent router classifies this as a "complex + high-stakes fraud" query in 15ms. It bypasses the cache and the small model entirely.

The RAG pipeline simultaneously retrieves FinSmart's fraud response procedures and the customer's recent transaction history from the account database.

The query is batched with 3 other concurrent fraud-adjacent queries and dispatched together to the inference server.

The model receives a role-prompted system context (customer-facing safety assistant), few-shot examples of proper fraud response (immediate protective action, not passive monitoring), and the retrieved documents. It uses chain-of-thought to assess the situation and generate a response advising the customer to immediately use the in-app card block feature, and that a fraud specialist will call within 30 minutes.

The output guardrail pipeline runs in parallel — the crisis detector checks for distress signals (none triggered), the regulatory compliance filter checks for money laundering facilitation patterns (not triggered), and the financial accuracy verifier confirms the response doesn't cite any specific figures that could be wrong.

The approved response reaches the customer. The full interaction is tagged with the release bundle ID of the prompt-model-config version that handled it. The Urgent Action Rate metric registers a positive signal. The response is sampled at 2% probability for overnight critic model evaluation.

The entire exchange — from customer tap to response displayed — takes 1.9 seconds.

That is what a well-architected AI system looks like in banking — each pattern earning its place by solving a real, specific problem, and all of them working together as a coherent, safe, efficient, and continuously improving system.
