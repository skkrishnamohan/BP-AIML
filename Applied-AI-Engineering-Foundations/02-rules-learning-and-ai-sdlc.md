# Session notes — Traditional SDLC vs AI SDLC

*Based on: Week 1 slides, reading “Traditional SDLC vs AI SDLC,” “Phases of AI-Specific SDLC.”*

**SDLC** means **Software Development Life Cycle**: the repeating path from a need, through design, build, test, release, and maintenance. An **AI SDLC** is that path when part of the behavior comes from data and a trained model rather than only from source code.

---

## 🟢 The one idea to memorize

> **In AI systems, data is part of the code.**

- Traditional app: change behavior → change **code**.
- AI system: behavior can change when **data**, **labels**, or **training** change — even if application code stays the same.

### What this sentence means in practice

**What:** Two deployments can run the same Python service and make different decisions because one was trained on January’s transactions and the other on July’s.

**Why:** If you version only Git commits, you cannot explain or reproduce a decision. The missing ingredient is the dataset, the labels, and the training run.

**How:** Treat a training dataset like a dependency. Record its version next to the code version and the metric you promised.

**Where:** Loan models, spam filters, demand forecasts, and any notebook that someone later “just retrains.”

**Meaning of “behavior”:** Not the button color. Behavior is the decision: approve, decline, flag, or the sentence the assistant writes.

---

## 🟢 Traditional SDLC (what you already know)

| Property | Meaning |
|----------|---------|
| **Deterministic** | Same input → same output |
| **Logic in code** | if/else, functions |
| **Bugs** | Reproduce with logs |
| **Deploy** | Near “end” of project; monitor uptime/errors |

**Example:** Login if password matches.

**What deterministic means:** Give the function the same password hash and you get the same allow or deny, today and next year, unless someone edits the code.

**Why teams like it:** A bug has a stack trace. You replay the request. You write a test that fails, then a fix that passes.

**How a traditional release works:** Requirements → design → implement → test → deploy → monitor uptime. After deploy, the main questions are “is it up?” and “are there errors?” The decision rule itself does not rot when customers change habits.

**Where it still wins:** Tax, authentication, eligibility rules that a lawyer can point to line by line, and anything that must be identical every time.

---

## 🟢 AI SDLC (what’s different)

| Property | Meaning |
|----------|---------|
| **Probabilistic** | Scores, rankings, generated text — can vary |
| **Logic in data + model** | Weights learned from history |
| **Failures** | Often **silent** (slow drift) |
| **Deploy** | **Beginning** of improvement (monitor, retrain) |

**Example:** Spam filter — spammers adapt; model degrades without retraining.

**What silent failure means:** The API still returns HTTP 200. Latency is fine. The fraction of spam that lands in the inbox slowly rises. No exception is thrown, because the model is doing exactly what its weights say. The world changed.

**Why deploy is the start:** The first day in production is the first day the model sees truly new traffic. Offline accuracy was a rehearsal. Monitoring tells you whether the rehearsal still matches the audience.

**How you operate it:** Log inputs, scores, and downstream outcomes. Compare this week’s feature distributions with the training week. Retrain or roll back when the gap becomes a business problem, not merely because a calendar said so.

**Where you see it:** Fraudsters change devices and amounts. A weather sensor is replaced. A new product line has no history. The code did not break; the data distribution moved. That movement is **drift**.

---

## Side-by-side (exam-friendly)

| Traditional | AI |
|-------------|-----|
| Behavior in code | Behavior in data + model |
| Deterministic | Probabilistic |
| Test correctness | Test performance + robustness |
| Deploy = done | Deploy = start monitoring |

**Correctness vs performance:** A login test is pass/fail. A fraud model is never perfectly correct. You test whether precision and recall meet a bar, whether every segment is treated acceptably, and whether small input changes cause wild output changes (**robustness**).

**Robustness** means the decision stays sensible under noise, missing values, and slight shifts. A model that collapses when one column arrives in a new unit is not robust.

---

## 🟡 AI lifecycle loop (8–9 phases)

Think **loop**, not waterfall. A **waterfall** finishes each phase once and moves only forward. AI work returns to data and training after monitoring.

1. **Problem definition** — Who benefits? What **metric**? What **decision** changes?
2. **Data collection & preprocessing** — sourcing, cleaning, train/val/test split
3. **Feature engineering** — signals the model can use **at prediction time**
4. **Model training** — baselines first, track experiments
5. **Evaluation** — precision/recall, segments, fairness where needed
6. **Deployment** — batch, API, or edge; same preprocessing as training
7. **Monitoring** — latency, drift, business KPIs
8. **Ethics & compliance** — privacy, explainability, audit trail
9. **Iteration** — retrain when drift or goals change

### What each phase is asking

