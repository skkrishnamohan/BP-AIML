# Class notes — Metrics, and the numbers that flatter you

*Student guide, the confusion-matrix and Iris material, and Week 1 quiz 3.*

A metric is a number that answers one question. The failure this class is about is celebrating a number that answered a different question from the one the business asked. We are going to build the number by hand, on a hundred payments, so that “accuracy is misleading” is an arithmetic fact and not a slogan.

---

## 🟢 Four cells, then the words

Pick the **positive class** and write it down. Positive is not a moral compliment. It is the class you chose to care about. In this class, positive means fraud, unless a dataset codes it the other way. If someone flips the coding so that 1 means legitimate, every formula still runs and every conclusion is backwards. State the coding before you compute.

|  | We predicted fraud | We predicted legitimate |
|--|--------------------|-------------------------|
| It really was fraud | **TP**, true positive, a catch | **FN**, false negative, a miss |
| It really was legitimate | **FP**, false positive, a false alarm | **TN**, true negative, correctly left alone |

Rows are reality. Columns are the model’s claim. Cover one cell and say who was harmed. A false negative is a stolen payment you allowed. A false positive is a genuine customer you blocked or delayed. Both are errors. They are not equally expensive, which is why one accuracy cannot represent them.

**Recall** is also called **sensitivity** and the **true positive rate (TPR)**. **Specificity** is the twin for the negative class: of the truly legitimate payments, how many did we correctly leave alone? Specificity = TN / (TN + FP).

---

## 🟢 A hundred payments, done in full

There are 100 payments. 1 is fraud. 99 are legitimate. That is the 1% base rate from the quiz.

**Model A always says legitimate.** It never fires.

- TP = 0, because it caught nothing
- FN = 1, the fraud it waved through
- FP = 0, because it raised no alarm
- TN = 99

\[
\text{Accuracy} = \frac{TP + TN}{100} = \frac{0 + 99}{100} = 0.99
\]

\[
\text{Recall} = \frac{TP}{TP + FN} = \frac{0}{0 + 1} = 0
\]

Precision is TP / (TP + FP). Here the model made no positive predictions, so precision is undefined. You do not get to call it 100%. There were no alarms to be right about.

This is the **accuracy paradox**. The model is a disaster for the fraud team and a hero on the accuracy slide. Anyone who ships Model A because “we hit 99%” has not said which question the 99% answered. It answered: “how often did we match the majority label?”

**Model B flags 10 payments, and 1 of them is the real fraud.** So it caught the fraud and annoyed 9 legitimate customers.

- TP = 1, FP = 9, FN = 0, TN = 90

\[
\text{Accuracy} = \frac{1 + 90}{100} = 0.91
\]

\[
\text{Precision} = \frac{TP}{TP + FP} = \frac{1}{1 + 9} = 0.10
\]

\[
\text{Recall} = \frac{1}{1 + 0} = 1.0
\]

Model B has worse accuracy than Model A and is the one that did the job. Precision is only 10%: when it rings the bell, it is usually wrong. Whether that is acceptable depends on what the bell costs. If the bell is “add one review in a queue,” 9 false alarms may be cheap. If the bell is “freeze the customer’s account,” it is not.

**Model C flags 2 payments and both are wrong.** It missed the real fraud.

- TP = 0, FP = 2, FN = 1, TN = 97
- Accuracy = 97 / 100 = 0.97, which still looks like a good student
- Precision = 0 / 2 = 0
- Recall = 0 / 1 = 0

High accuracy, zero precision, zero recall. The matrix is the only reason we can see it. This is why the lab prints `confusion_matrix` and not only `accuracy_score`.

---

## 🟢 The two questions, and the formula that refuses to average them kindly

**Precision** asks: of the alarms we raised, how many were real? The denominator is our positive predictions, TP + FP. A spam filter with low precision marks genuine mail as spam. The user stops trusting the folder.

**Recall** asks: of the real positives, how many did we find? The denominator is reality, TP + FN. A screening test with low recall sends sick people home. A fraud model with low recall leaves the money on the table.

