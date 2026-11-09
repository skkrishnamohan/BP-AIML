# Session notes — GenAI, prompts, responsible AI

*Based on: Student Guide W1 (GenAI, RLHF, prompting), evolutionary stages, responsible AI pillars.*

Generative models feel conversational. The engineering questions are the same as the rest of this module: what is the system allowed to know, why should you trust this output, and where does a human stay accountable?

---

## 🟢 GenAI in one paragraph

**Generative AI** creates new content (text, images, audio) by learning statistical patterns from huge datasets. **LLMs** predict the **next token** given context — they are not databases of facts.

Training phases (high level):

1. **Pretraining** — learn language structure from massive text
2. **RLHF** — humans rate outputs → model learns helpful/safe style
3. **Safety guardrails** — refuse harmful requests

### What each phase actually changes

| Phase | What happens | Why it exists | What it does not do |
|-------|--------------|---------------|---------------------|
| **Pretraining** | The model reads a huge corpus and practices next-token prediction | Gives the model grammar, facts it saw often, and general skills | It does not know your private documents, and it does not guarantee truth |
| **RLHF** | Reinforcement learning from human feedback. People rank answers; a reward model learns those preferences; the LLM is updated to prefer higher-ranked answers | Makes answers more helpful, polite, and aligned with instructions | It does not turn the model into a verified database |
| **Guardrails** | Filters and policies around the model, before the prompt or after the answer | Stop disallowed content even if the base model would produce it | They are not perfect. They are a layer, like a threshold on a fraud model |

**Reinforcement learning** is a training style where the model gets a reward signal rather than only a single correct label. **RLHF** uses human preferences as that signal.

**Why “not a database” matters:** Ask for a citation and the model may produce a real-looking title that does not exist. That failure is a **hallucination**: fluent text that is not grounded. The fix is to retrieve a trusted document and require the answer to stay inside it (retrieval-augmented generation, taught in Module 2), or to keep a human on the decision.

**Where you see the three phases:** The chat product you use has all three. Pretraining is the base model. RLHF is why it follows “answer briefly” better than a raw pretrained model. Guardrails are why some requests are refused before a normal completion.

**Token, restated:** A token is a chunk of text, often part of a word. Context length is how many tokens the model can consider at once. A 60-page circular may not fit, which is why summarization pipelines chunk documents.

---

## 🟢 Human–AI partnership

You still own:

- **What to ask** (task + context)
- **Whether to trust** the output (hallucinations happen)
- **Compliance** (PII, regulations)

> AI may not take your job; people using AI productively might.

**What “you own the ask” means:** The model completes a pattern. A vague pattern produces a vague completion. Specifying the user, the decision, the format, and the constraint is part of the engineering, the same way defining `y` is part of supervised learning.

**What “you own trust” means:** The model does not know when it is guessing. Verification is an external step: check a source, run the code, compare with a policy, or have a reviewer sign off.

**PII** means **personally identifiable information**: names, account numbers, phone numbers, anything that can identify a person. Pasting a customer’s full record into a public chatbot can be a compliance incident even if the answer that comes back is useful.

**Where compliance shows up:** Regulated text such as an RBI circular may be summarized, but the summary is not itself the regulation. A loan denial still needs an approved decision process, not only a well-written paragraph.

---

## 🟢 Good prompt structure (from student guide)

| Part | Purpose |
|------|---------|
| **Task** | What you want (“agenda for workshop”) |
| **Context** | Who, why, constraints (aerospace leaders, 4 hours) |
| **Clarifying invitation** | “Ask me questions before you start” |

**Bad prompt:** “Write something about AI.”
**Better:** Task + audience + format + constraints + invite clarifications.

### Why the three parts work

**Task** is the verb and the artifact. “Write an agenda” is checkable. “Write something” is not.

**Context** is the condition that changes the artifact. Aerospace leaders for four hours is a different agenda from schoolchildren for twenty minutes. Leaving it out forces the model to invent an audience, and it will, confidently.

**Clarifying invitation** stops the model from filling gaps with fiction. It is the prompt equivalent of refusing to impute a label you do not know.

**A fuller prompt also names:**

| Extra piece | What it controls | Example |
|-------------|------------------|---------|
| **Format** | Shape of the output | A table with time, topic, and outcome |
| **Constraints** | What must not happen | No vendor pitches, no more than six sessions |
| **Evidence** | What the model may rely on | Only the attached brief; say “not in the brief” otherwise |
| **Success check** | How you will grade it | Each session has one verb and one deliverable |

**How to test a prompt:** Run it several times. If the structure collapses, the instruction is too soft. If the facts change, the prompt needs a source document, not a higher temperature.

