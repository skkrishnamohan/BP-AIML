# Session notes — How a model learns (neuron → logistic regression → neural nets)

*Based on: Student Guide, Week 1 slides (bank auto-loan), insurance coding material.*

---

## 🟢 Everything is numbers

Neural networks don’t read English or pixels directly. Data becomes **numbers** (vectors). Learning = adjusting **weights** so predictions get closer to true **labels**.

---

## 🟢 One neuron (the “bouncer” analogy)

Steps:

1. **Inputs** \(x_1, x_2, ...\) — e.g. age, income, education
2. **Weights** \(w_i\) — importance dials (positive = pushes toward “yes”)
3. **Bias** \(b\) — starting mood
4. **Weighted sum:** \(z = w_1 x_1 + w_2 x_2 + ... + b\)
5. **Activation** — squash \(z\) to a probability (sigmoid for binary yes/no)

**One neuron + sigmoid ≈ logistic regression** (linear boundary in feature space).

---

## 🟢 Sigmoid and the 0.5 rule

Sigmoid maps any \(z\) to **(0, 1)** = probability.

- If \(p \geq 0.5\) → predict class **1** (e.g. “will buy car”)
- If \(p < 0.5\) → predict class **0**

**Business insight:** Banks use **tiers** (e.g. 88% → pre-approved loan), not only 0/1.

---

## 🟡 Classification vs regression

| | **Classification** | **Regression** |
|---|-------------------|----------------|
| **Target y** | Categories (0/1, spam/inbox, species) | Continuous number (price, minutes, MW) |
| **Output** | Probability or class | Single number |
| **Examples** | Fraud, churn, rain yes/no | Flat price, delivery ETA |

**Rule:** Look at **type of y**, not the app domain.

---

## 🟡 Training loop (what “learning” means)

1. Start with **random weights**
2. **Predict** \(p\) for each row
3. Measure **loss** (how wrong — e.g. log loss for classification)
4. **Gradient descent:** nudge each weight to reduce loss
5. **Repeat** (epochs) until good enough

**Learning rate \(\alpha\):** step size — too big = unstable; too small = slow.

**Key phrase from class:** *Weights are not chosen; they are learned.*

---

## 🟡 Indian bank auto-loan case (preprocessing you must understand)

Features: age, income, education, city → target: buy car (0/1).

| Step | Why |
|------|-----|
| **Ordinal encoding** | Education has order: HS < Bachelor < Masters < PhD |
| **One-hot encoding** | City has **no** order — don’t fake Mumbai=1, Delhi=2 |
| **Scaling (z-score)** | Income in lakhs must not dominate age |

---

## 🔴 Data leakage (industry pitfall #1)

- **Train/test split:** ~80% learn, ~20% final exam (never seen during training)
- **Fit scalers on train only**, then transform test
- Never tune on test repeatedly

**Symptom:** 99% train accuracy, ~52% test → **overfitting** or leakage.

---

## 🟡 ML lifecycle (6 steps from student guide)

1. Load data  
2. Split **X** (features) and **y** (target)  
3. Train/test split  
4. Train  
5. Predict on test  
6. Evaluate metrics  

**Golden rule:** Never evaluate on training data.

---

## 🟡 ML vs deep learning (when to care)

| | Classical ML | Deep learning |
|---|--------------|---------------|
| **Data** | Structured tables | Images, long text, audio |
| **Features** | Often human-designed | Learned in layers |
| **Data size** | Can be smaller | Usually needs lots |

**Start simple:** logistic regression / trees on tabular data before deep nets.

---

## 🟢 Complexity vs explainability (architect trade-off)

1. **Glass-box** — logistic regression, shallow trees (regulated credit)
2. **Ensembles** — XGBoost + SHAP for attribution
3. **Black-box** — deep nets, LLMs for unstructured data

Higher complexity ≠ guaranteed better accuracy.

---

## Checkpoint

1. What does a **negative weight** mean?
2. Why one-hot encode **city** but ordinal-encode **education**?
3. What’s the difference between **loss** and **accuracy**?

**Next:** [04-classification-regression-and-metrics.md](04-classification-regression-and-metrics.md)
