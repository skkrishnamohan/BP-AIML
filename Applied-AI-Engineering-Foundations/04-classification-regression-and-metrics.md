# Session notes — Metrics that don’t lie (and often do)

*Based on: Student Guide (confusion matrix, Iris lab), Week 1 quiz #3.*

A metric is a number that answers one question. The common failure in this module is using a number that answers a different question from the one the business asked.

---

## 🟢 Confusion matrix — four boxes

For “positive class” (e.g. Dog, Fraud, Rain):

|  | Predicted + | Predicted − |
|--|-------------|-------------|
| **Actually +** | **TP** (right hit) | **FN** (miss) |
| **Actually −** | **FP** (false alarm) | **TN** (right reject) |

### Definitions, not just letters

| Cell | Expansion | Definition | Meaning |
|------|-----------|------------|---------|
| **TP** | True positive | Predicted positive and it was positive | You caught a real fraud, a real tumor, a real rainy day |
| **FN** | False negative | Predicted negative and it was positive | You missed it. The costly miss in safety and fraud |
| **FP** | False positive | Predicted positive and it was negative | A false alarm. A legitimate mail marked spam |
| **TN** | True negative | Predicted negative and it was negative | Correctly left alone |

**What “positive” means:** It is the class you chose to care about, not a moral good. You must declare it. In fraud, positive usually means “fraud.” If someone flips the coding so 1 means “legit,” every formula still runs and every conclusion reverses.

**Why four boxes instead of one accuracy:** Accuracy adds TP and TN and forgets which kind of mistake you made. The matrix keeps the mistakes separate so precision and recall can ask different questions.

**How to read a matrix:** Rows are reality. Columns are the model’s claim. Practice by covering one cell and explaining who was harmed.

**Where:** Iris species lab, fraud quizzes, and any `confusion_matrix(y_test, y_pred)` printout.

---

## 🟢 Accuracy

\[
\text{Accuracy} = \frac{TP + TN}{\text{all samples}}
\]

**When it’s fine:** Balanced classes, errors cost similarly.

**When it lies:** Rare events (fraud 1%, cancer 1%).

**What it literally is:** The fraction of rows where the predicted class matches the label.

**Why it lies on rare events:** If 1% of transactions are fraud, a model that always says “legit” scores 99%. The denominator is full of easy true negatives. The business wanted the rare positives.

**Balanced classes** means each class is roughly as common as the others. Iris species in the teaching dataset are balanced, so accuracy is less misleading there than it is in fraud.

**Class imbalance** means one class is rare. The rare class is usually the one you cannot afford to ignore, which is exactly when accuracy looks best while doing nothing.

---

## 🟡 The accuracy paradox (class quiz)

Model that **always predicts “legit”** on fraud:

- Accuracy ≈ **99%** (looks great)
- Fraud caught = **0%** (business disaster)

**Production ask:** Recall ≥ 85% at precision ≥ 40% (example from slides) + human review band.

**What the production sentence is demanding:** Catch at least 85% of real fraud, and when the model raises an alarm, be right at least 40% of the time. The leftover alarms go to a person. That is an operating point, not “maximize accuracy.”

**Why both numbers appear together:** You can get 100% recall by flagging everyone, and precision collapses. You can get high precision by flagging only the most obvious fraud, and recall collapses. The slide’s pair is a business compromise.

**Human review band:** Scores in a middle range are not auto-decided. The model triages; a person handles the uncertain band. That is a system design, not a metric.

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

### How to remember which denominator

**Precision** starts from what you predicted. The denominator is every positive alarm (TP + FP). A spam filter with low precision annoys people because many alarms are wrong.

**Recall** starts from what was real. The denominator is every actual positive (TP + FN). A tumor screen with low recall sends sick people home. Recall is also called **sensitivity** or **true positive rate (TPR)**.

**Specificity** is the twin of recall for the negative class: TN / (TN + FP). It answers “of the truly healthy or truly legitimate cases, how many did we correctly pass?”

**F1** is the harmonic mean:

\[
F1 = 2 \cdot \frac{\text{precision} \cdot \text{recall}}{\text{precision} + \text{recall}}
\]

**Why harmonic, not a plain average:** If precision is 1.0 and recall is 0.0, a plain average says 0.5, which sounds acceptable. The harmonic mean is 0, which matches reality: the model found nothing. F1 is a summary, not a substitute for knowing which error the business hates.

**Where to choose:** Write the cost of one FP and one FN in rupees or in harm before you pick the metric. Then move the threshold until the costs are acceptable. Do not hunt for a universal “best” metric.

---

## 🟡 Iris workflow (sklearn mental model)

1. `load_iris()` → X = measurements, y = species (0,1,2)
2. `train_test_split(..., test_size=0.2, stratify=y, random_state=42)`
3. `model.fit(X_train, y_train)`
4. `model.predict(X_test)`
5. `confusion_matrix`, `classification_report`

**Stratify:** keep same class mix in train and test.

**What the Iris data is:** A small classic table of flowers. Four measurements (sepal and petal length and width) and three species. It is multi-class classification. It is taught because the table is clean, not because flowers are the goal.

**What each call does:**

