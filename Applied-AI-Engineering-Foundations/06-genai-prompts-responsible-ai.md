# Session notes — GenAI, prompts, responsible AI

*Based on: Student Guide W1 (GenAI, RLHF, prompting), evolutionary stages, responsible AI pillars.*

---

## 🟢 GenAI in one paragraph

**Generative AI** creates new content (text, images, audio) by learning statistical patterns from huge datasets. **LLMs** predict the **next token** given context — they are not databases of facts.

Training phases (high level):

1. **Pretraining** — learn language structure from massive text  
2. **RLHF** — humans rate outputs → model learns helpful/safe style  
3. **Safety guardrails** — refuse harmful requests  

---

## 🟢 Human–AI partnership

You still own:

- **What to ask** (task + context)
- **Whether to trust** the output (hallucinations happen)
- **Compliance** (PII, regulations)

> AI may not take your job; people using AI productively might.

---

## 🟢 Good prompt structure (from student guide)

| Part | Purpose |
|------|---------|
| **Task** | What you want (“agenda for workshop”) |
| **Context** | Who, why, constraints (aerospace leaders, 4 hours) |
| **Clarifying invitation** | “Ask me questions before you start” |

**Bad prompt:** “Write something about AI.”  
**Better:** Task + audience + format + constraints + invite clarifications.

---

## 🟡 Stages of AI evolution (memorize lightly)

1. Rule-based  
2. Context-based (personalization)  
3. Narrow domain experts  
4. Reasoning models  
5. **AGI** (aspirational)  
6. Superintelligence (theoretical)

You’re firmly in **3–4** today for products.

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

---

## 🟡 Emergent capabilities

Larger models gain skills not explicitly programmed (coding, reasoning-ish behavior). Still **probabilistic** — verify outputs.

**Multimodality:** text + image + audio in one model (“whatever-to-whatever”).

---

## Checkpoint

Name two responsibilities humans keep even with a strong LLM.

**Next:** [07-case-studies-quizzes-and-coding-lab.md](07-case-studies-quizzes-and-coding-lab.md)
