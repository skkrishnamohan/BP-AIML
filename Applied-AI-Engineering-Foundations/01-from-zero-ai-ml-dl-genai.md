# Session notes — From zero: AI, ML, DL, GenAI

*Based on: Student Guide W1, Week 1 slides, reading “AI vs ML vs DL,” Historical Context, Current AI Landscape.*

---

## 🟢 The picture in your head (concentric circles)

Imagine nested circles:

1. **Artificial Intelligence (AI)** — broadest: machines doing tasks that *look* intelligent (recommendations, fraud alerts, chatbots).
2. **Machine Learning (ML)** — a *subset* of AI: the machine **learns from data** instead of only following hand-written rules.
3. **Generative AI (GenAI)** — creates **new** content (text, images, code), not just “spam vs not spam.”
4. **Large Language Models (LLMs)** — GenAI for language, usually built on the **Transformer** architecture (2017 paper: *Attention Is All You Need*).

**Memory trick:** AI is the campus; ML is one department; GenAI is a lab inside ML; LLMs are one kind of experiment in that lab.

---

## 🟢 Three ways a “model” decides (AI → ML → DL)

| Layer | How it decides | Human analogy |
|-------|----------------|---------------|
| **Classic AI** | Rules you wrote | Traffic cop with a manual: red → stop |
| **ML** | Learns from examples | Child learns “dog” after seeing many dogs |
| **Deep Learning** | Many layers learn features automatically | Brain builds vision in stages (edges → shapes → face) |

**Key:** All are “models” = **input → process → output**. The difference is *where the logic lives* (your code vs learned weights).

---

## 🟢 Rules vs ML vs GenAI (from class quizzes)

| Situation | Best tool | Why |
|-----------|-----------|-----|
| GST = 18% of price | **Rules** | Exact formula, no learning needed |
| UPI fraud in 15 ms | **Predictive ML** | Patterns in 50+ signals |
| Summarize 60-page RBI circular | **GenAI (LLM)** | New natural language from unstructured text |
| Password ≥12 chars + digit | **Rules** | Deterministic check |
| Tomorrow’s grid load (MW) | **ML regression** | Patterns in time series / weather |

**Career rule from faculty:** Don’t use ML when **~5 IF/ELSE rules** work. Use ML when patterns are **too complex or change too fast** to hard-code.

---

## 🟡 Why GenAI answers change each time

LLMs don’t “look up” one answer. They compute **probability for the next token** (word piece), then **sample**.

- **Temperature 0** → almost always pick the top token → stable (good for SQL/code).
- **Temperature 0.7–1.0** → more random → creative writing.

**Engineering rule:** Treat ML/GenAI as **probabilistic**. Pair with **thresholds, guardrails, and human review**.

---

## 🟢 History in 4 milestones (why MLOps exists now)

1. **Rule-based (pre-2010)** — predictable, explainable, brittle at scale.
2. **Classical ML** — learns from data; humans still craft **features**.
3. **Deep learning** — learns features from raw images/text/audio; needs **GPU + big data**.
4. **Foundation models / LLMs** — huge pretrained models; fine-tune or prompt for tasks.

**Takeaway:** Modern AI is an **engineering** problem: data volume, deployment, **drift**, cost, safety.

---

## 🟡 AI in the real world (landscape)

Don’t start with “which algorithm?” Start with:

**Problem → Decision → Data → System → Model**

AI fits when decisions are:

- High volume, repetitive, time-sensitive, costly if wrong, or too complex for manual rules.

Examples: fraud (finance), triage support (health), demand forecast (energy), defect detection (manufacturing).

---

## 🟢 Glossary (pin this)

| Term | Plain English |
|------|----------------|
| **Feature (X)** | Input the model sees (age, income, pixels, words) |
| **Label / target (y)** | Correct answer in training (fraud yes/no, price, species) |
| **Weight (w)** | How important each feature is (learned, not guessed) |
| **Bias (b)** | Baseline “starting point” before features |
| **Inference** | Using a trained model on **new** data |
| **Ground truth** | The true label you trust (when you have it) |

---

## Checkpoint (answer without peeking)

1. Is “predict apartment price in ₹” classification or regression?
2. Why is 99% accuracy misleading for 1% fraud rate?
3. Name one task for rules and one for GenAI.

*Answers in [04-classification-regression-and-metrics.md](04-classification-regression-and-metrics.md) and [07-case-studies-quizzes-and-coding-lab.md](07-case-studies-quizzes-and-coding-lab.md).*

---

**Next:** [02-rules-learning-and-ai-sdlc.md](02-rules-learning-and-ai-sdlc.md)
