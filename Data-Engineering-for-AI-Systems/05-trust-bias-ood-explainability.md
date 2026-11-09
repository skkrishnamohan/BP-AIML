# Session notes — Trust: bias, OOD, explainability

*Module 2 Session 3 + Labs 5, 6 + Trust Gate lab.*

Trust in this module is not a feeling about the model. It is a set of checks that can fail a dataset or a prediction before a customer is affected. Three families of checks appear here: unfair outcomes (**bias**), inputs the model should not score (**out of distribution**), and reasons you can inspect (**explainability**).

---

## 🟢 Four kinds of bias

| Type | Example |
|------|---------|
| **Data** | Stations only near cities; loan history reflects past discrimination |
| **Algorithm** | Optimizing overall accuracy ignores rare group |
| **Model** | Uses **proxy** (postcode → group) even if group column removed |
| **Evaluation** | Average metrics hide group failure |

**Bias can enter at every stage** — check at each arrow in the pipeline, not once at the end.

**What bias means in this file:** A systematic skew that makes outcomes worse for some people or some places, beyond what a fair decision procedure would produce. It is not the same word as the bias term `b` in a neuron, and not the same as statistical bias of an underfit model. When you hear “bias,” ask which of the three vocabularies the speaker is using.

| Type | What is going wrong | Where to catch it |
|------|---------------------|-------------------|
| **Data bias** | The sample or the labels are skewed before any model exists | Collection and labeling. City-only stations will not represent a valley farm. Historical loan labels can repeat past discrimination. |
| **Algorithm bias** | The objective ignores the harm. Overall accuracy is happy to be right on the majority. | Metric choice. Optimize the group that can be harmed, not only the average. |
| **Model bias** | The fitted model uses a stand-in feature. Remove the sensitive column and the postcode still carries it. That stand-in is a **proxy**. | Feature review and correlation checks, not only the feature list |
| **Evaluation bias** | You publish one number that hides a failing slice | Always report the metric per group that matters |

**Why one check at the end is late:** A proxy discovered after launch has already been served. A missing rural station discovered after training means the model never saw the population. Check at the arrow where the defect enters.

---

## 🟡 LoanLens (human-centric case)

- **Task:** approve/decline loans
- **12k** synthetic applicants; group B ~30%
- **Historical labels** favored group A
- Features: income, credit, debt ratio, **postcode score** (proxy corr ≈ **-0.61** with group B)

| Metric | Group A | Group B |
|--------|---------|---------|
| Historical approval | ~75.5% | ~44.2% |

**Fairness through unawareness fails** when proxies exist.

**What the case is:** A teaching dataset, synthetic, so you can inspect unfairness without using real applicants’ data. Twelve thousand rows. Group B is about 30% of applicants. The historical approval labels already approve group A far more often (about 75.5% versus about 44.2%).

**What the model will do if you do nothing:** Supervised learning copies `y`. If `y` encodes the historical gap, the model’s job is to reproduce it, especially when a feature helps it identify the group.

**Proxy, defined:** A feature that is not the sensitive attribute but is correlated with it enough to reconstruct it. `postcode_score` correlates about **−0.61** with group B. Correlation near 0 is unrelated. Correlation near −1 or +1 is a strong linear relationship. −0.61 is strong enough that the model can treat postcode as a substitute for group.

**Fairness through unawareness:** The hope that deleting the column `group_b` makes the model fair because it “cannot see” the group. It fails when any remaining column still sees the group. Unawareness is not a mitigation. It is how the proxy hides.

**Why this is human-centric:** The numbers are approval rates for people, not a leaderboard. A lower score for group B is a denied loan, not a rounding error.

---

## 🟡 Fairness metrics (know tradeoffs)

| Metric | Meaning |
|--------|---------|
| **Disparate impact (DI)** | Approval rate B / rate A — **< 0.8** often “four-fifths” concern |
| **Statistical parity diff** | Rate B − rate A |
| **Equal opportunity diff** | TPR gap among **truly creditworthy** |

Metrics **conflict** — choose with stakeholders which harm matters.

**Disparate impact:** Divide the favorable-outcome rate of the protected group by the rate of the reference group. Using the lab’s historical rates, \(0.442 / 0.755 \approx 0.59\). The **four-fifths rule** is a long-standing regulatory rule of thumb: a selection rate below 80% of the reference group’s rate is a warning to investigate, not an automatic legal conclusion. The course uses 0.8 as that warning line.