| Phase | What | Why it exists | A failure if you skip the point |
|-------|------|---------------|----------------------------------|
| Problem definition | The decision, the user, the metric | Stops you training a model nobody acts on | A rain model with no time it must be ready |
| Data collection | Gather examples you have a legal right to use | The model can only repeat patterns that are in the sample | City stations only, then surprise at a rural site |
| Preprocessing | Clean types, units, missing values | Raw files hide defects that do not crash code | Fahrenheit stored in a Celsius column |
| Train / validation / test | Three jobs for three slices of data | Train learns, validation chooses settings, test is the final exam taken once | Tuning on the test set until the score looks good |
| Feature engineering | Build columns available at decision time | A perfect column that only exists after the event is cheating | Using tomorrow’s rainfall to predict tomorrow’s rainfall |
| Training | Fit a simple baseline, then a stronger model | A baseline tells you whether the complex model earned its complexity | Starting with a deep net and no comparison |
| Evaluation | Metrics and slices, not one average | Average accuracy hides a group the model fails | 99% accuracy and zero fraud caught |
| Deployment | Put the same transforms in the serving path | Training-only cleanup means live data looks different | Scaled with training mean, served unscaled |
| Monitoring | Watch score, data, and KPI | Silent decay | Spam catch rate falls and nobody has a graph |
| Ethics & compliance | Privacy, explanation, audit | A legal and human constraint, not a slide at the end | No record of which data produced a denial |
| Iteration | Change data or model on purpose | The loop closes | A one-time notebook nobody can rerun |

**Common failures from reading:**

- Building AI where **rules** suffice
- Optimizing **accuracy** when business needs **recall**
- **Data leakage** (test info in training)
- No **monitoring** after launch

**Data leakage** means information from outside the training moment sneaks into the features. The model looks brilliant offline and fails live, because that clue will not exist in time.

**Baseline:** The simplest honest model, such as “predict the average” or a logistic regression. If a complex model cannot beat it by enough to matter, ship the simple one.

---

## 🟡 Class diagram: AI lifecycle on slides

```
Problem → Data → Model → Deploy → Monitor → (back to Data/Model)
```

**Monitoring** connects back because **data drift** and user behavior change.

**Data drift:** The inputs move. Customers are younger, or humidity readings sit higher than in training.

**Concept drift:** The relationship moves. The same inputs no longer mean the same outcome, because fraud tactics or climate changed.

**Label delay:** You may not know the true outcome for days (a chargeback, a medical result). Monitoring has to live with late labels rather than pretending yesterday’s score was confirmed.

---

## 🟡 Feasibility gates (from Week 1)

Before building:

1. **Gate 1:** Can <10 static rules solve it? → use traditional code
2. **Gate 2:** Is outcome pure noise (coin toss)? → **don’t build**
3. Else pick model family by data shape (tabular / text / images)

**GIGO:** Garbage in, garbage out — data quality beats fancy algorithms.

**What a gate is:** A stop/go check. Failing a gate is a successful project decision. You avoided a model that cannot work.

**Why the coin-toss gate exists:** Machine learning finds signal. If the label is unrelated to every feature you can legally collect, more data and a bigger model still produce a guess. An IPL coin toss is the course’s example of an outcome that is pure chance.

**How you pick a family after the gates:**

| Shape of data | Usual starting family | Why |
|---------------|----------------------|-----|
| Tabular rows | Logistic regression, trees, gradient boosting | Columns already have meaning |
| Text | Embeddings or an LLM, depending on whether you classify or write | Words are sequences, not tidy numbers |
| Images | Convolutional nets or vision transformers | The signal is spatial |

**GIGO expanded:** The algorithm cannot invent a clean label that the dataset does not contain. Cleaning and a clear target beat switching libraries.

---

## 🟢 What “good” looks like at problem definition

- Clear **business KPI** (not only model accuracy)
- **Integration plan** (who acts on the score?)
- Constraints: latency, cost, explainability, compliance

**Mini exercise:** Rain for farmers — decision = skip irrigation if P(rain) > 0.7 by 6am.

**Model metric vs business KPI:** Accuracy might be 0.80. The KPI is “irrigation cycles skipped without drying the crop.” A model can win on accuracy and still warn too late to be useful. Write both numbers down at the start.

**Integration plan:** Name the human or the system that consumes the score. If nobody’s workflow changes, the model is a report, not a product.

**Constraints, in words:**

| Constraint | Meaning |
|------------|---------|
| **Latency** | The maximum wait. 6 a.m. batch is fine for irrigation; 15 ms matters for UPI. |
| **Cost** | Training, storage, and every prediction. An LLM call per transaction can cost more than the fraud it stops. |
| **Explainability** | Ability to say which inputs drove a decision, often required when money or rights are affected. |
| **Compliance** | Laws and policy: consent, retention, what you are allowed to use as a feature. |

---

## Checkpoint

1. Why is deployment the *beginning* in AI SDLC?
2. Name two failure modes in the data collection phase.
3. What’s the difference between model metrics and business KPIs?

**Answers to check against:**

1. Production is the first stream of truly new cases, and the world keeps moving, so you monitor and retrain after release.
2. Examples: collecting the wrong population, missing consent or licence, labels that arrive too late, or columns that will not exist at prediction time.
3. A model metric scores predictions against labels (recall, RMSE). A KPI scores the business result (fraud losses, water used, time saved).

**Next:** [03-how-a-model-learns.md](03-how-a-model-learns.md)
