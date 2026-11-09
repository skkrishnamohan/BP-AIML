# Class notes — The Python workflow, and the bugs that look like ordinary code

*Student guide phases 18–20, the tools reading, and the insurance coding PDF.*

You can now do one neuron by hand and you can compute precision on a hundred rows. This class puts that story into the libraries the labs actually run. The failure mode is not a wrong import. It is a correct import, called in an order that peeks at the exam.

---

## 🟢 The tools, in the order you should meet them

Learn them in this order because each one answers a question the next one assumes.

**Python 3.9 or newer** is the language that glues the rest together. It will not choose your split or your metric for you.

**venv** or **conda** is an isolated environment. A virtual environment is a private set of package versions for one project. Without it, upgrading scikit-learn for a new lab changes the old lab. Conda does the same job and also installs non-Python libraries. Either is fine. “It worked on my laptop” is the symptom of skipping both.

**NumPy** is Numerical Python. It holds arrays and does the vectorized math that a weight update is made of. You do not clean messy column names here.

**Pandas** holds a table, called a `DataFrame`. It reads a **CSV** (comma-separated values), finds missing entries, and selects columns. Pandas prepares rows. It does not learn weights. A CSV is fine for a class file and a poor production store. Module 2 moves the same table to Parquet on S3.

**Matplotlib** and **Seaborn** draw the pictures you use to debug data: a histogram that shows a spike at −999, a count of missing values. A pretty chart is not a test metric.

**scikit-learn**, imported as `sklearn`, is the classical ML library: the split, the preprocessing, the models, and the metrics. It is not a deep-learning library and it is not an LLM.

**Jupyter** is the notebook where you run one cell and look at the table. It is the right place to learn and a bad place to deploy, because a notebook remembers whatever you ran earlier, including a cell you have since deleted in your head.

**Git** versions the code. It is the wrong place for a two-gigabyte CSV. Data versions belong to DVC or object storage, which is Module 2.

Later names, so the roadmap is not a surprise. **PyTorch** and **TensorFlow** train neural networks. **MLflow** records experiments. **Docker** packages the environment so another machine runs the same code. **FastAPI** serves predictions over HTTP. You do not need them to finish this module. You need to know which problem each one is for, so you do not reach for FastAPI when the bug is leakage.

---

## 🟢 The ten-line pattern, narrated

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)

model = LogisticRegression(max_iter=2000)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