**Where prompts are not the whole system:** A prompt cannot enforce a 15 ms fraud decision or a legal eligibility rule. Those stay code. The prompt belongs where the output is language and a human or a checker is in the loop.

---

## 🟡 Stages of AI evolution (memorize lightly)

1. Rule-based
2. Context-based (personalization)
3. Narrow domain experts
4. Reasoning models
5. **AGI** (aspirational)
6. Superintelligence (theoretical)

You’re firmly in **3–4** today for products.

| Stage | What it means | A real example | Why it is not the next stage |
|-------|---------------|----------------|------------------------------|
| **Rule-based** | Humans wrote the logic | GST at 18% | It does not adapt |
| **Context-based** | The system changes with user or session context | Recommendations that use your history | It is still a narrow predictor |
| **Narrow expert** | Strong on one domain | A fraud model, a weather model, a coding assistant in a sandbox | It does not transfer to every job |
| **Reasoning model** | Spends extra steps trying to chain a solution | A model that shows work on a multi-step problem | The chain can still be wrong |
| **AGI** | Artificial general intelligence: human-level breadth across tasks | Not a product you can buy | The word is a goal, not a description of today’s API |
| **Superintelligence** | Hypothetical systems beyond human level across the board | Not an engineering input | Do not use it to skip metrics and monitoring |

**Why the course says you are in 3–4:** The tools you call are excellent at specific tasks and can attempt multi-step reasoning. They are not general agents that can be handed an open-ended company and trusted. Shipping them still requires the SDLC from the earlier notes.

---

## 🟢 Responsible AI pillars (course)

| Pillar | Meaning |
|--------|---------|
| **Unbiased** | Reduce unfair prejudice in data/model |
| **Human-centred** | Design for people, not only metrics |
| **Secure & reproducible** | Protect data; repeat experiments |
| **Explainable** | Know *why* a decision happened |
| **Monitorable** | Watch performance and safety in production |

Connects directly to **Module 2** (bias, SHAP, data cards, trust gates).

### What each pillar asks you to do

**Unbiased.** Compare errors and outcomes across groups that could be harmed. A high overall accuracy can hide a group that is always rejected. Module 2 names this with disparate impact and proxies. “We did not include the sensitive column” is not the end of the check, because other columns can stand in for it.

**Human-centred.** Name the person who benefits and the person who is harmed if the model is wrong. A farmer who skips irrigation and a patient who is triaged are not “users” in the abstract. Design the review path for them.

**Secure and reproducible.** Secure means access control, encryption, and not leaking PII into logs or third-party prompts. Reproducible means a later audit can rebuild the same data version, code version, and metric. If you cannot rerun it, you cannot defend it.

**Explainable.** For a scored decision, be able to say which inputs moved it. For a generated paragraph, be able to say which source it was supposed to use. Explanation is for the affected person and for the engineer debugging a mistake. SHAP is one feature-level tool, not the whole pillar.

**Monitorable.** Define what you will watch before launch: data drift, refusal rate, hallucination checks on a fixed prompt set, latency, and the business KPI. A pillar you cannot graph will be skipped on a busy week.

**Where the pillars sit in the SDLC:** They are not a phase 8 slide you visit once. Bias starts at collection. Explanation starts when you choose a glass-box or a black-box. Monitoring starts at deployment, which the AI SDLC treats as the beginning.

---

## 🟡 Emergent capabilities

Larger models gain skills not explicitly programmed (coding, reasoning-ish behavior). Still **probabilistic** — verify outputs.

**Multimodality:** text + image + audio in one model (“whatever-to-whatever”).

**What “emergent” means:** A skill that was not a separate labeled project, but appears once the model and the data are large enough. It is an observation about scale, not a promise that the skill is reliable.

**Why you still verify:** An emergent coding skill can produce code that runs and is wrong. Probability did not go away because the output looks like reasoning.

**Multimodal, unpacked:** The model maps more than one kind of input into a shared representation. A picture plus a question, or speech plus a document. “Whatever-to-whatever” is the slogan; the engineering limit is still context length, latency, and whether you have a way to check the answer.

**Where this changes design:** A weather assistant that accepts a radar image and a bulletin is multimodal. The rain decision should still come from a measured pipeline with a metric, not only from a caption the model wrote about the image.

---

## Checkpoint

Name two responsibilities humans keep even with a strong LLM.

**Answer:** You still decide what to ask and what context the model is allowed to see, and you still decide whether the output is true, safe, and compliant enough to act on. A third that counts: you own the monitoring and the audit trail after the system is live.

**Next:** [07-case-studies-quizzes-and-coding-lab.md](07-case-studies-quizzes-and-coding-lab.md)
