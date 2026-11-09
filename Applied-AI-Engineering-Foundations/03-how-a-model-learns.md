# Class notes — How a model learns

*Student guide, the bank auto-loan slides, and the insurance coding material.*

The last class said “the model learns from data.” This class opens the box. There is no magic in the box. There is a weighted sum, a way to squash it into a probability, a number that says how wrong the probability was, and a small step that makes it less wrong. If you can do that by hand for one customer, the libraries in the next classes are just the same story with more rows.

---

## 🟢 Everything becomes a list of numbers

A network does not read the word “Mumbai” and it does not see a photograph as a photograph. It sees a **vector**, which is an ordered list of numbers. One customer might be `[34, 12.5, 2]`, meaning age 34, income 12.5 lakh, education coded as 2. A table row is one vector. A batch of rows is a matrix, a grid of those lists.

Learning means moving the **weights** so that the vector goes in and a prediction comes out closer to the **label**. If you cannot turn the world into an honest list of numbers, you are not ready to train. The rest of this class is what happens after you can.

---

## 🟢 One neuron, with the arithmetic on the board

We called the neuron a bouncer. Each fact is a voice at the door. The weight is how much that voice counts. The bias is the bouncer’s mood before anyone speaks. The calculation is not poetic. It is a sum of products.

Copy the five lines.

1. Inputs \(x_1, x_2, \ldots\). Age, income, education.
2. Weights \(w_1, w_2, \ldots\). Learned dials. Positive pushes toward “yes.” Negative pushes toward “no.”
3. Bias \(b\). The starting score when every input is zero.
4. The weighted sum, called the **logit**: \(z = w_1 x_1 + w_2 x_2 + \cdots + b\).
5. An **activation** that squashes \(z\). For a yes/no decision we use the **sigmoid**, and the result is treated as a probability.

One neuron plus a sigmoid is **logistic regression**. Geometrically it cuts the space of features with a straight boundary. On one side the probability leans toward yes. On the other it leans toward no. If the true pattern bends (“young and high income, or older with very low debt”), one straight cut is the wrong shape. That is when trees or extra layers earn their place. It is not the place we start.

### A customer, by hand

Suppose the frozen weights for “will buy a car” are:

- age weight \(w_{\text{age}} = 0.03\)
- income weight \(w_{\text{income}} = 0.20\), and income is in lakhs
- bias \(b = -2.0\)

A customer is 40 years old with income 15 lakh. Education is left out so the arithmetic fits on one line.

\[
z = (0.03)(40) + (0.20)(15) + (-2.0) = 1.2 + 3.0 - 2.0 = 2.2
\]

The sigmoid is

\[
p = \frac{1}{1 + e^{-z}} = \frac{1}{1 + e^{-2.2}} \approx \frac{1}{1 + 0.111} \approx 0.90
\]

About 90%. The class rule of 0.5 would say “predict buy.” A bank may not use 0.5. That comes next.

Now change only the income weight to negative, \(w_{\text{income}} = -0.20\), which would be a strange car-loan model but a normal “will default” model.

\[
z = 1.2 + (-0.20)(15) - 2.0 = 1.2 - 3.0 - 2.0 = -3.8
\]

\[
p \approx \frac{1}{1 + e^{3.8}} \approx 0.02
\]

A **negative weight** does not mean “this feature does not matter.” It means the feature votes for class 0. More income, in this second model, pushes hard away from the positive class. Students who say “negative means unimportant” are reading the sign as a magnitude. The magnitude is how loud the vote is. The sign is which way it votes.

Why have a bias at all? If 90% of historical customers did not buy, the starting score should lean toward no before age and income speak. The bias is that lean. It is learned, not typed because someone felt pessimistic.

---

## 🟢 Sigmoid, 0.5, and why a bank does not live at 0.5

The sigmoid maps any real number \(z\) to a number strictly between 0 and 1. Very negative \(z\) sits near 0. Very positive \(z\) sits near 1. Zero sits at 0.5. It never quite touches 0 or 1, which is a useful humility: the model does not claim impossibility or certainty.

The textbook decision is:

- if \(p \ge 0.5\), predict class 1
- if \(p < 0.5\), predict class 0

**0.5 is the point where the model thinks the two classes are equally likely. It is not a business rule.** If missing a fraud is expensive, you lower the cutoff so more cases are flagged. If a false alarm is expensive, you raise it. The neuron emits one number. The policy around the number is still written by people.

The bank story on the slide uses tiers. A probability of 88% might be a pre-approved offer. A band underneath might go to a human. Below a floor, decline. Three actions from one number. Do not throw the number away and keep only the yes/no unless the action truly is yes/no.

**Calibration** is the extra question students skip. A calibrated 0.88 means that, among people the model scores near 0.88, about 88% really do buy. A model can rank people correctly and still be badly calibrated, the way a student can get the order of difficulty right and still be overconfident. For a threshold of 0.75 to mean what the bank thinks it means, calibration matters.