**Statistical parity difference:** The same comparison as a subtraction, rate B minus rate A, about \(0.442 - 0.755 = -0.313\). Zero would mean equal approval rates. DI and this difference describe the outcome rates and do not ask who was actually creditworthy.

**Equal opportunity difference:** Among people who truly should have been approved (the true positives’ population), compare true-positive rates. **TPR** is the true positive rate, which is recall. This metric allows different overall approval rates if they are explained by different base rates of creditworthiness, and it flags a model that finds the creditworthy members of one group and misses the other.

**Why the metrics conflict:** Forcing equal approval rates (parity) can mean rejecting qualified people in one group or approving unqualified people in another. Optimizing equal opportunity can leave the raw approval rates unequal. There is no metric that satisfies every definition of fair at once. Stakeholders have to say which harm they are refusing to accept.

**Where students stop too early:** Computing DI, seeing a failure, and deleting a column. The next section is the set of actions that can actually change the harm.

---

## 🟡 Mitigation timeline

| Phase | Actions |
|-------|---------|
| **Pre-processing** | More data for underrepresented groups, reweigh/resample, fix labels, remove proxies |
| **In-processing** | Fairness constraints, adversarial debiasing, Fairlearn |
| **Post-processing** | Group thresholds, reject-option, human review band |

**Prefer fixing data first** — helps every future model.

**SageMaker Clarify:** bias reports pre/post training; wire as **pipeline gate**.

| Phase | What you change | Why it helps | The tradeoff |
|-------|-----------------|--------------|--------------|
| **Pre-processing** | The dataset, before any model | Every later model inherits the repair. Collect more group-B examples, fix bad labels, drop or rebuild a proxy, or reweigh rows so the loss cannot ignore them. | You can destroy a legitimate signal if the “proxy” is also a real risk factor. That decision needs a domain owner, not only a correlation. |
| **In-processing** | The training objective | Add a constraint so the optimizer is penalized for a fairness gap. **Adversarial debiasing** trains a second model that tries to guess the group from the first model’s representation; the first model is penalized when the adversary succeeds. **Fairlearn** is a Python library that implements several of these constraints. | The average metric may drop. You are spending accuracy to buy a fairer error rate. |
| **Post-processing** | The threshold after training | Different cutoffs per group, or a band of scores that go to a human (**reject option**). | Thresholds that use group membership at decision time may themselves be restricted by policy or law. Know the rule before you ship group-specific cutoffs. |

**Why data first:** A constraint on today’s model does not stop the next model from learning the same proxy. A removed proxy and a repaired label benefit every retraining.

**Clarify** is the SageMaker component that computes bias and explanation reports. A **pipeline gate** means the training pipeline stops, or refuses to promote the model, when Clarify’s numbers miss the bar. A report nobody blocks on is a PDF, not a gate.

---

## 🟡 Trust Gate lab (data engineer view)

**Purpose:** decide if data is fit to use **before** modeling.

**Bias checks (loan CSV, no model needed):**

1. **Representation** — group B ≥ 10% of rows?
2. **Disparate impact** on historical approvals ≥ 0.80?
3. **Proxy** — any feature |corr| with group ≥ 0.50?

**Course checkpoint:** DI ≈ **0.59** → **FAIL**; `postcode_score` proxy → **FAIL**.

> Removing `group_b` column does **not** fix bias if proxy remains.

**Drift check (weather):** **PSI** on features vs training reference.

| PSI | Interpretation |
|-----|----------------|
| < 0.10 | Stable |
| 0.10–0.25 | Watch |
| > 0.25 | Major shift |

**Routing:** pass → `curated/`; fail → `quarantine/`; publish metrics to **CloudWatch** + JSON on S3.

**What the gate is for:** Stop unfit data from becoming the training set. This is a data engineer’s control. You do not need a trained model to see representation, historical disparate impact, or a proxy correlation.

**How to read the three loan checks:**

| Check | Pass means | Why this threshold |
|-------|------------|--------------------|
| Representation ≥ 10% | Group B is present in more than a token sample | A 1% slice cannot support a conclusion about that group, and a model can ignore them at almost no accuracy cost |
| DI ≥ 0.80 | Historical approval rates are inside the four-fifths warning | Below that, the labels themselves are the incident |
| Absolute correlation < 0.50 | No feature is a strong linear stand-in for the group | 0.50 is the course’s alarm. The observed postcode correlation of about 0.61 exceeds it. |

**The checkpoint is a double failure:** DI about 0.59 is under 0.80, and the proxy is over 0.50. Dropping `group_b` changes neither number if postcode remains. The gate should still fail.