You can have either number alone by being extreme. Flag everybody: recall is 1, precision collapses to the base rate, here 1%. Flag nobody: you raise no false alarms, and recall is 0. The slide’s production sentence is a compromise, not a definition of a perfect model. An example from class: recall at least 85% while precision stays at least 40%, and a human reviews a middle band of scores. Catch most of the fraud. When you alarm, be right often enough that the reviewers are not buried. Leave the uncertain scores to a person.

The **threshold** moves you along that compromise. Lower it and you flag more, so recall tends to rise and precision tends to fall. The default 0.5 is not on the slide because it is sacred. It is on the slide because it is the default, and defaults are how accidental products get made.

**F1** is the harmonic mean of precision and recall:

\[
F1 = 2 \cdot \frac{\text{precision} \cdot \text{recall}}{\text{precision} + \text{recall}}
\]

Why not a plain average? Take precision 1.0 and recall 0.0. A plain average says 0.5, which sounds like a pass. The harmonic mean is 0, because one of the two numbers is 0. That matches the story of Model A’s cousin: a model that is perfectly precise only because it never finds anything. F1 is a summary for when you truly value both. If one error is much more expensive, do not hide behind F1. Look at the expensive cell.

---

## 🟡 The curve names, so the letters in Module 2 are not new

Slide the threshold from “flag nobody” to “flag everybody” and plot the result. Two plots show up in this program.

The **ROC** curve is the receiver operating characteristic. It plots the true positive rate against the false positive rate. **AUROC**, the area under that curve, is the probability that a random real-positive is scored higher than a random real-negative. 0.5 means the ranking is chance. 1.0 means every positive outranks every negative. WeatherWise later uses an example bar of AUROC at least 0.75.

On rare events, the ROC can look cheerful while precision is awful, because the false positive rate’s denominator is the huge pile of legitimate cases. The **precision–recall curve** is the more honest picture for 1% fraud. Module 2 will say AUROC. You should remember when it flatters.

---

## 🟡 Iris, as the mental model of an honest notebook

Iris is a small flower table used because it is clean, not because the career is botany. Four measurements, three species, roughly balanced. Multi-class classification.

The calls, and what each one is for:

1. Load X, the measurements, and y, the species coded 0, 1, and 2.
2. `train_test_split(..., test_size=0.2, stratify=y, random_state=42)`. Hold out 20%. **Stratify** means the species mix in test matches the mix in the full data. On a small table, an unstratified shuffle can put almost all of one species into the exam by accident. **random_state=42** fixes the shuffle. The number 42 is a joke that became a convention. Any fixed integer works. No integer means every run sits a different exam and you cannot tell a better model from a luckier shuffle.
3. `fit` on the training rows only. This is the only call allowed to let labels influence weights.
4. `predict` on the test measurements. Labels do not enter.
5. Print the matrix and `classification_report`.

The report gives precision, recall, and F1 for each species, plus **support**, which is how many true test rows of that species exist. A species with support 2 can show a perfect precision by luck. Read the support before you believe the percentage.

What we do not do: call `predict` on the training rows and celebrate, or fit a scaler on the whole table before the split.

---

## 🟡 Overfitting, underfitting, and the two meanings of “bias” that collide here

| | Underfitting | Overfitting |
|--|--------------|-------------|
| What you see | Poor on train and on test | Excellent on train, poor on test |
| What is wrong | The model is too simple to express the pattern | The model memorized noise in this sample |
| What you try | Richer features, a more flexible model | More data, regularization, a simpler model |

Statisticians call underfitting **high bias**: systematically too rigid. They call overfitting **high variance**: a different sample would have produced a very different model. This “bias” is not the neuron’s bias term, and it is not the social bias of Module 2. The **bias–variance tradeoff** says simple models tend toward bias and flexible models tend toward variance. You spend flexibility until the test score, not the training score, stops improving.

The table cannot diagnose leakage. Both scores excellent, and then production poor, is the signature of a leaked test or of drift. Train excellent and test poor is the signature of overfitting on a fair split, or of a train and test that were never the same problem.

---

## 🔴 Cross-validation, and the time-series warning

One 80/20 cut can be lucky. **Cross-validation** cuts the training data into k **folds**. Train on k − 1 of them, score the leftover, and rotate until every fold has been the leftover once. The mean is a steadier estimate. The spread tells you whether the score is stable or whether one lucky fold is carrying it. k = 5 is a common default.

