# Session notes — Traditional SDLC vs AI SDLC

*Based on: Week 1 slides, reading “Traditional SDLC vs AI SDLC,” “Phases of AI-Specific SDLC.”*

---

## 🟢 The one idea to memorize

> **In AI systems, data is part of the code.**

- Traditional app: change behavior → change **code**.
- AI system: behavior can change when **data**, **labels**, or **training** change — even if application code stays the same.

---

## 🟢 Traditional SDLC (what you already know)

| Property | Meaning |
|----------|---------|
| **Deterministic** | Same input → same output |
| **Logic in code** | if/else, functions |
| **Bugs** | Reproduce with logs |
| **Deploy** | Near “end” of project; monitor uptime/errors |

**Example:** Login if password matches.

---

## 🟢 AI SDLC (what’s different)

| Property | Meaning |
|----------|---------|
| **Probabilistic** | Scores, rankings, generated text — can vary |
| **Logic in data + model** | Weights learned from history |
| **Failures** | Often **silent** (slow drift) |
| **Deploy** | **Beginning** of improvement (monitor, retrain) |

**Example:** Spam filter — spammers adapt; model degrades without retraining.

---

## Side-by-side (exam-friendly)

| Traditional | AI |
|-------------|-----|
| Behavior in code | Behavior in data + model |
| Deterministic | Probabilistic |
| Test correctness | Test performance + robustness |
| Deploy = done | Deploy = start monitoring |

---

## 🟡 AI lifecycle loop (8–9 phases)

Think **loop**, not waterfall:

1. **Problem definition** — Who benefits? What **metric**? What **decision** changes?
2. **Data collection & preprocessing** — sourcing, cleaning, train/val/test split
3. **Feature engineering** — signals the model can use **at prediction time**
4. **Model training** — baselines first, track experiments
5. **Evaluation** — precision/recall, segments, fairness where needed
6. **Deployment** — batch, API, or edge; same preprocessing as training
7. **Monitoring** — latency, drift, business KPIs
8. **Ethics & compliance** — privacy, explainability, audit trail
9. **Iteration** — retrain when drift or goals change

**Common failures from reading:**

- Building AI where **rules** suffice
- Optimizing **accuracy** when business needs **recall**
- **Data leakage** (test info in training)
- No **monitoring** after launch

---

## 🟡 Class diagram: AI lifecycle on slides

```
Problem → Data → Model → Deploy → Monitor → (back to Data/Model)
```

**Monitoring** connects back because **data drift** and user behavior change.

---

## 🟡 Feasibility gates (from Week 1)

Before building:

1. **Gate 1:** Can <10 static rules solve it? → use traditional code
2. **Gate 2:** Is outcome pure noise (coin toss)? → **don’t build**
3. Else pick model family by data shape (tabular / text / images)

**GIGO:** Garbage in, garbage out — data quality beats fancy algorithms.

---

## 🟢 What “good” looks like at problem definition

- Clear **business KPI** (not only model accuracy)
- **Integration plan** (who acts on the score?)
- Constraints: latency, cost, explainability, compliance

**Mini exercise:** Rain for farmers — decision = skip irrigation if P(rain) > 0.7 by 6am.

---

## Checkpoint

1. Why is deployment the *beginning* in AI SDLC?
2. Name two failure modes in the data collection phase.
3. What’s the difference between model metrics and business KPIs?

**Next:** [03-how-a-model-learns.md](03-how-a-model-learns.md)