print(accuracy_score(y_test, y_pred))
print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
```

Read it as a story, not as a spell.

The split happens before any learning. `test_size=0.2` holds out a fifth. `stratify=y` keeps the class mix. `random_state=42` makes the shuffle repeatable.

`LogisticRegression` is a classifier even though the name says regression. It models the log-odds, which is a continuous number, and the decision you take is a class. Do not put it in the regression column of your notes when y is yes/no.

`max_iter=2000` is a hyperparameter: the maximum number of solver steps. If you see a convergence warning, the fit stopped early and raising this number lets it finish. It does not repair a wrong target.

`fit` is allowed to read `y_train`. `predict` must not read `y_test`. Two calls, so the wall is visible.

Then print three things. Accuracy is one number and, as of last class, it can lie. The matrix shows the cells. The classification report gives precision, recall, and F1 per class, plus support. This snippet still assumes X is already numeric. Cities, blanks, and income in rupees need the pipeline below. Do not pretend this snippet is the production pattern.

`predict_proba` is the call that returns the probability. `predict` applies 0.5. For fraud you usually want the probability and your own cutoff.

---

## 🟡 The insurance lab, as a story rather than a script

The target is `have_insurance`, yes or no. The features are age, income, and education. The lab exists so you either write the gradient step yourself or call sklearn and see that it is the same problem.

The honest sequence:

1. Load the CSV with pandas. Print the row count, the dtypes, and a few education values. Education is text until you encode it.
2. Separate X and y. If `have_insurance` is still a column of X, stop. That is leakage by copy-paste.
3. Split.
4. Scale income so it does not drown age. Dividing by 1000 is a constant you chose, so it does not peek at the test set. Computing a mean on the full file does peek. A z-score fit on train only is the general version.
5. One-hot encode education. In pandas, `get_dummies(..., drop_first=True)`.
6. If you are coding the math yourself, append a column of ones so the bias is just another weight. If you are calling sklearn, do not also add that column. Sklearn already fits an intercept. Doing both double-counts it.
7. Train, either with your loop or with `LogisticRegression`.
8. Evaluate on the test rows with a matrix, not only with accuracy.

**Why `drop_first=True`.** Suppose education has three levels and you keep all three dummy columns. On every row they add up to 1. A column of ones is exactly the bias term. The optimizer has two ways to write the same score, so the weights wander. That is the **dummy-variable trap**. Dropping one level makes the others “compared with the dropped level.” All zeros means the dropped level.

If your handmade loss explodes, the learning rate is too big or income is still in rupees. If the loss falls and the sklearn metrics land in a similar range on the same split, you have connected the board-work to the library. The weights will not match to the last decimal, because the solvers differ. The direction of the story should match.

Use log loss in the handmade loop, not squared error. Squared error is the natural pain for a price. Log loss is the pain for a probability that was confident and wrong. The formula is in the previous class notes. Print the loss after one epoch. It should usually fall. If it does not, do not add epochs. Fix the scale or the step size.

---

## 🔴 The professional order, and the bug written out

Never scale or encode before the split. The correct order is:

1. Split train and test.
2. `fit` the imputer and the scaler on the training rows only. Fit means “learn the statistic.”
3. `transform` both the training rows and the test rows with that frozen statistic.
4. Put the steps in a **Pipeline** so the order cannot be forgotten.

`fit_transform` does both, and it is legal on the training fold only. These two lines are the bug, even though they look tidy:

```python
X_scaled = scaler.fit_transform(X)           # the mean includes the future exam
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y, random_state=42)
```

Suppose five incomes: 10, 10, 10, 10, and 100. The mean of all five is 28. The mean of the first four, if those are the training rows, is 10. The test income of 100 has pulled the training mean from 10 up to 28. Every training row is now scaled with a number that knows about the exam. On a real file the damage is quieter and the accuracy moves the wrong way: up. That upward move is the smell of leakage.

The objects you are expected to recognize:

| Object | What it learns on fit | Why it has to be inside the pipeline |
|--------|----------------------|--------------------------------------|
| `SimpleImputer` | A fill value, often the median | A blank is not a number. The median of the test rows is not allowed to help. |
| `StandardScaler` | Mean and standard deviation | This is the z-score. Same rule. |
| `OneHotEncoder` | The list of categories | An unknown city at test time needs a policy (`handle_unknown`), not a crash and not a refit. |
| `ColumnTransformer` | Nothing by itself | It applies the scaler to income and the encoder to city, instead of scaling a column of city names. |
| `Pipeline` | The chain, in order | `fit` trains every step on train. `predict` replays the frozen steps and then the model. |

`cross_val_score` repeats that whole pipeline across folds. Pass it the pipeline, not the model alone, or the scaler will have been fit outside the fold.

`GridSearchCV` tries a grid of hyperparameters using cross-validation, so the choice of tree depth or of `C` is not made by peeking at the final test set. `RandomizedSearchCV` tries a random subset when the grid is too large. **`C`** in logistic regression is the inverse of regularization strength. Smaller `C` means a stronger penalty on large weights, a simpler model, less memorizing. Remember the direction. There is no magic value.

After the search is frozen, the test set is used once.

---

## 🟡 Parameters, regression metrics, and the other pitfalls

A **parameter** is learned during `fit`: the weights. A **hyperparameter** is chosen before `fit`: `max_iter`, tree depth, learning rate, `C`. Searching hyperparameters against the test score is the slow way to leak. You look, you tweak, you look again. After twenty looks the test rows have selected the model. Report that number as exploration. The number you publish comes from a set you did not use to choose.

| | Regression | Classification |
|--|------------|----------------|
| y | A number | A category |
| Models you will see | `LinearRegression`, `Ridge` | `LogisticRegression`, `RandomForest` |
| Metrics | RMSE, R² | Accuracy, precision, recall, F1 |

**RMSE** is the root mean squared error. Subtract predicted from actual, square the differences so that negative errors count, average them, and take the square root so the unit is rupees or minutes again. Squaring means one wild miss hurts more than several small ones.

**R²** is the fraction of the variation in y that you explain beyond simply predicting the average every time. 1 is a perfect fit. 0 is no better than the average. Negative is worse than predicting the average. A high R² on the training rows is not a result.

**Ridge** is linear regression plus a penalty on large weights. **RandomForest** is many decision trees, each trained on a random subset of rows and features, voting together. It can represent bends that one logistic regression cannot, and it can still overfit.

The pitfalls, said as symptoms:

| What you did | What you will see |
|--------------|-------------------|
| Any form of leakage | A score that collapses when the leaked column is removed, or that collapses in production |
| Fit the scaler on the full file | A quietly optimistic test score |
| Tuned twenty times on the test set | A test score you cannot reproduce on the next month |
| Optimized accuracy on 1% fraud | A model that flags nobody and passes the unit test |

**Reproducibility** is the unglamorous close of the class. Fix the seed. Keep the preprocessing in a pipeline so the order is the code. Write down the data version, because Git remembers the notebook and does not remember which CSV was on the disk. Reproducible does not mean correct. It means the next person can rerun the same experiment and argue about the same number.

---

## Checkpoint

Explain, with the five incomes 10, 10, 10, 10, and 100, why `fit_transform` on all of them before the split has already used the test answer’s neighborhood. You do not need the word “leakage” in the first sentence. Say which number moved.

The number that moved is the mean, from 10 to 28, because the held-out 100 was allowed into the statistic that then scales the training rows. The training data is no longer independent of the exam.

**Next class:** [06-genai-prompts-responsible-ai.md](06-genai-prompts-responsible-ai.md)
