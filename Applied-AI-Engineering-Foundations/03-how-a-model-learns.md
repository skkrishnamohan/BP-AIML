# Session notes — How a model learns (neuron → logistic regression → neural nets)

*Based on: Student Guide, Week 1 slides (bank auto-loan), insurance coding material.*

This file is the mechanics underneath “the model learned.” The story is always the same: turn the world into numbers, score a guess, measure how wrong it is, and nudge the numbers so the next guess is less wrong.

---

## 🟢 Everything is numbers

Neural networks don’t read English or pixels directly. Data becomes **numbers** (vectors). Learning = adjusting **weights** so predictions get closer to true **labels**.

**What a vector is:** An ordered list of numbers, such as `[age, income, education_code]`. **Vector** is the math name for that list. A table row is one vector. A batch of rows is a matrix.

**Why numbers:** Multiplication and addition are how a neuron combines evidence. Text and categories must be encoded first. If the encoding invents a fake order, the model will treat that order as real.

**How:** Map every input to a number, compute a score, compare the score with `y`, update weights.

**Where:** Every classical ML lab in this module, and the insurance gradient-descent exercise.

---

## 🟢 One neuron (the “bouncer” analogy)

Steps:

1. **Inputs** \(x_1, x_2, ...\) — e.g. age, income, education
2. **Weights** \(w_i\) — importance dials (positive = pushes toward “yes”)
3. **Bias** \(b\) — starting mood
4. **Weighted sum:** \(z = w_1 x_1 + w_2 x_2 + ... + b\)
5. **Activation** — squash \(z\) to a probability (sigmoid for binary yes/no)

**One neuron + sigmoid ≈ logistic regression** (linear boundary in feature space).

### What each symbol means

| Symbol | Name | Definition | Everyday meaning |
|--------|------|------------|------------------|
| \(x_i\) | Feature | One input number | A fact about this person or this day |
| \(w_i\) | Weight | A learned multiplier | How hard that fact pushes the decision |
| \(b\) | Bias term | A learned constant | The starting score before any fact is considered |
| \(z\) | Logit | The weighted sum | The raw score, still not a probability |
| \(\sigma(z)\) | Sigmoid | \(1 / (1 + e^{-z})\) | Squashes any score into a number between 0 and 1 |

**Why a weight can be negative:** A negative weight pushes toward class 0. If “number of recent defaults” has a negative weight for “will repay,” more defaults lower the approval score. Negative does not mean the feature is unimportant. It means the feature votes the other way.

**Why bias exists:** Even with all features at zero, the base rate may not be 50%. If 90% of historical customers did not buy, the bias can start the score toward “no” before age and income speak.

**Linear boundary:** Logistic regression draws a straight cut through the feature space. Points on one side lean yes, points on the other lean no. If the true pattern is a curve or a combination such as “young and high income, or older and low debt,” one straight cut is not enough. That is when trees or extra layers help.

**The bouncer analogy, mapped:** Each fact is a person at the door. The weight is how much that person influences the bouncer. The bias is the bouncer’s mood at the start of the night. The sigmoid is the bouncer turning a messy argument into “how sure am I, from 0 to 100%?”

---

## 🟢 Sigmoid and the 0.5 rule

Sigmoid maps any \(z\) to **(0, 1)** = probability.

- If \(p \geq 0.5\) → predict class **1** (e.g. “will buy car”)
- If \(p < 0.5\) → predict class **0**

**Business insight:** Banks use **tiers** (e.g. 88% → pre-approved loan), not only 0/1.

**What 0.5 is and is not:** It is the point where the model thinks yes and no are equally likely, and it is the usual default cutoff. It is not a law. If missing a fraud is costly, you lower the cutoff so more cases are flagged. If a false alarm is costly, you raise it.

**Why the output is a probability and not a certainty:** The sigmoid never quite reaches 0 or 1. An output of 0.88 means “among people who looked like this in the training data, about 88% had the positive label,” and only if the model is well calibrated. **Calibration** means a predicted 0.88 really happens about 88% of the time. A sharp model can still be badly calibrated.

**Where tiers show up:** A bank might auto-approve above 0.75, send 0.40–0.75 to a human, and decline below 0.40. The model emits one number. The policy around that number is still rules.

---

## 🟡 Classification vs regression

