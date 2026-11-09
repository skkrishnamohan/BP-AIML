# Class notes — The cases, the quizzes, and the insurance lab

*Week 1 slides and the coding reading, gathered into one rehearsal.*

This is the class where we stop introducing words and start using them under exam pressure. For each case, the method is the same. Read the output. Decide whether it is a rule, a class, a quantity, or a new paragraph. Only then name the tool. The industry in the question is camouflage.

---

## 🟢 Fraud, from a single row to a live decision

One historical row looks like this. The features X might be the amount, the distance from the customer’s usual place, and the hour. The label y is 1 if the payment was later confirmed as fraud and 0 if it was legitimate. Training is the search for weights that make those historical rows less wrong. Inference is one new payment, scored before anyone knows whether it will be charged back.

Walk the live path, because the quiz version hides it.

1. The payment switch assembles only features that exist now. “This was charged back next week” is a label, not a feature. Using it in training as if it were known at the swipe is leakage.
2. The service applies the same preprocessing the training run froze. If training scaled amount by the training mean, serving does too. Raw rupees into scaled weights are a different model.
3. The frozen model returns a probability. Frozen means these weights will not update while the customer waits. Updating on one request would be too slow and would let one payment change the model for everyone else.
4. Thresholds, which are rules, turn the probability into an action: block, step-up authentication, or allow. A middle band can go to a person. That is the review band from the metrics class.
5. Chargebacks and analyst decisions arrive later. They become labels for the next training set. They do not arrive in the 15 milliseconds.

**Chargeback** is the later dispute where the real customer says “I did not make this payment.” It is a delayed label. **txn** in the slides just means transaction.

Why this is ML and not five rules: the signal is many weak clues, and attackers adapt as soon as a rule is known. Why it is not an LLM: the product needs a calibrated probability on a millisecond budget, not a paragraph. Why accuracy will betray you: almost every row is legitimate, so a model that never blocks scores near 99% and catches nothing. Ask for recall at a precision the review team can staff.

---

## 🟢 Quiz 1, the way we argued it

| Case | Answer | The sentence that decides it |
|------|--------|------------------------------|
| GST at 18% | Rules | The output is a published rate. A different answer is a bug, not a pattern. |
| UPI fraud shield | Predictive ML | Many signals, a millisecond budget, and an adversary who reads your rules. |
| A brief on an RBI policy | GenAI | Long text in, new text out, no single numeric label. A person still checks it. |
| Password length and a digit | Rules | One boolean expression. Learning it would only make the audit worse. |
| Tomorrow’s grid load | Regression ML | The output is megawatts, a quantity. Classification would throw the amount away. |

If you missed one, it was probably because you answered from the industry. Banking appeared three times and got three different tools. The output changed. The logo did not.

---

## 🟢 Quiz 2, only one question: what type is y?

| Case | Answer | What y actually is |
|------|--------|--------------------|
| A Bengaluru flat’s price in rupees | Regression | A quantity |
| Spam versus inbox | Binary classification | Two categories |
| Normal, pre-diabetic, or diabetic | Multi-class | Three categories |
| Delivery ETA in minutes | Regression | A quantity. ETA is estimated time of arrival |
| Churn, yes or no | Binary classification | Two categories. Churn means the customer leaves |

The medical line is the one the class argues about. The three states have an order, so they are ordinal categories. You might be tempted to code them 0, 1, 2 and run a regression. That pretends the step from normal to pre-diabetic equals the step from pre-diabetic to diabetic. The course answer is multi-class, which keeps them as categories. If a later product truly needs a probability of diabetes rather than a name, you redefine y. You do not quietly change the meaning by choosing a convenient model.

---

## 🟡 The auto-loan inference story, timed

The slide has four beats.

1. The new customer’s features are scaled with the training statistics.
2. They are plugged into weights that are already frozen.
3. The model returns P(buy) = 88%.
4. A business rule says 75% and above receives a pre-approved offer.

