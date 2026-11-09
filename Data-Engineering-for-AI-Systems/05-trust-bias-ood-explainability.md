# Class notes — Bias, strange inputs, and explanations

*Module 2, Session 3, Labs 5 and 6, and the Trust Gate lab.*

Trust, in this class, is not a feeling about a model. It is a set of checks that can fail a dataset or a single prediction before a person is affected. Three families of checks: unfair outcomes, inputs the model should refuse to score, and reasons you can inspect when someone asks why.

Before any of them, a vocabulary warning. **Bias** in this file is a systematic skew that makes outcomes worse for some people or some places. It is not the bias term \(b\) in a neuron, and it is not the statistical bias of an underfit model. When a slide says bias, decide which of the three languages it is speaking.

---

## 🟢 Four doors the skew walks through

**Data.** The sample or the labels are skewed before any model exists. Weather stations only near cities will not represent a valley farm. Historical loan labels can repeat past discrimination. You catch this at collection and at labeling, or you do not catch it at all.

**Algorithm.** The objective ignores the harm. Overall accuracy is delighted to be right on the majority and silent on a small group. You catch this when you choose the metric. Optimize for the group that can be harmed, not only for the average.

**Model.** The fitted model uses a stand-in. You delete the sensitive column, and the postcode still carries the group. That stand-in is a **proxy**. You catch it by looking at correlations and at explanations, not by reading the feature list and feeling reassured.

**Evaluation.** You publish one number that hides a failing slice. The repair is dull: report the metric for each group that can be harmed.

The reason to check at every arrow, rather than once at the end, is that a missing rural station cannot be fixed by a fairness constraint. The model never saw those farms. A proxy discovered after launch has already been served.

---

## 🟡 LoanLens, with the arithmetic

The task is approve or decline. The table is synthetic, about **12,000** applicants, so the class can inspect unfairness without real people’s data. Group B is about 30% of the rows. The features include income, credit, a debt ratio, and a **postcode score**. The historical labels already favor group A.

| | Group A | Group B |
|--|---------|---------|
| Historical approval rate | about 75.5% | about 44.2% |

If you train on those labels, supervised learning will try to copy them. That is what supervised learning is. The model is not rebelling. It is obeying `y`.

**Disparate impact** divides the favorable rate of the group you are worried about by the rate of the reference group:

\[
DI = \frac{0.442}{0.755} \approx 0.59
\]

The **four-fifths rule** is an old regulatory rule of thumb: if the selection rate falls below 80% of the reference group’s rate, you investigate. It is a warning, not an automatic legal verdict. The course uses 0.8 as that warning line. 0.59 is under it. The labels themselves are the incident, before any algorithm.

**Statistical parity difference** is the same comparison as a subtraction: \(0.442 - 0.755 = -0.313\). Zero would mean equal approval rates. Both of these metrics look at outcomes. They do not ask who was actually creditworthy.

**Equal opportunity difference** does. Among people who truly should have been approved, compare true positive rates. The true positive rate is recall. This metric can allow unequal raw approval rates when the groups truly have different base rates of creditworthiness, and it flags a model that finds the qualified members of one group and misses the other.

These metrics conflict, and the conflict is the lesson. Forcing equal approval rates can mean rejecting qualified people in one group or approving unqualified people in the other. Optimizing equal opportunity can leave the headline approval rates unequal. There is no number that satisfies every definition of fair. Someone who owns the harm has to say which harm they refuse. That someone is not the default setting of a library.

**The proxy.** `postcode_score` correlates about **−0.61** with membership in group B. Correlation near 0 means unrelated. Near −1 or +1 means a strong linear relationship. −0.61 is strong enough for a model to treat postcode as a substitute for the group. **Fairness through unawareness** is the hope that deleting the column `group_b` makes the decision fair because the model “cannot see” the group. It fails as soon as any remaining column can reconstruct the group. Unawareness is not a mitigation. It is how the proxy hides. The trust gate later uses an absolute correlation of 0.50 as its alarm. 0.61 exceeds it.

---

## 🟡 What you can do, and why the order is data first

**Before training.** Collect more examples from the underrepresented group. Fix labels that are wrong rather than merely inconvenient. Remove or rebuild a proxy, knowing that a feature can be both a proxy and a genuine risk factor. That distinction needs a domain owner, not only a correlation. Reweigh or resample so the loss cannot ignore the smaller group. Everything you repair here is inherited by the next model, not only by today’s model.

