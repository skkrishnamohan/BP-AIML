# Session notes — From zero: AI, ML, DL, GenAI

*Based on: Student Guide W1, Week 1 slides, reading “AI vs ML vs DL,” Historical Context, Current AI Landscape.*

This file names the family of technologies, then shows **when** each one is the right tool. Every section answers what the thing is, why anyone uses it, how it decides, and where it shows up.

---

## 🟢 The picture in your head (concentric circles)

Imagine nested circles. The outer circle contains the inner ones. Saying “we will use AI” does not tell you which inner circle you are in.

1. **Artificial Intelligence (AI)** — broadest: machines doing tasks that *look* intelligent (recommendations, fraud alerts, chatbots).
2. **Machine Learning (ML)** — a *subset* of AI: the machine **learns from data** instead of only following hand-written rules.
3. **Deep Learning (DL)** — a subset of ML: many layers of a neural network learn features from raw inputs such as pixels or tokens.
4. **Generative AI (GenAI)** — creates **new** content (text, images, code), not just “spam vs not spam.” Most modern GenAI is deep learning, but not all deep learning generates new content. A face detector is DL and is usually **discriminative** (it labels), not generative.
5. **Large Language Models (LLMs)** — GenAI for language, usually built on the **Transformer** architecture (2017 paper: *Attention Is All You Need*).

**Memory trick:** AI is the campus; ML is one department; DL is a method inside that department; GenAI is a lab that creates new artifacts; LLMs are one kind of experiment in that lab.

### What / why / how / where

| | |
|--|--|
| **What** | A map of terms so “AI,” “ML,” and “ChatGPT-style models” are not treated as synonyms. |
| **Why** | The wrong circle leads to the wrong project. A tax formula does not need an LLM. A 60-page policy summary is a poor fit for a five-line `if`. |
| **How** | Ask: does the system follow rules I wrote, learn a pattern, or produce new content? |
| **Where** | Product pitches, architecture reviews, and the Week 1 quizzes. |

**Discriminative vs generative:** A discriminative model answers “which class is this?” A generative model answers “what new thing could come next?” Fraud scoring is discriminative. Drafting an email is generative.

---

## 🟢 Three ways a “model” decides (AI → ML → DL)

| Layer | How it decides | Human analogy |
|-------|----------------|---------------|
| **Classic AI** | Rules you wrote | Traffic cop with a manual: red → stop |
| **ML** | Learns from examples | Child learns “dog” after seeing many dogs |
| **Deep Learning** | Many layers learn features automatically | Brain builds vision in stages (edges → shapes → face) |

**Key:** All are “models” = **input → process → output**. The difference is *where the logic lives* (your code vs learned weights).

### Definitions

- **Model:** A function that turns inputs into an output. In rules, you wrote the function. In ML, training estimates the function from examples.
- **Rule:** An explicit instruction such as “if password length < 12, reject.” Same input always gives the same output. That property is called **deterministic**.
- **Learning:** Adjusting internal numbers so the model’s mistakes on past examples get smaller.
- **Feature (human-made):** A column you designed, such as “distance from home.” Classical ML often needs these.
- **Representation (learned):** A pattern the network discovers, such as “this patch of pixels is an edge.” Deep learning is valuable when you cannot hand-write those patterns.

**Why deep learning is not always better:** On a small table of age and income, a simple model is easier to explain, cheaper to train, and often just as accurate. Deep learning earns its cost when the raw signal is an image, a long document, or audio.

**Where the logic lives:** Open the source of a rules engine and you can read the decision. Open a trained network and you see numbers called **weights**. The decision procedure is those numbers plus the architecture, which is why explanation tools show up later in the course.

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

### Say each abbreviation out loud