| | **Classification** | **Regression** |
|---|-------------------|----------------|
| **Target y** | Categories (0/1, spam/inbox, species) | Continuous number (price, minutes, MW) |
| **Output** | Probability or class | Single number |
| **Examples** | Fraud, churn, rain yes/no | Flat price, delivery ETA |

**Rule:** Look at **type of y**, not the app domain.

**What classification is:** Learn a boundary between named buckets. **Binary** means two buckets (churn or stay). **Multi-class** means three or more (clear, cloudy, rain). The model still emits numbers; you turn them into a bucket with `argmax` or a threshold.

**What regression is:** Learn a number on a scale. Price in rupees, minutes until delivery, megawatts of load. There is no “positive class.”

**Why the domain is a trap:** “Will it rain?” is classification even though the setting is weather. “How many millimetres will fall?” is regression in the same setting. Read the target column, not the product name.

**ETA** means estimated time of arrival. Minutes are continuous, so delivery ETA is regression.

**Churn** means the customer leaves. Yes/no churn is binary classification.

---

## 🟡 Training loop (what “learning” means)

1. Start with **random weights**
2. **Predict** \(p\) for each row
3. Measure **loss** (how wrong — e.g. log loss for classification)
4. **Gradient descent:** nudge each weight to reduce loss
5. **Repeat** (epochs) until good enough

**Learning rate \(\alpha\):** step size — too big = unstable; too small = slow.

**Key phrase from class:** *Weights are not chosen; they are learned.*

### The words inside the loop

| Term | Definition | What it feels like when it is wrong |
|------|------------|--------------------------------------|
| **Loss** | A single number that is large when predictions disagree with labels | You optimize this. You do not show it to the business as the success metric. |
| **Log loss** | \(-(y \log p + (1-y) \log (1-p))\) | Confident and wrong is punished hard. Saying 0.99 when the truth is 0 hurts more than saying 0.60. |
| **Gradient** | The direction each weight should move to reduce loss | The slope of the hill. |
| **Gradient descent** | Repeatedly step opposite the gradient | Walk downhill. |
| **Learning rate** | How large each step is | Too large and you jump over the valley. Too small and training crawls. |
| **Epoch** | One pass through the training set | More epochs are not automatically better; extra passes can memorize noise. |
| **Batch** | A subset of rows used for one update | A compromise between a noisy single-row update and a slow full-dataset update. |

**Why start random:** If every model started from the same hand-picked weights, you would be writing rules again. Random start plus a loss function lets the data move the weights. The seed makes that randomness repeatable.

**Loss vs accuracy:** Loss is the training signal and uses probabilities. Accuracy is a report card after you threshold those probabilities. Two models can share an accuracy and have very different loss if one is calmly right and the other is luckily right. Training almost always minimizes loss, not accuracy directly, because accuracy is flat (a tiny weight change may not flip any class) and loss is smooth.

**SGD** means **stochastic gradient descent**: the gradient is estimated from a batch rather than from every row. It is the practical form of the idea above.

---

## 🟡 Indian bank auto-loan case (preprocessing you must understand)

Features: age, income, education, city → target: buy car (0/1).

| Step | Why |
|------|-----|
| **Ordinal encoding** | Education has order: HS < Bachelor < Masters < PhD |
| **One-hot encoding** | City has **no** order — don’t fake Mumbai=1, Delhi=2 |
| **Scaling (z-score)** | Income in lakhs must not dominate age |

### What the encodings are doing

**Ordinal encoding:** Replace an ordered category with integers that respect order, such as High School = 1, Bachelor = 2, Master = 3, PhD = 4. **Ordinal** means the order is meaningful. The model may treat the gaps as equal, which is only an approximation, but it is an honest approximation because more education really is “more.”

**One-hot encoding:** Make one column per city, with 1 in that city’s column and 0 elsewhere. Mumbai is not “more city” than Delhi. If you stored Mumbai as 1 and Delhi as 2, the model would think Delhi is twice Mumbai. **Nominal** data has names and no order. Cities, colors, and product IDs are nominal.

**Dummy-variable caution:** If you keep every city column, they add up to 1 and duplicate the bias term. Dropping one city (the reference city) avoids that trap. Sklearn’s encoder can do this for you.

**Z-score scaling:** Subtract the training mean and divide by the training standard deviation. A value of 0 means “average.” A value of 2 means “two standard deviations above average.” Income in rupees (hundreds of thousands) would otherwise swamp age (tens) because gradient steps follow the raw magnitude.