| Call | What | Why |
|------|------|-----|
| `train_test_split` | Holds out 20% of rows | The held-out rows are the exam |
| `test_size=0.2` | 20% test, 80% train | A common teaching split, not a law |
| `stratify=y` | Preserves the species mix | A random split of a small set can accidentally put almost all of one species in test |
| `random_state=42` | Fixes the shuffle | 42 is a convention. Any fixed integer makes the split repeatable. |
| `fit` | Learns from training rows only | This is the only place labels may influence weights |
| `predict` | Emits a class per test row | Uses frozen weights, no labels |
| `classification_report` | Precision, recall, and F1 per class | One accuracy would hide a species the model confuses |

**Where students go wrong:** Calling `predict` on `X_train` and celebrating that score. Also fitting a scaler on `X` before the split.

---

## 🟡 Overfitting vs underfitting

| | Underfitting | Overfitting |
|---|--------------|-------------|
| **Symptom** | Poor on train and test | Great train, poor test |
| **Cause** | Too simple | Memorized noise |
| **Fix ideas** | Richer features / model | More data, regularization, simpler model |

**What underfitting means:** The model’s family cannot express the pattern. A straight line through a U-shaped relationship fails on the training rows themselves. Adding data does not fix a model that is the wrong shape. This is also called **high bias** in the statistical sense: the model is systematically too rigid. That “bias” is not the neuron’s bias term and not social bias.

**What overfitting means:** The model fits quirks of this sample, including noise. Training accuracy is flattering. Test accuracy drops. This is **high variance**: a different sample would have produced a very different model.

**Bias–variance tradeoff:** Simple models have more bias and less variance. Flexible models have less bias and more variance. Regularization, more data, and a smaller tree pull a flexible model back.

**How you tell them apart:** Compare train and test. Both bad → underfit or a data problem. Train excellent and test poor → overfit or a leakage-free distribution gap. Both excellent, then production poor → leakage or drift. The table alone cannot diagnose leakage.

---

## 🔴 Cross-validation

One split can be lucky. **cross_val_score** with k folds → **mean CV score** = more stable estimate.

**What a fold is:** Split the training data into k parts. Train on k−1 parts, score the leftover part, and rotate until every part has been the leftover once.

**Why:** A single 80/20 cut might put the easy rows in test. The mean across folds is harder to luck into. The spread across folds tells you whether the score is stable.

**How it must be built:** The scaler and the model belong inside the same pipeline, refit on each training fold. If you scale the whole dataset first, every fold has seen the held-out fold’s mean. That is leakage with extra steps.

**Where it is not enough:** Time series. Random folds let the future train the past. Use a forward-rolling split instead. Module 2 makes chronological splits mandatory for rain.

**CV** here means **cross-validation**, not computer vision. The same letters are used for both; read the sentence.

**k** is the number of folds. 5 is a common default.

---

## 🟡 Imbalanced data toolkit

- Don’t trust accuracy alone
- Use precision-recall, class weights, resampling
- Inspect FP and FN manually (**error analysis**)

**Class weight:** Tell the loss function that a rare positive counts more than a common negative, so the model cannot ignore it.

**Resampling:** Either duplicate or synthesize rare rows (**oversampling**) or drop some majority rows (**undersampling**). Do this only inside the training fold. A synthetic copy that lands in both train and test is leakage.

**Error analysis:** Read actual false positives and false negatives. The metric says how many. The rows say why. Maybe every miss is a new merchant category, which is a data problem, not a reason to change algorithms.

**PR curve:** Precision–recall curve. It shows the tradeoff as you slide the threshold. **ROC** means receiver operating characteristic, a plot of true positive rate against false positive rate. **AUROC** (area under the ROC) is one summary of ranking quality. On rare events, a precision–recall curve is often the more honest picture. Module 2 uses AUROC as a WeatherWise target.

---

## Quiz #3 answers (from slides)

| Dilemma | Answer |
|---------|--------|
| RBI must explain rejection factors | **Glass-box** + SHAP |
| 99% accuracy, 1% tumors | **Accuracy trap** → demand **recall** |
| Predict IPL coin toss | **Infeasible** (random) |
| 99% train, 52% test | **Overfitting** (or leakage) |

**Why each answer is the answer:**

- A regulator asking for rejection factors needs a model you can explain. Glass-box coefficients, or SHAP on a more complex model, are the course’s pair. Accuracy is irrelevant to that question.
- A 1% tumor rate makes “always healthy” look 99% accurate. The metric that counts missed tumors is recall.
- A fair coin has no features. Gate 2 from the SDLC notes says do not build.
- A huge train-to-test gap means the model memorized, or the test was not as clean as you think. Inspect leakage and simplicity before you celebrate the training number.

**IPL** is the Indian Premier League. The coin toss is a stand-in for any label that is not predictable from the features.

---

## Checkpoint

If precision is 100% and recall is 10%, what’s happening?

**Answer:** Every alarm is correct, and the model only alarms on a tenth of the real positives. It is extremely cautious. That is acceptable when a false alarm is intolerable and missing most cases is acceptable. It is a failure when the job is to find tumors or fraud. Read the business cost before you call the number “good.”

**Next:** [05-python-ml-workflow-and-pitfalls.md](05-python-ml-workflow-and-pitfalls.md)