| Short form | Full form | Meaning in the row above |
|------------|-----------|--------------------------|
| **GST** | Goods and Services Tax | A legally fixed percentage. There is nothing to learn. |
| **UPI** | Unified Payments Interface | India’s real-time payment rail. Fraud decisions must finish in milliseconds, so the model scores a transaction as it happens. |
| **RBI** | Reserve Bank of India | The regulator. A circular is long unstructured text; the useful output is a new summary, which is generation. |
| **MW** | Megawatt | A continuous amount of electrical power. Predicting a number is **regression**, not a yes/no class. |
| **ms** | Millisecond | One thousandth of a second. “15 ms” means the score must be ready before the payment flow times out. |

**How to choose in a meeting:**

1. Is the correct answer a formula or a policy you can write completely? Use rules.
2. Is the correct answer a label or a number that humans can show with past examples, but cannot write as a short formula? Use predictive ML.
3. Is the useful output new language, code, or media? Use GenAI, and still check facts.
4. If a task mixes them, split it. Example: an LLM drafts a fraud analyst’s note, but a classical model produces the risk score. Do not ask the LLM to be the only control on a 15 ms payment decision.

**Where this fails:** People reach for GenAI because it is new. A password rule implemented as a prompt is slower, less reliable, and harder to audit than `len(password) >= 12`.

---

## 🟡 Why GenAI answers change each time

LLMs don’t “look up” one answer. They compute **probability for the next token** (word piece), then **sample**.

- **Temperature 0** → almost always pick the top token → stable (good for SQL/code).
- **Temperature 0.7–1.0** → more random → creative writing.

**Engineering rule:** Treat ML/GenAI as **probabilistic**. Pair with **thresholds, guardrails, and human review**.

### What a token and a temperature actually are

**Token:** The unit the model predicts. It is often a word piece, not a whole word. “unbelievable” might be several tokens. The model does not retrieve a stored paragraph. It repeatedly asks, “given everything so far, which token is likely next?”

**Probability distribution:** For the next position the model has a score for every token in its vocabulary. Sampling means drawing from that list. The most likely token is not the only legal one, which is why two calls can differ.

**Temperature:** A dial on how peaked that distribution is. Low temperature makes the top token dominate. High temperature lets less likely tokens win more often. It is not a measure of factual accuracy. Temperature 0 can still be confidently wrong.

**Why this matters:** A support bot that must return the same refund policy every time should not be highly sampled. A brainstorming assistant can be. Neither setting removes **hallucination**, which means a fluent statement that is not grounded in a trusted source.

**Where you control it:** API parameters (`temperature`, sometimes `top_p`), product settings, and eval tests that run the same prompt many times and measure how often the answer changes.

| Term | Meaning |
|------|---------|
| **Probabilistic** | The system returns a score or a sample, not a proof. |
| **Threshold** | A cutoff you choose, such as “block only if fraud probability ≥ 0.92.” |
| **Guardrail** | A check around the model: block disallowed topics, require a citation, or reject invalid JSON. |
| **SQL** | Structured Query Language. Temperature near 0 is preferred because a creative comma can break a query. |

---

## 🟢 History in 4 milestones (why MLOps exists now)

1. **Rule-based (pre-2010)** — predictable, explainable, brittle at scale.
2. **Classical ML** — learns from data; humans still craft **features**.
3. **Deep learning** — learns features from raw images/text/audio; needs **GPU + big data**.
4. **Foundation models / LLMs** — huge pretrained models; fine-tune or prompt for tasks.

**Takeaway:** Modern AI is an **engineering** problem: data volume, deployment, **drift**, cost, safety.

### Why the history is an engineering story

**What changed** is not only accuracy. Each stage moved work from “a developer writes logic” to “a pipeline produces a model that must be fed, deployed, and watched.”

**Rule-based systems** are easy to audit and painful to maintain when the world has thousands of exceptions. A fraud rule “amount > ₹50,000 and new device” is clear, and criminals learn to stay just under it.

**Classical ML** (logistic regression, decision trees, gradient boosting) learns those exceptions from rows in a table. A human still decides which columns exist. That design work is **feature engineering**.