**Where this bites:** Fit the mean and standard deviation on the training rows only. If the test rows help compute the mean, the exam has leaked into the study sheet.

**HS** in the table means high school, the bottom of that education order.

---

## 🔴 Data leakage (industry pitfall #1)

- **Train/test split:** ~80% learn, ~20% final exam (never seen during training)
- **Fit scalers on train only**, then transform test
- Never tune on test repeatedly

**Symptom:** 99% train accuracy, ~52% test → **overfitting** or leakage.

**What leakage is:** Any path by which information that would be unavailable at prediction time influences training. It is not always a dramatic column named `answer`. It is often a scaler, a feature chosen because it worked on the test set, or a target built with tomorrow’s measurement.

**Why the 80/20 habit exists:** The test set estimates future performance. The moment you change the model because of the test score, that estimate becomes optimistic. Use a **validation** slice, or cross-validation, to make those choices. Touch the test set once at the end.

**How overfitting looks different from leakage:** Overfitting memorizes training noise and still had a fair test. Leakage contaminates the test, so the test score itself is a lie and can look excellent until production. The 99% train / 52% test pattern is the honest symptom of “learned the training sheet, failed the exam.” A leaked test can hide even that warning.

**Where to look first:** Was the imputer, scaler, or encoder fit on the full frame before `train_test_split`? Was a future aggregate merged onto past rows? Was the split random on a time series?

---

## 🟡 ML lifecycle (6 steps from student guide)

1. Load data
2. Split **X** (features) and **y** (target)
3. Train/test split
4. Train
5. Predict on test
6. Evaluate metrics

**Golden rule:** Never evaluate on training data.

**Why step 2 is its own step:** If the target column stays inside `X`, the model learns to copy it. That is the most literal form of leakage, and accuracy becomes perfect for a useless reason.

**Why training metrics are not the result:** The model has already seen those answers. A student who grades their own open-book practice is not sitting the exam. Report test metrics, and say which split produced them.

---

## 🟡 ML vs deep learning (when to care)

| | Classical ML | Deep learning |
|---|--------------|---------------|
| **Data** | Structured tables | Images, long text, audio |
| **Features** | Often human-designed | Learned in layers |
| **Data size** | Can be smaller | Usually needs lots |

**Start simple:** logistic regression / trees on tabular data before deep nets.

**What a layer adds:** Each layer transforms the previous vector into a new vector. Early layers in vision pick up edges. Later layers combine edges into parts. You did not name those parts; the loss function pressured the weights until the parts became useful.

**Why “start simple” is an engineering rule, not a lack of ambition:** A logistic regression trains in seconds, explains coefficients, and sets the baseline a deep model must beat. On the bank table, a deep net can overfit long before it beats a regularized linear model.

---

## 🟢 Complexity vs explainability (architect trade-off)

1. **Glass-box** — logistic regression, shallow trees (regulated credit)
2. **Ensembles** — XGBoost + SHAP for attribution
3. **Black-box** — deep nets, LLMs for unstructured data

Higher complexity ≠ guaranteed better accuracy.

| Term | Meaning |
|------|---------|
| **Glass-box** | You can read the decision in coefficients or a short tree. |
| **Ensemble** | Many models vote. **XGBoost** is a popular boosted-tree ensemble: each new tree tries to fix the errors of the previous ones. |
| **SHAP** | SHapley Additive exPlanations. A method, from game theory, that assigns each feature a share of the blame or credit for one prediction. |
| **Black-box** | Accurate enough to use, opaque enough that you need a separate explanation tool. |
| **Regularization** | A penalty that keeps weights smaller so the model is less tempted to memorize. |

**Where regulation pushes you left:** If a lender must tell a customer which factors drove a rejection, a glass-box model or a documented explanation method is part of the requirement, not a nice-to-have. The RBI-style quiz in the next file makes this explicit.

---

## Checkpoint

1. What does a **negative weight** mean?
2. Why one-hot encode **city** but ordinal-encode **education**?
3. What’s the difference between **loss** and **accuracy**?

**Answers to check against:**

1. The feature pushes the score toward the negative class. Larger values of that feature lower the probability of class 1.
2. Education has a real order. City names do not, so integers would invent a fake ranking.
3. Loss is the smooth training objective, sensitive to confidence. Accuracy is the share of labels correct after a threshold, and it is what people report, not what gradient descent usually optimizes.

**Next:** [04-classification-regression-and-metrics.md](04-classification-regression-and-metrics.md)