**During training.** Change the objective. A constraint penalizes a fairness gap. **Adversarial debiasing** trains a second model that tries to guess the group from the first model’s representation, and penalizes the first model when that guess succeeds. **Fairlearn** is a Python library of these constraints. The average accuracy may fall. You are spending it on purpose.

**After training.** Change the threshold. Different cutoffs per group, or a band of scores that go to a human, which is the **reject option**. Thresholds that use group membership at decision time may themselves be restricted. Know the rule before you ship a group-specific cutoff.

Prefer the data repair. A constraint on today’s model does not stop the next model from learning the same postcode. **SageMaker Clarify** computes bias reports before and after training. Wire the report as a **pipeline gate**: the model is not promoted when the number misses the bar. A report nobody can fail is a PDF.

---

## 🟡 The trust gate, as a data engineer’s control

The gate answers “is this file fit to train on?” It does not need a trained model.

Three checks on the loan file:

1. **Representation.** Is group B at least 10% of the rows? A 1% slice cannot support a claim about that group, and a model can ignore them at almost no cost in accuracy.
2. **Disparate impact** on the historical approvals, at least 0.80.
3. **Proxy.** Does any feature have absolute correlation with the group of 0.50 or more?

The course checkpoint fails twice. DI is about **0.59**, under 0.80. `postcode_score` is a proxy, over 0.50. Deleting `group_b` changes neither fact. The gate still fails. That sentence is the one to remember in an exam: removing the group column does not remove the bias while the proxy remains.

On the weather side, the same style of gate watches **PSI**, the Population Stability Index. PSI bins a feature and sums how much the share of rows in each bin has moved, compared with the training reference. You do not need the full formula to use the course bands.

| PSI | What the class says to do |
|-----|---------------------------|
| Below 0.10 | The feature looks stable. |
| 0.10 to 0.25 | Look. Do not ignore it, and do not panic. |
| Above 0.25 | A major shift. The old score is less trustworthy. |

PSI does not say the model is wrong. It says the inputs are different. A shift can be a broken sensor or a real change in the climate. Those demand opposite responses, which is why “retrain immediately” is not the automatic next step.

**Routing.** Pass, and the file may go to `curated/`. Fail, and it goes to `quarantine/`, kept for debugging and forbidden to training. Publish the metrics to **CloudWatch**, which is AWS’s monitoring service, and also write the JSON on S3. An alarm can page a person. A JSON file alone will not.

---

## 🟡 Out of distribution: the model will answer anyway

A model is a function. Give it an input and it returns a number, including when the input is unlike anything it trained on. **Out of distribution** means you should not trust that number. Detection is the extra component that is allowed to say “abstain.”

Four situations from class:

| Situation | The example | Why the score is hollow |
|-----------|-------------|-------------------------|
| A new population | A desert station, trained on temperate cities | The pattern that separates rain there may not travel |
| A broken sensor | Pressure stuck at 870 hPa, humidity stuck at 99% | That is a fault. **hPa** is the hectopascal. Ordinary sea-level pressure is about 1013. 870 is extreme, not a normal Tuesday. |
| The world moved | A new climate regime | The training winters are no longer this winter |
| The wrong task | A weather bot asked for legal advice | There is no label space for that answer |

Three families of detectors:

**Output-based.** Look at the model’s own confidence, often the largest **softmax** probability. Softmax turns a vector of scores into positive numbers that add up to 1. This needs no extra model. It fails when the model is confidently wrong, which deep networks often are, far from their training data.

**Density-based.** Ask whether the input sits inside the cloud of training inputs. **Mahalanobis** distance measures that, taking correlation between features into account. An **Isolation Forest** flags points that are easy to separate from the crowd, a sign they are rare. These can catch a strange input even when the classifier is sure of itself. They need a maintained picture of “normal.”

**Distribution-based.** Compare a recent window with training. PSI is one. A **KS test**, Kolmogorov–Smirnov, asks whether two samples could share a distribution. **Model Monitor** is the SageMaker service that watches live data this way. A window is good at slow drift and blind to one weird row. Pair it with a per-row detector.

