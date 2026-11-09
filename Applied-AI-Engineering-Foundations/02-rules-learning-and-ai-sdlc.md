# Class notes — Traditional SDLC and the AI SDLC

*Week 1 slides and the readings on the traditional lifecycle versus the AI lifecycle.*

Last class sorted the tools. This class is about the life of the system after you have chosen one. The slide headline is short enough to miss. The sentence we wrote in large letters:

> In an AI system, the data is part of the code.

---

## 🟢 What that sentence means on a Tuesday

**SDLC** is the Software Development Life Cycle. It is the repeating path from a need, through design and build and test, to a release, and then to whatever you do when the release misbehaves.

In an ordinary application, if you want different behavior you change the code. The login function compares a password hash. You can read the function. Two deployments with the same commit behave the same.

In an AI system, two deployments can run the same Python service and make different decisions. One was trained on January’s transactions. The other was trained on July’s. The code is identical. The weights are not, because the data, the labels, or the training run changed. If your release process versions only the Git commit, you cannot explain a decision and you cannot rebuild it.

**Behavior**, in this class, does not mean the color of a button. It means the decision: approve, decline, flag, skip the irrigation, or the paragraph the assistant sends to a customer.

So the practical instruction is: a training dataset is a dependency, the way a library version is a dependency. The release record contains the code version, the data version, and the metric you promised.

---

## 🟢 The traditional lifecycle, as a contrast

Four properties, and one example so they stay concrete.

**Deterministic.** The same input gives the same output. The same password hash is allowed or denied today and next year, unless someone edits the code.

**Logic in code.** You can open the function and read the if statements.

**Bugs are loud.** A failure throws, or it fails a test, or a user hits an error page. You replay the request from logs. You write a test that fails, then a fix that makes it pass.

**Deploy is near the end.** After release, the questions are “is it up?” and “are there errors?” The decision rule does not rot because customers changed their habits. A tax calculation does not become wrong because fashion changed.

The example we used was login. There is nothing to learn. Either the secret matches or it does not.

---

## 🟢 The AI lifecycle, using a spam filter

**Probabilistic.** The output is a score, a ranking, or a generated sentence. It can vary, and it is almost never a proof that the mail is spam. It is an estimate.

**Logic in the data and the model.** The weights were estimated from historical mail that someone had labeled. Change the labels, or change which years you train on, and the filter changes while the service code stays still.

**Failures are often silent.** This is the property people do not believe until they have seen it. The API returns HTTP 200. The latency graph is flat. Nobody gets an exception, because the model is doing exactly what its weights say. Meanwhile the share of spam that lands in the inbox rises, because spammers changed the wording. The code is healthy. The decision is sick. That slow movement is **drift**.

**Deploy is the beginning.** The test set was a rehearsal. The first day in production is the first day the model meets truly new traffic, and the traffic keeps changing. Monitoring is not an extra chapter after the project. It is the project continuing.

We put the two lifecycles side by side because the exam asks it this way:

| Ordinary software | AI system |
|-------------------|-----------|
| Behavior is in the code | Behavior is in the data and the model |
| Same input, same output | A score, a rank, or a sample |
| You test correctness | You test performance and robustness |
| Release means you are done | Release means you start watching |

**Correctness** is pass or fail. The password matched or it did not. **Performance** for a model is “do the errors we care about stay inside the limit?” **Robustness** means the decision stays sensible under noise, a missing value, or a small change of unit. A model that collapses because income arrived in rupees instead of thousands is not robust, even if last month’s test score was excellent.

---

## 🟡 The loop, phase by phase

A **waterfall** finishes each phase once and only moves forward. AI work is a loop. Monitoring sends you back to data and training. Draw it as a circle, not a staircase.

```
Problem → Data → Features → Train → Evaluate → Deploy → Monitor
   ^                                                       |
   +----------------------- Iteration ----------------------+
```

Ethics is not a decoration at the end. It constrains which data you were allowed to collect at the start, and which explanations you must be able to give at the end.

### 1. Problem definition

Who benefits, what decision changes, and what number will tell us it worked? A rain model with no time by which the score must arrive is not defined. “Predict rain” is a topic. “By 6 a.m., give P(rain tomorrow) so the planner can skip irrigation” is a problem.

Write two numbers, not one. The **model metric** scores predictions against labels: recall, RMSE, AUROC. The **business KPI** scores the world: water used, fraud losses, hours saved. A model can win on accuracy and still warn too late to skip the irrigation cycle.

Also write the **integration plan**: the name of the human or the system that consumes the score. If no workflow changes, you have a report, not a product.

And write the constraints in words. **Latency** is the maximum wait. Six in the morning is enough for irrigation. Fifteen milliseconds matters for UPI. **Cost** is training, storage, and every prediction. An LLM call on every payment can cost more than the fraud it stops. **Explainability** is the ability to say which inputs moved a decision, often required when money or a right is at stake. **Compliance** is the set of rules about consent, retention, and features you are not allowed to use.

### 2. Data collection and preprocessing

Where do the rows come from, are you allowed to use them, and do they represent the people you will score later? City weather stations will not teach you a valley farm. Preprocessing is the cleaning: types, units, missing values, and the split into train, validation, and test.

Three slices, because students try to live with two. **Train** is what the weights may study. **Validation** is what you consult while choosing settings. **Test** is the final exam, taken once. The moment you change the model because of the test score, the exam has become practice.

### 3. Feature engineering

Build only signals that will exist at prediction time. A perfect column that appears after the event is cheating. Tomorrow’s rainfall cannot be a feature of a model that predicts tomorrow’s rainfall. This is the door through which **data leakage** walks in. Leakage means information from outside the decision moment sneaks into training. Offline, the model looks brilliant. Live, the clue does not exist, and the score falls over.