The scaler and the model have to sit inside the same pipeline and be refit on each training fold. If you scale the whole dataset first, every fold has already seen the held-out fold’s mean. That is leakage with a more impressive function name.

**CV** in `cross_val_score` means cross-validation. In a vision paper, CV means computer vision. Read the sentence.

Cross-validation with a random fold is the wrong tool for rain. A random fold lets Tuesday train the model that is examined on Monday. Module 2 will insist on a chronological split. The principle is already here: the exam must be data the model would not have seen at the time of the decision.

---

## 🟡 When the classes are not 50/50

The toolkit from class:

- Do not report accuracy alone.
- Look at precision and recall, or a precision–recall curve.
- Tell the loss that a rare positive counts more, which is a **class weight**.
- Or resample the training fold: **oversampling** duplicates or synthesizes rare rows, **undersampling** drops majority rows. Do this only inside training. A synthetic copy that lands in both train and test is leakage.
- Do **error analysis**: read actual false positives and false negatives. The metric says how many. The rows say why. If every miss is a new merchant category, you have a data problem, and a new algorithm will rediscover it.

---

## Quiz 3, with the reason next to the letter

| The dilemma | The answer | Why it is not the other answers |
|--------------|------------|---------------------------------|
| RBI requires the factors behind a rejection | A glass-box model, plus SHAP if the model is an ensemble | Accuracy does not answer “why this person.” |
| 99% accuracy, and 1% of cases are tumors | The accuracy trap. Demand recall | Model A from the board would score 99% and find no tumors. |
| Predict the IPL coin toss | Do not build. The label is noise | No feature is related to a fair toss. Gate 2 from the lifecycle class. |
| 99% on train, 52% on test | Overfitting, and go look for leakage too | The model learned the practice sheet. If the test was contaminated, even this warning can disappear, so check the split either way. |

---

## 🟡 One more matrix, small enough to finish in the margin

A screening set has 20 people. 5 really have the condition. The model flags 8 people. Of those 8, 4 really have it. So it missed 1 sick person and flagged 4 healthy ones.

Fill the cells before you read on.

- TP = 4 (flagged, and sick)
- FP = 4 (flagged, and healthy)
- FN = 1 (not flagged, and sick)
- TN = 11 (not flagged, and healthy). Check: 4 + 4 + 1 + 11 = 20.

Accuracy = (4 + 11) / 20 = 15/20 = 0.75.

Precision = 4 / (4 + 4) = 0.50. Half of the alarms are real.

Recall = 4 / (4 + 1) = 0.80. We found four of the five sick people.

F1 = 2 × (0.50 × 0.80) / (0.50 + 0.80) = 0.80 / 1.30 ≈ 0.615.

Now change only the threshold story. Suppose the clinic says a missed person is much worse than an extra test, and they would rather flag 12 people if that catches all 5. Recall would go to 1. Precision would fall, because the extra flags are mostly healthy. Accuracy might fall too. The clinic is still more successful. Write that in the margin so the 0.75 does not hypnotize you: the number moved in the “wrong” direction and the product got better.

**Class weight, said with this table.** If the training loss treats each of the 15 healthy people as equal to each of the 5 sick people, the optimizer is under pressure from the healthy majority. A class weight can count each sick person three times as heavily in the loss, so ignoring them stops being cheap. The weight does not change the test matrix. It changes what the training is willing to get wrong. You still report precision and recall on an unweighted test set, or you will be grading your own reweighted fantasy.

## Checkpoint

Precision is 100% and recall is 10%. Say what the model is doing, in counts, and name a product where that would be acceptable and one where it would be a failure.

One consistent picture: the model made 10 alarms, all of them real, and there were 100 real positives, so it missed 90. It is extremely cautious. That is acceptable when a false alarm is intolerable and missing most cases is tolerable, for example a fully automated irreversible action. It is a failure for fraud or for tumors, where the missed positives are the harm you built the system to reduce.

**Next class:** [05-python-ml-workflow-and-pitfalls.md](05-python-ml-workflow-and-pitfalls.md)