---

## 🟡 Classification and regression are about y, not about the industry

| | Classification | Regression |
|--|----------------|------------|
| What y is | A category | A quantity |
| What comes out | A class, or a probability you then cut | A number in the unit of y |
| Examples from class | Fraud, churn, rain or not, iris species | Flat price in rupees, delivery minutes, grid load in MW |

**Binary** means two categories. **Multi-class** means three or more: clear, cloudy, rain. The model still emits numbers. You turn them into a name by taking the largest, or by applying a threshold.

**Regression** has no positive class. Rupees, minutes, and megawatts are distances on a scale.

The trap is the domain. “Will it rain?” is classification in a weather company. “How many millimetres?” is regression in the same company. Read the target column.

**Churn** means the customer leaves. Yes or no is binary classification. **ETA** is estimated time of arrival. Minutes are regression.

---

## 🟡 The training loop, and a tiny weight update

Copy the loop.

1. Start the weights at random. If you hand-picked them, you would be writing rules again.
2. For each row, compute \(p\).
3. Measure the **loss**, one number that grows when \(p\) disagrees with \(y\).
4. Compute the **gradient**, the direction to move each weight to reduce the loss.
5. Step a small distance in that direction. The distance is the **learning rate**, written \(\alpha\).
6. Repeat. One pass through the training rows is an **epoch**.

The sentence from class: **weights are not chosen. They are learned.**

### What log loss is punishing

For binary labels, **log loss** (also called cross-entropy) for one row is

\[
-(y \log p + (1-y)\log(1-p))
\]

If the truth \(y\) is 1, this collapses to \(-\log p\). If you said 0.9, the loss is \(-\log 0.9 \approx 0.105\). If you said 0.1 while the truth was 1, the loss is \(-\log 0.1 \approx 2.30\). Confident and wrong is a different offence from cautiously wrong. That is why we train on log loss and not on “right or wrong.”

### One step of gradient descent, so the phrase is not magic

Suppose a single weight, loss \(L = (w - 3)^2\), which is smallest at \(w = 3\). The gradient is \(2(w - 3)\). Start at \(w = 0\), and take \(\alpha = 0.1\).

\[
w \leftarrow w - \alpha \cdot 2(w - 3)
\]

First step: \(0 - 0.1 \cdot 2(0 - 3) = 0 - 0.1 \cdot (-6) = 0.6\).

Second step starts at 0.6 and moves toward 3 again. We are walking downhill. The real logistic-regression gradient is messier and uses every row’s error, but the step is the same idea: subtract a fraction of the slope.

If \(\alpha\) is enormous, you jump over the valley and the loss explodes. If \(\alpha\) is tiny, you will get there next year. The insurance lab’s usual bug, when the handmade loop diverges, is a learning rate that is too large for features that were not scaled.

**SGD** means stochastic gradient descent. Stochastic means the slope is estimated from a **batch**, a subset of rows, rather than from the entire dataset every time. It is noisier and much cheaper. It is the practical form of the walk above.

**Loss is not accuracy.** Loss uses the probability and is smooth, so a tiny weight change changes it. Accuracy flips only when a probability crosses the threshold, so it is a bad thing to walk downhill on. You optimize loss. You report accuracy, or something better, after you have chosen a threshold. Two models can share an accuracy and have very different loss if one is calmly right and the other is luckily right.

A **parameter** is a number the loop learns, such as \(w\) and \(b\). A **hyperparameter** is a choice you make before the loop, such as \(\alpha\), the number of epochs, or `max_iter`. You do not learn the learning rate by pretending it is a weight, not in this class.

---

## 🟡 The auto-loan table, and three encodings

Features: age, income, education, city. Target: bought a car, 0 or 1.

Three preparations, each repairing a different lie.

**Ordinal encoding for education.** The levels have a real order: high school, then bachelor’s, then master’s, then PhD. Replacing them with 1, 2, 3, 4 lets the model hear “more.” **Ordinal** means the order is meaningful. The gaps are only approximately equal. A PhD is not exactly one unit more than a master’s. It is still an honest approximation, and it is the right family of encoding.

**One-hot encoding for city.** Mumbai is not more city than Delhi. If you store Mumbai as 1 and Delhi as 2, the weighted sum treats Delhi as twice Mumbai. **Nominal** data has names and no order. Cities, colors, and product codes are nominal. One-hot encoding makes one column per city, with a 1 in that city’s column and 0 elsewhere. Then a weight can say “Mumbai pushes up” without saying “Delhi is twice Mumbai.”

The **dummy-variable trap**: if every city column is kept, those columns sum to 1 on every row. That sum is a perfect copy of a column of ones, which is the bias. The optimizer cannot tell the bias and the cities apart, and the weights swing wildly. Drop one city. The dropped city is the reference. All zeros in the remaining city columns means “the reference city.”