### 4. Model training

Start with a **baseline**, the simplest honest model: predict the average, or fit a logistic regression. If a complicated model cannot beat that baseline by enough to matter, ship the simple one. Track the experiment. A notebook you cannot rerun is a rumor.

### 5. Evaluation

Precision and recall, and the slices that can be harmed, not one flattering average. Accuracy on rare fraud is the worked example in the metrics class. Fairness, where the decision affects people, is a required slice and not a polite extra.

### 6. Deployment

Put the model where the decision happens: a batch file, an API, or a device at the edge. The preprocessing in that path must be the same preprocessing as training, with the same means and the same category lists, frozen from the training run. Scaling with the training mean and then serving raw numbers is a different model.

### 7. Monitoring

Watch latency, watch the data, and watch the KPI. Log inputs, scores, and, when they arrive, outcomes.

Two drifts. **Data drift**: the inputs moved. Customers got younger, or the humidity readings sit higher than in training. **Concept drift**: the relationship moved. The same inputs no longer mean the same outcome, because the fraud tactic or the climate changed.

**Label delay** is the awkward practical fact. A chargeback, or a medical result, may arrive days later. You cannot pretend yesterday’s score was confirmed this morning. The monitor has to live with late labels.

### 8. Ethics and compliance

Privacy, explainability, and an audit trail. If you cannot say which data and which code produced a denial, you do not have a system you can defend.

### 9. Iteration

Retrain when the drift or the goal has earned it. Retraining on a calendar, without looking, can also learn a broken sensor. Iteration is a decision, not a cron job with no owner.

### The failures the reading listed, in plain speech

- Building a learner for a problem five rules would solve. You take on drift and explanation cost for no benefit.
- Optimizing accuracy when the business is paying for missed positives, which is recall.
- Leakage, so the test score was never an estimate of the future.
- No monitoring, so the silent failure has weeks to become expensive.

---

## 🟡 The two gates before you are allowed to build

A gate is a stop/go check. Failing it is a successful project. You avoided a model that cannot work.

**Gate 1.** Can fewer than about ten static rules solve it? Then write the rules. Login, GST, password length.

**Gate 2.** Is the outcome pure noise? Then do not build. A fair coin, or the IPL toss in the quiz, has no relationship for a model to find. More data and a larger network still produce a guess. **IPL** is the Indian Premier League. The toss is a stand-in for any label that is not predictable from the features you are allowed to have.

If you pass both gates, pick the family from the shape of the data.

| Shape | Start here | Why |
|-------|------------|-----|
| A table | Logistic regression, trees, gradient boosting | The columns already have meaning |
| Text | Embeddings if you are classifying, an LLM if you are writing | Words are not tidy numbers |
| Images | A convolutional net or a vision transformer | The signal is spatial |

**GIGO** means garbage in, garbage out. The procedure cannot invent a clean label the file does not contain. A clean target and an honest split beat a more fashionable library. Write GIGO in the margin of every data lecture. It is the reason Module 2 exists.

---

## The mini exercise we did out loud

Rain for farmers. The decision is: skip irrigation if the probability of rain is above 0.7, and the score must exist by 6 a.m.

Say what is still missing if all you have is “we will use a neural net.” You do not know the features that exist at 6 a.m. You do not know whether 0.7 was chosen because a missed rain is worse than a wasted watering, or because 0.5 was the textbook default. You do not know who looks at the score. You do not know the KPI. The network is the last and the most replaceable piece.

---

## 🟡 The spam filter, followed for six months

January: the filter is trained, recall on the test set is acceptable, and it is deployed. The service code will not change until June. This is the story that makes “deploy is the beginning” concrete.

February: a new style of spam arrives, short messages with a single link and no keywords the model weighted heavily. Nothing throws. Latency is unchanged. The inbox spam rate, the KPI, starts to rise. Accuracy on the old test set is still the January number, because nobody is recomputing it on February’s mail. A monitor that only watches the old test set will stay green. A monitor that watches the share of mail users mark as spam will not.

March: an engineer retrains on the new labels. The code commit is identical. The data version is not. If the release record says only the git commit, February’s model and March’s model are indistinguishable, and you cannot roll back to the weights that matched a regulatory test. The data version is part of the release.

April: someone adds a feature, “does the message contain a URL?”, and fits the scaler on all mail including the evaluation week. The new score looks wonderful and then falls the next week. That is leakage, and it happened inside a healthy lifecycle, not instead of one.

May: the business asks for fewer false positives, because legitimate mail is being buried. The model metric they were maximizing was recall. The KPI they are now feeling is “genuine mail delayed.” Both numbers were available in January. Only one was written into the problem statement. Changing the threshold is a product decision. It is not a reason to distrust the entire lifecycle.

June: the code finally changes, because the service must log the score. The behavior change everyone argued about in March did not come from this commit. It came from the data. Write that on the diagram so the next incident review looks at both.

## Checkpoint

1. Why do we say deployment is the beginning, not the end?
2. Name two ways the data-collection phase fails even when the files arrive on time.
3. What is the difference between a model metric and a business KPI? Give one of each for the farmer.

Answers, after you have tried:

1. Production is the first stream of genuinely new cases, and the world keeps moving after release. The rehearsal is over. Watching and retraining are the rest of the lifecycle.
2. The sample is the wrong population. The licence or the consent does not cover the use. The labels arrive too late to learn from. A column you trained on will not exist at the moment of prediction.
3. A model metric compares a prediction with a label. Recall of “rain” is a model metric. A KPI compares the system with the work. Irrigation cycles skipped without the crop drying is a KPI. You can improve one and hurt the other.

**Next class:** [03-how-a-model-learns.md](03-how-a-model-learns.md)
