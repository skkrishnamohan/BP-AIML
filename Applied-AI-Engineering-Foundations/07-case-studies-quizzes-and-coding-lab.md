# Session notes — Case studies, quizzes, coding lab tie-in

*Consolidates Week 1 slides + coding reading.*

Use this file to rehearse decisions. Each case below says what the target is, why that tool fits, how the system would actually run, and which words in the question are doing the work.

---

## 🟢 Fraud detection blueprint (features → label)

Each transaction row:

- **X:** amount, distance from home, time of day, …
- **y:** 1 = confirmed fraud, 0 = legit

Training = find weights that separate patterns. Inference = score new txn in milliseconds.

**What you are building:** A binary classifier. The positive class is fraud. The output that matters is a probability, then a threshold, then an action (block, step-up authentication, or let it through).

**Why ML instead of a single rule:** Fraud is many weak signals together. Amount alone is not enough. Amount, plus a new device, plus a city far from home, plus an odd hour, is a pattern that changes as attackers adapt. Five static rules lose that race. That is the faculty test from the first notes.

**How a live decision flows:**

1. The payment switch assembles only features knowable now. It cannot use “this payment was later charged back.”
2. The same preprocessing as training runs inside the service.
3. The frozen model returns a probability.
4. Policy thresholds turn the probability into an action.
5. Later, chargebacks and analyst decisions become new labels. They arrive late, so they feed the next training set, not this request.

**Where the metric trap sits:** Almost every row is legitimate. Accuracy will flatter a model that never blocks anyone. Ask for recall at a precision the review team can afford.

**txn** is just “transaction.” **Chargeback** is the later event where the real customer disputes the payment. It is a delayed label, not a feature at scoring time.

---

## 🟢 Quiz #1 answers (rules / ML / GenAI)

| Case | Answer |
|------|--------|
| GST 18% | Rules |
| UPI fraud shield | Predictive ML |
| RBI policy brief | GenAI |
| Password validator | Rules |
| Grid load forecast | Regression ML |

**How to see the answer in the wording:**

| Case | The clue | Why the other tools are worse |
|------|----------|-------------------------------|
| GST 18% | A published rate | An LLM might say 18 today and 12 tomorrow. The statute is deterministic. |
| UPI fraud shield | Many signals, milliseconds, attackers adapt | Rules go stale. Generation does not produce a calibrated risk score on time. |
| RBI policy brief | Long text in, new short text out | There is no single numeric `y`. The deliverable is language, so it is GenAI, reviewed by a person. |
| Password validator | A length and a character class | The policy is an `if`. Learning it from examples can only do worse. |
| Grid load forecast | A number of megawatts tomorrow | Predicting a quantity is regression. Classification would throw away the amount. |

---

## 🟢 Quiz #2 answers (classification vs regression)

| Case | Answer |
|------|--------|
| Bengaluru flat price ₹ | Regression |
| Spam vs inbox | Binary classification |
| Normal / pre-diabetic / diabetic | Multi-class |
| Delivery ETA minutes | Regression |
| Churn yes/no | Binary classification |

**The only question that matters:** What type is `y`?

- Rupees and minutes are quantities, so regression.
- Two named states are binary classification, even when the business calls one of them “risk.”
- Three ordered medical states are still classification if the product must name the state. They are **ordinal** categories. If you encode them as 0, 1, 2 and regress, you are pretending the gap from normal to pre-diabetic equals the gap from pre-diabetic to diabetic. The course answer is multi-class, which keeps them as categories.

**ETA** is estimated time of arrival. **Churn** is “the customer leaves.”

---

## 🟡 Bank auto-loan inference story

1. New customer features scaled
2. Plug into **frozen** weights from training
3. Get P(buy) = 88%
4. Business rules: ≥75% → pre-approved offer

**Latency target in story:** < 10 ms for mobile banking.

**What “frozen” means:** Training is over. These weights will not update while the customer is looking at the phone. Updating live on one customer would both be too slow and would let one request change the model for everyone.