**PSI, the population stability index,** compares a feature’s distribution now with a reference distribution from training. It bins the values and sums how much the share in each bin has moved. The course bands are the usual ones: under 0.10 is stable, 0.10 to 0.25 deserves a look, above 0.25 is a major shift. PSI does not say the model is wrong. It says the inputs are different and the old score is less trustworthy.

**Quarantine** is a prefix for data that failed the gate. It is kept for debugging and not consumed by training. **CloudWatch** is AWS monitoring: metrics and alarms. Publishing there means a failed gate can page a person, not only sit in a JSON file nobody opens.

---

## 🟡 Out-of-distribution (OOD)

Model always outputs something — **OOD detection** says “I shouldn’t trust this.”

| OOD type | Example |
|----------|---------|
| Unseen population | Desert station, trained on temperate |
| Broken sensor | Stuck 870 hPa / 99% humidity |
| World change | New climate regime |
| Out-of-scope ask | Weather bot asked for legal advice |

**Detector families:**

1. **Output-based** — low max softmax probability (model-dependent)
2. **Density-based** — Mahalanobis, Isolation Forest
3. **Distribution-based** — **PSI**, KS test (Model Monitor)

**Production pattern:** OOD gate → abstain / human / fallback → log for future labels.

**What OOD means:** The input is outside the distribution the model was trained on. The model still emits a number, because that is what a function does. The number is not evidence that the input was valid.

| Type | What changed | Why the score is untrustworthy |
|------|--------------|--------------------------------|
| Unseen population | A new kind of place or customer | The patterns that separate rain in a temperate city may not apply in a desert |
| Broken sensor | A value stuck at an impossible or constant reading | 870 hPa is an extremely low pressure; 99% humidity stuck there is a fault, not a climate |
| World change | The real relationship moved | Training winters no longer describe this winter. This is concept drift over a longer scale. |
| Out of scope | The question is a different task | A weather model asked for legal advice has no label space for that answer |

**hPa** means hectopascal, the usual unit of atmospheric pressure. Typical sea-level pressure is about 1013 hPa. 870 hPa is typhoon-eye territory, not a normal stuck reading to trust blindly.

**Detector families:**

| Family | What it looks at | Strength | Weakness |
|--------|------------------|----------|----------|
| **Output-based** | The model’s own confidence, often the largest **softmax** probability. Softmax turns a vector of scores into positive numbers that sum to 1. | No extra model | A wrong model can be confidently wrong. Deep networks are especially overconfident far from training data. |
| **Density-based** | Whether the input sits in a dense region of training inputs. **Mahalanobis** distance measures distance from the training cloud, accounting for correlation between features. **Isolation Forest** is a tree method that flags points that are easy to isolate, a sign they are rare. | Sees strange inputs even when the classifier is confident | Needs a maintained picture of “normal” |
| **Distribution-based** | A shift between a recent window and training. PSI is one. A **KS test** (Kolmogorov–Smirnov) asks whether two samples could come from the same distribution. **Model Monitor** is the SageMaker service that watches live data for this. | Good for slow drift across many requests | One weird row is not a distribution shift. Pair it with a per-row detector. |

**The production pattern:** If the detector fires, do not show the raw score as if it were normal. Abstain, send it to a human, or use a safer fallback (yesterday’s forecast, a rules baseline). Log the case. Those logs are the future labels and the evidence that the gate works.

**Lab 5’s lesson:** A baseline DI check flags LoanLens, and Mahalanobis catches the desert station and the broken sensor better than “the softmax was unsure.” Do not rely on confidence alone.

---

## 🟡 Explainability — three lenses

| Lens | Question | Tools |
|------|----------|-------|
| **Sample** | Why this row? | Nearest neighbors, prototypes |
| **Feature** | What inputs matter? | Permutation importance, **SHAP**, PDP |
| **Distribution** | Does today match training? | **PSI**, KS |

**Weather model insight:** shuffling **pressure** hurt AUROC most — matches meteorology (sanity check).

| Lens | What you do | What the answer sounds like |
|------|-------------|-----------------------------|
| **Sample** | Find training rows closest to this one, or a **prototype**, a real example that represents a neighborhood | “This day looks like these ten historical fog days.” Useful for a human reviewer. |
| **Feature** | Measure how much each input moves the prediction | “Pressure mattered more than wind.” |
| **Distribution** | Compare today with training | “Humidity has shifted; I would not trust yesterday’s explanation either.” |

