# Session notes — Metrics that don’t lie (and often do)

*Based on: Student Guide (confusion matrix, Iris lab), Week 1 quiz #3.*

---

## 🟢 Confusion matrix — four boxes

For “positive class” (e.g. Dog, Fraud, Rain):

|  | Predicted + | Predicted − |
|--|-------------|-------------|
| **Actually +** | **TP** (right hit) | **FN** (miss) |
| **Actually −** | **FP** (false alarm) | **TN** (right reject) |

---

## 🟢 Accuracy

\[
\text{Accuracy} = \frac{TP + TN}{\text{all samples}}
\]

**When it’s fine:** Balanced classes, errors cost similarly.

**When it lies:** Rare events (fraud 1%, cancer 1%).

---

## 🟡 The accuracy paradox (class quiz)

Model that **always predicts “legit”** on fraud:

- Accuracy ≈ **99%** (looks great)
- Fraud caught = **0%** (business disaster)

**Production ask:** Recall ≥ 85% at precision ≥ 40% (example from slides) + human review band.

---

## 🟢 Precision vs recall (different questions)

| Metric | Question | Base |
|--------|----------|------|
| **Precision** | Of all **positive predictions**, how many were right? | Predictions |
| **Recall** | Of all **actual positives**, how many did we find? | Truth |

Formulas:

- Precision = TP / (TP + FP)
- Recall = TP / (TP + FN)

**When precision matters:** False alarm is expensive (legit email marked spam).

**When recall matters:** Missing a positive is deadly (tumor, fraud).

**F1:** balances precision and recall (harmonic mean).

---

## 🟡 Iris workflow (sklearn mental model)

1. `load_iris()` → X = measurements, y = species (0,1,2)
2. `train_test_split(..., test_size=0.2, stratify=y, random_state=42)`
3. `model.fit(X_train, y_train)`
4. `model.predict(X_test)`
5. `confusion_matrix`, `classification_report`

**Stratify:** keep same class mix in train and test.

---

## 🟡 Overfitting vs underfitting

| | Underfitting | Overfitting |
|---|--------------|-------------|
| **Symptom** | Poor on train and test | Great train, poor test |
| **Cause** | Too simple | Memorized noise |
| **Fix ideas** | Richer features / model | More data, regularization, simpler model |

---

## 🔴 Cross-validation

One split can be lucky. **cross_val_score** with k folds → **mean CV score** = more stable estimate.

---

## 🟡 Imbalanced data toolkit

- Don’t trust accuracy alone
- Use precision-recall, class weights, resampling
- Inspect FP and FN manually (**error analysis**)

---

## Quiz #3 answers (from slides)

| Dilemma | Answer |
|---------|--------|
| RBI must explain rejection factors | **Glass-box** + SHAP |
| 99% accuracy, 1% tumors | **Accuracy trap** → demand **recall** |
| Predict IPL coin toss | **Infeasible** (random) |
| 99% train, 52% test | **Overfitting** (or leakage) |

---

## Checkpoint

If precision is 100% and recall is 10%, what’s happening?

**Next:** [05-python-ml-workflow-and-pitfalls.md](05-python-ml-workflow-and-pitfalls.md)