The latency target in the story is under 10 milliseconds, because this sits on a mobile banking screen. That single constraint removes a slow generative call. A small classical model can do one forward pass, the weighted sum and the sigmoid, inside that budget. A model that needs a second has failed the product even if its training accuracy was better.

What 88% is, and what it is not. It is the model’s estimated probability that a customer who looks like this will buy, and it means that in the calibrated sense only if you have checked calibration. It is not a promise. The 75% line is policy sitting on top of the model. Customers between two cutoffs can go to a human. Training, remember, was the long phase that saw historical buy/no-buy labels. This request does not see whether the customer eventually buys. That outcome is next quarter’s label.

---

## 🔴 The insurance homework, what to notice when you rerun it

Features: age, income, education. Target: `have_insurance`. Binary, so the story ends in a sigmoid and the pain is log loss.

The coding PDF either implements the weight update with pandas and a loop, or it calls sklearn. Both should be the same problem. The update you are implementing is

\[
w \leftarrow w - \alpha \frac{\partial L}{\partial w}
\]

which is the downhill step from the neuron class. Print after every stage, or the notebook will “work” while you have no idea which stage did the work.

| After this step | What you print | The lesson if it looks wrong |
|-----------------|----------------|------------------------------|
| Load | Row count, dtypes, a sample of education | Education is still text. A model cannot multiply a string. |
| Split X and y | The list of columns in X | If the target is in the list, stop. |
| Encode | Dummy columns, one level dropped | Integers 1, 2, 3 invent an order you may not have justified. |
| Scale | Income brought next to age | Otherwise one gradient step is almost entirely about income. |
| One epoch | The loss | If it explodes, reduce the learning rate or finish scaling. Do not add epochs to an explosion. |
| Test | The confusion matrix | Accuracy alone will hide which kind of mistake you make. |

Squared error will still move the weights. It is the wrong pain for this target, because it does not specially punish a probability of 0.99 on a customer who does not have insurance. Use log loss in the handmade loop. Sklearn’s logistic regression is already optimizing that family of loss, with a more stable solver and its own intercept.

You are done with the lab when you can point at one test row and say: these were the inputs, this was the probability, this was the true label, and this is the cell of the matrix it landed in.

---

## Where Module 1 sits in the program

1. **Applied AI foundations.** The concepts in these seven classes.
2. **Data engineering for AI.** The next folder. The ingredients and the plumbing.
3. **Model engineering.** Architectures and tuning beyond a deliberate baseline.
4. **AI quality.** Tests for models, not only for ordinary functions.
5. **Platform and MLOps.** A path that retrains and serves without a person’s laptop.
6. **Responsible AI and the capstone.** The five pillars on one real decision.

The reason you cannot skip to model engineering: a clever architecture on a leaked, unlabeled, or unfair file learns the leak, the missing label, or the unfairness. Module 2 is the module about refusing to train on that file.

---

## The mastery checklist, said as things you can do at the board

- Draw AI containing ML containing the generative lab, and place an LLM inside the lab. Say one product that sits in ML and is not generative.
- Take three new one-line problems and assign rules, ML, or GenAI by looking at the output.
- From a matrix, write precision as TP / (TP + FP) and recall as TP / (TP + FN), and say which harm each one measures.
- Describe an 80/20 split and give the five-income leakage example, or any example where a statistic was fit before the split.
- Sketch inputs, weights, a sum, a sigmoid, and a threshold, and say who chooses the threshold.
- Name three responsible-AI pillars and one action each pillar requires this week, not a synonym.

When you can do most of those without opening the notes, start the data-engineering folder in order. The first class there will ask who, what, why, and where, which is the problem-definition phase you already have.

**Next:** [../Data-Engineering-for-AI-Systems/00-START-HERE.md](../Data-Engineering-for-AI-Systems/00-START-HERE.md)