**Why scaling is step 1 of inference and not only of training:** The weights were learned on scaled inputs. A raw income in rupees fed to those weights is a different problem. The serving path must apply the training mean and standard deviation.

**What 88% means:** The model’s estimated probability that this customer buys, not a promise. The 75% line is a business rule on top of the model. Customers between two cutoffs can go to a human. That layered design is the “tiers” point from the neuron notes.

**Why 10 ms:** The score sits on the mobile banking path. A person will not wait seconds for a pre-approval hint. This constraint rules out a slow generative call and favors a small classical model.

**Where training and inference differ:** Training saw the historical buy/no-buy label and looped for many epochs. Inference sees one new customer, runs one forward calculation, and does not see whether they eventually buy. That later outcome is a label for a future retraining.

---

## 🔴 Insurance dataset (class + homework)

**Features:** age, income, education → **y:** have_insurance

Walkthrough covered:

- Log loss measures wrong confident predictions
- Gradient updates weights toward less loss

**Your coding PDF** implements this with pandas + manual gradient descent OR sklearn equivalent.

**Study tip:** Run the insurance notebook once line-by-line with print statements after each step.

### What to understand when you rerun it

**The target** is binary, so the neuron ends in a sigmoid and the training loss is log loss, not squared error. Squared error is the more natural fit for a regression price. Using it here would still move weights, but it would not punish a confident wrong probability the way log loss does.

**What you should print after each step:**

| After | What you want to see | What it teaches |
|-------|----------------------|-----------------|
| Load | Row count, dtypes, a few education values | The category is text until you encode it |
| Split X and y | `have_insurance` is not inside X | Leaving it in X is instant leakage |
| Encode | One column per extra education level, not a fake 1, 2, 3 order unless you have justified an order | Nominal versus ordinal |
| Scale | Income no longer thousands of times larger than age | Otherwise the income weight takes over the update |
| One epoch | Loss should usually fall | If it explodes, the learning rate is too high or the scale is wrong |
| Test metrics | A confusion matrix, not only accuracy | You need FP and FN, not a single compliment |

**Manual gradient descent vs sklearn:** The manual loop shows the weight update \(w \leftarrow w - \alpha \partial L / \partial w\). Sklearn’s `LogisticRegression` solves the same problem with a more robust optimizer and an intercept. If your handmade loss falls and then your sklearn test metrics are in the same range, you have connected the story to the library.

---

## Module 1 “you are here” in program roadmap

1. **Applied AI Foundations** ← you completed concepts here
2. **Data Engineering for AI** ← Module 2 notes
3. Model Engineering
4. AI Quality
5. Platform & MLOps
6. Responsible AI & Capstone

**What each later module is for:**

| Module | What it adds | Why it cannot be skipped |
|--------|--------------|--------------------------|
| Data engineering | Reliable inputs, labels, versions | A correct algorithm on a dirty file learns the dirt |
| Model engineering | Better architectures and tuning | Foundations stop at a deliberate baseline |
| AI quality | Tests for models, not only for functions | Accuracy on one split is not a release test |
| Platform and MLOps | Repeatable train and serve paths | A notebook on one laptop is not a system |
| Responsible AI and capstone | The pillars on a real decision | Fairness, explanation, and monitoring have to land in one project |

---

## Module 1 mastery checklist

- [ ] Explain AI ⊃ ML ⊃ GenAI in one diagram
- [ ] Choose rules vs ML vs GenAI for 3 scenarios
- [ ] Write formulas for precision & recall from a confusion matrix
- [ ] Describe train/test split and one leakage example
- [ ] Sketch neuron → sigmoid → threshold
- [ ] List 3 responsible AI pillars and what they mean in practice

**How to use the checklist:** Say the answer aloud before you open the earlier file. If you can only name the term, you are not done; add the why. Example: “Recall is TP / (TP + FN), so it is about missed positives, which is why a fraud team demands it.”

When most boxes are checked, start **Data Engineering** folder in order.

**Next module:** [../Data-Engineering-for-AI-Systems/00-START-HERE.md](../Data-Engineering-for-AI-Systems/00-START-HERE.md)