**Deep learning** removes much of that design for images, speech, and text, and sends the bill to hardware. A **GPU** (graphics processing unit) does the huge matrix math. A normal **CPU** is often too slow for training large networks.

**Foundation model:** A model pretrained on a very broad dataset so many tasks can start from it. You **prompt** it (give instructions at request time) or **fine-tune** it (continue training on your smaller dataset). You usually do not train it from empty weights.

**Drift:** The world moves away from the data the model studied. Last year’s spam no longer looks like this year’s spam. The code did not change; the correct answers did. That is why MLOps exists: training is not the end of the job.

**Where you feel this:** A model that was approved in January quietly catches less fraud in July. Monitoring, not another algorithm name, is the fix.

---

## 🟡 AI in the real world (landscape)

Don’t start with “which algorithm?” Start with:

**Problem → Decision → Data → System → Model**

AI fits when decisions are:

- High volume, repetitive, time-sensitive, costly if wrong, or too complex for manual rules.

Examples: fraud (finance), triage support (health), demand forecast (energy), defect detection (manufacturing).

### Walk the chain before you name an algorithm

| Step | Question you must answer | Weather example |
|------|--------------------------|-----------------|
| **Problem** | Whose pain is this? | A farmer waters a field that then gets rain. |
| **Decision** | What action changes? | Skip irrigation when rain is likely. |
| **Data** | What can we know before the decision? | Yesterday’s pressure, humidity, radar. Not tomorrow’s rain. |
| **System** | Where does the score go, and how fast? | A 6 a.m. batch file to the planner’s app. |
| **Model** | Which family fits that data? | Tabular time series first, not a chatbot. |

**Why this order:** Choosing “we will use a transformer” before you know the decision produces a demo, not a system. The model is the last box because it is replaceable. The decision and the data contract are not.

**Where AI is a bad fit:** One-off judgments with no past examples, decisions that are pure chance, or places where a wrong automated action cannot be reviewed and the harm is irreversible. The course later calls the pure-chance case infeasible (a coin toss).

**Triage** means sorting cases by urgency so a human sees the risky ones first. The model does not replace the clinician or the fraud analyst; it ranks the queue.

---

## 🟢 Glossary (pin this)

| Term | Plain English |
|------|----------------|
| **Feature (X)** | Input the model sees (age, income, pixels, words) |
| **Label / target (y)** | Correct answer in training (fraud yes/no, price, species) |
| **Weight (w)** | How important each feature is (learned, not guessed) |
| **Bias (b)** | Baseline “starting point” before features. In this math sense it is not social bias. |
| **Inference** | Using a trained model on **new** data |
| **Training** | The earlier phase that adjusts weights using historical examples |
| **Ground truth** | The true label you trust (when you have it) |
| **Algorithm** | The learning procedure (how weights are updated) |
| **Architecture** | The shape of the model (one neuron, a tree, a transformer) |
| **Parameter** | A number the training procedure learns (weights and biases) |

**Inference vs training:** Training may take hours and looks at the answers. Inference takes milliseconds and must not look at the answer, because in production the answer has not happened yet.

---

## Checkpoint (answer without peeking)

1. Is “predict apartment price in ₹” classification or regression?
2. Why is 99% accuracy misleading for 1% fraud rate?
3. Name one task for rules and one for GenAI.

*Answers in [04-classification-regression-and-metrics.md](04-classification-regression-and-metrics.md) and [07-case-studies-quizzes-and-coding-lab.md](07-case-studies-quizzes-and-coding-lab.md).*

**How to grade yourself:** (1) Regression, because rupees are a continuous number. (2) A model can ignore every fraud and still be right on the 99% of legitimate payments. (3) GST or password length is rules; summarizing a circular is GenAI.

---

**Next:** [02-rules-learning-and-ai-sdlc.md](02-rules-learning-and-ai-sdlc.md)