**Z-score scaling.** Income in rupees is hundreds of thousands. Age is tens. The gradient follows the raw size, so income shouts and age whispers. A z-score subtracts the training mean and divides by the training standard deviation. Zero means average. Two means two standard deviations above average. After that, the weights are comparable as “how hard this clue pushes,” which is what we wanted at the start of class.

The mean and the standard deviation come from the training rows only. If the test rows help compute them, the exam helped you study. That is leakage, and it is the next section.

---

## 🔴 Leakage, overfitting, and the 99 / 52 symptom

The **train/test split** in this class is about 80% to learn and 20% as a final exam. The exam rows are not used to move weights, to choose features, or to fit a scaler.

**Overfitting** means the model memorized quirks of the training sample, including noise. Training accuracy is flattering. Test accuracy drops. The symptom on the slide is about 99% on train and about 52% on test, which is coin-toss territory for a balanced yes/no. The model learned the practice sheet and failed the paper.

**Underfitting** is the opposite illness. The model is too rigid to express the pattern, so it is poor on the training rows themselves. More data does not fix a straight line drawn through a U-shape. Adding a better feature or a more flexible model does.

Leakage can hide even the 99/52 warning. If the test set helped you train, the test score itself is a lie and may look excellent until production, where the leaked clue does not exist. How leakage actually happens:

- The target column was left inside X. The model copies it. Accuracy becomes perfect and meaningless.
- The scaler, the imputer, or the encoder was fit on the full file before the split.
- You tuned the model twenty times on the test score. The test set has become training data with extra steps.
- A feature uses the future. Tomorrow’s rain, or a daily average that includes hours after the decision.

A fair test can still show 99/52. That is overfitting or a genuinely unpredictable label. A leaked test can show 99/99 and then fail on Monday. When a score looks too good for the problem, assume leakage until you have pointed at the split and the feature list.

**Regularization** is a penalty on large weights, so the model is less tempted to memorize. It is one of the fixes for overfitting, along with more data and a simpler model. It does not fix leakage. You cannot regularize your way out of having peeked.

---

## 🟡 The six-step lifecycle from the student guide

1. Load the table.
2. Separate X from y. Look at the columns of X and make sure the answer is not sitting in there.
3. Split train and test.
4. Fit on train only.
5. Predict on test. The weights are frozen. y_test is not an input to predict.
6. Compute metrics against y_test.

The golden rule on the slide: never evaluate on the training rows and call it the result. Those rows have already been seen. A student grading their own open-book practice is not sitting the exam.

---

## 🟡 When the one neuron is no longer enough

| | Classical ML | Deep learning |
|--|--------------|---------------|
| Typical data | Tables | Images, long text, audio |
| Who builds the clues | A person, as features | The layers |
| How much data | A careful table can be enough | Usually a lot, because the model is flexible enough to memorize |

A **layer** transforms one vector into another. In vision, early layers respond to edges and later layers respond to parts. You did not name the parts. The loss pressured the weights until those parts became useful. That flexibility is exactly why a deep net on the bank table can overfit long before it beats a regularized logistic regression. Start simple. Make the simple model the thing the fancy model has to beat.

The tradeoff the architect draws:

1. **Glass-box.** Logistic regression, or a short tree. You can read the decision. Regulated credit wants this, because a rejected customer can be told which factors mattered.
2. **Ensembles with an explanation tool.** **XGBoost** is a boosted-tree ensemble: each new tree tries to correct the errors of the ones before it. **SHAP** (SHapley Additive exPlanations) assigns each feature a share of the credit for one prediction. You cannot read a thousand trees, so you attach an explanation.
3. **Black-box.** Deep nets and LLMs for data that has no columns. Use them when the input is unstructured, and budget for the fact that you will need a separate way to debug them.

Higher complexity is not a promise of higher accuracy. It is a promise of a harder audit.

---

## Checkpoint

1. In the hand-worked customer, what changed when the income weight flipped from +0.20 to −0.20? Answer in terms of the vote, not the word “negative.”
2. Why is city one-hot encoded while education may be 1, 2, 3, 4?
3. A model is right on the class but was 99% sure when the truth was the other class. Which number feels that pain, the loss or the accuracy? Why do we train on that one?

After you try:

1. Income stopped pushing toward “buy” and started pushing away from it. The probability fell from about 0.90 to about 0.02. The feature got louder in the other direction. It did not become irrelevant.
2. Education has a real order. City names do not. Integers would invent a ranking the cities do not have.
3. Log loss. Accuracy only notices that the class was wrong, and it may not notice at all until a threshold is crossed. Log loss is large precisely because the model was confident and wrong, and it is smooth enough to train on.

**Next class:** [04-classification-regression-and-metrics.md](04-classification-regression-and-metrics.md)