**Permutation importance:** Shuffle one column at random and see how much the metric drops. If shuffling pressure hurts AUROC the most, the model was using pressure. That matches the meteorology you expect, which is a sanity check. If shuffling a leaky column named `rain_tomorrow` hurt the most, the model was cheating.

**PDP** means **partial dependence plot**. It shows the model’s average prediction as you vary one feature and hold the others as they appear in the data. It is a global picture, smoother and less local than one row’s SHAP values.

**Why explanations are not only for regulators:** They are how you catch a clever defect. A model that predicts rain from the station id, because one station’s sensor is broken, will say so if you look at feature importance. Accuracy will not.

---

## 🟡 SHAP (intuition)

From game theory: fairly split “payout” (prediction) among features.

- **Local:** why **this** day got high P(rain)
- **Global:** average |SHAP| across rows

**TreeSHAP** fast on tree models; **SageMaker Clarify** at scale.

**Cookie vs dog story:** explanation shows model used **eye/nose shape** — fix with hard negatives + augmentation.

**The game-theory idea, in plain language:** Imagine the features as players on a team that produced a score. A fair credit gives each player their average contribution across the orders they could have joined the team. That credit is the Shapley value. **SHAP** adapts it to a model’s output. The values for one prediction add up to the difference between that prediction and the average prediction. That property is why people call them additive explanations.

**Local vs global:** One day’s SHAP tells a forecaster why this morning looks rainy. The mean of the absolute SHAP values across many days tells you which features the model relies on in general. A feature can matter globally and not on this particular day.

**TreeSHAP** is an exact, fast algorithm for tree ensembles. You do not need it for a two-coefficient logistic regression; you can read the coefficients. You do need something like it for a large boosted-tree model. Clarify runs explanation jobs without you hosting the algorithm yourself.

**The cookie vs dog story:** A vision model seemed to recognize dogs, and the explanation showed it was using eyes and nose shape that also appear on a cookie in the dataset’s background, or the reverse failure the class described: the explanation revealed the real cue was not the one you hoped. **Hard negatives** are examples that look similar and have the other label, so the model must learn the true difference. Augmentation stops it from relying on a background accident. The lesson is general: an explanation that surprises you is a debugging tool, not a decoration.

---

## 🟡 Transparency artifacts

| Document | Contents |
|----------|----------|
| **Data card** | Source, labeling, gaps, version, intended use |
| **Model card** | Metrics per group, limits, training data version |
| **Lineage** | Which data + code + params → this artifact |
| **ADR** | Architecture decisions & rejected options |

| Artifact | What it is for | What “done” looks like |
|----------|----------------|------------------------|
| **Data card** | A label on the dataset, the way a model card is a label on a model | A new teammate can see source, licence, who labeled it, known gaps (no desert stations), the version, and what it must not be used for |
| **Model card** | The release note for a model | Metrics overall and per group, the threshold, the training data version, and the cases where it should abstain |
| **Lineage** | The parentage of an artifact | “Model 17 came from curated version 4, git commit abc, hyperparameters in this config.” This is the reproducibility triple from the versioning notes. |
| **ADR** | Architecture decision record | A short note: the decision, the options rejected, and why. “We chose batch scoring by 6 a.m. rather than an edge device because stations have reliable nightly connectivity.” |

**Why these exist:** Six months later the person who remembers has left. The card and the lineage are how the trust gate stays explainable to the next person.

---

## Lab 5 & 6 checkpoints

- **Lab 5:** baseline DI flagged; Mahalanobis catches desert/broken sensor better than naive softmax
- **Lab 6:** PSI flags temp/humidity/pressure; wind/cloud stable → targeted investigation

**How to talk about lab 6:** PSI above the major-shift band on temperature, humidity, and pressure, with wind and cloud stable, is not “retrain everything.” It is a clue that those three sensors or that climate slice changed. Investigate the instrument and the region before you spend a training run learning the fault.

---

## Checkpoint

Why did removing `group` from features not remove bias in LoanLens?

**Answer:** Group membership was still available through a proxy. `postcode_score` correlates about −0.61 with group B, which is above the trust gate’s 0.50 alarm. The historical labels also approve group A at about 75.5% and group B at about 44.2%, a disparate impact near 0.59. A model trained on those labels and that postcode can reproduce the gap without ever seeing the group column. Fairness through unawareness fails when a stand-in feature remains.

**Next:** [06-scale-integration-and-labs.md](06-scale-integration-and-labs.md)