Lab 5’s result is the comparison: the disparate-impact check flags LoanLens with no model at all, and Mahalanobis catches the desert station and the broken sensor better than “the softmax looked unsure.” Do not use confidence as your only gate.

The production pattern: if a detector fires, do not show the raw score as if it were ordinary. Abstain, send the case to a human, or fall back to something safer, such as yesterday’s forecast or a rules baseline. Log the case. Those logs are future labels, and they are the evidence that the gate works.

---

## 🟡 Three ways to explain, and the weather sanity check

| Lens | The question | How you answer it |
|------|--------------|-------------------|
| Sample | Why does this row look like a case we know? | Nearest training rows, or a **prototype**, a real example that stands for a neighborhood. “This morning looks like these ten historical fog days.” |
| Feature | Which inputs moved the score? | Permutation importance, SHAP, a partial dependence plot |
| Distribution | Does today even match training? | PSI, a KS test |

**Permutation importance.** Shuffle one column and see how much the metric drops. In the weather model, shuffling **pressure** hurt AUROC the most. That matches the meteorology, so it is a sanity check: the model is using a physically sensible clue. If shuffling a column named `rain_tomorrow` had hurt the most, the model would have been cheating, and the explanation would have caught what accuracy would have praised.

A **partial dependence plot** shows the model’s average prediction as you vary one feature. It is a global, smoothed picture. It is less local than one morning’s explanation.

Explanations are not only for a regulator. They are how you notice that a model predicts rain from the station id because one station’s sensor is broken. Accuracy will not tell you that.

---

## 🟡 SHAP, without the folklore

Imagine the features as players on a team that produced a score. A fair split gives each player their average contribution across the orders in which they could have joined the team. That idea is a Shapley value, from game theory. **SHAP** adapts it to a model. For one prediction, the feature contributions add up to the difference between that prediction and the average prediction. “Additive” is that bookkeeping property, not a slogan.

**Local** SHAP is why this day got a high probability of rain. **Global** importance, in this method, is the average of the absolute SHAP values across many days. A feature can matter in general and not matter this morning.

**TreeSHAP** computes this quickly and exactly for tree ensembles. A two-coefficient logistic regression does not need it. You can read the coefficients. A large boosted model does need something like it. Clarify can run the explanation job so you are not hosting the algorithm yourself.

The cookie-and-dog story is the debugging lesson. A vision model appeared to recognize the animal, and the explanation showed it was relying on a cue you did not intend, the shape of eyes and nose in a way that also fit a distraction in the images, such as a cookie. The repair is **hard negatives**, examples that look similar and have the other label, plus augmentation so a background accident is not a reliable clue. An explanation that surprises you is a debugging tool. If it only ever confirms what you hoped, you are not reading it.

---

## 🟡 The documents that outlive the person who remembers

A **data card** describes the dataset the way a label describes a product: source, licence, who labeled it, known gaps such as “no desert stations,” the version, the intended use, and the uses that are forbidden.

A **model card** is the release note: metrics overall and by group, the threshold, the training-data version, and the cases where the model should abstain.

**Lineage** is the parentage. Model 17 came from curated version 4, this git commit, and these hyperparameters. That is the reproducibility triple.

An **ADR**, an architecture decision record, is a short note of a decision, the options rejected, and why. “We score overnight rather than on the device, because the stations have a reliable nightly connection and the farmer acts at 6 a.m.” Six months later that sentence is the difference between a design and a rumor.

---

## Labs 5 and 6, in one paragraph each

Lab 5: the baseline disparate impact is already a failure, and the distance-based detector catches the desert station and the stuck sensor where a naive confidence score does not. Lab 6: PSI flags temperature, humidity, and pressure, while wind and cloud look stable. That pattern is a targeted investigation of those three measurements, not a blanket retrain. A retrain would happily learn a broken instrument.

---

## Checkpoint

Why did deleting the group column fail to remove the bias in LoanLens? Use the two numbers.

The historical approval rates are about 75.5% and 44.2%, so disparate impact is about 0.59, under the 0.8 warning. The postcode score correlates about −0.61 with group B, above the 0.50 proxy alarm. The labels still teach the gap, and the postcode still identifies the group. The model can reproduce the harm without ever seeing the group column.

**Next class:** [06-scale-integration-and-labs.md](06-scale-integration-and-labs.md)
