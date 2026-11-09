# Session notes — Python ML workflow (NumPy → sklearn → production habits)

*Based on: Student Guide phases 18–20, Tools reading, insurance coding PDF.*

This file turns the previous theory into a repeatable code path. The goal is not to memorize every function. The goal is to know **what** each tool owns, **why** the order of operations matters, and **where** leakage hides in ordinary-looking code.

---

## 🟢 Tool stack (learn in this order)

| Tool | Role |
|------|------|
| **Python 3.9+** | Language |
| **venv / conda** | Isolated environments |
| **NumPy** | Fast arrays, math |
| **Pandas** | Tables, CSV, cleaning |
| **Matplotlib / Seaborn** | Plots to debug data |
| **scikit-learn** | Classical ML + preprocessing |
| **Jupyter** | Experiments |
| **Git** | Version code |

Later: PyTorch/TensorFlow, MLflow, Docker, FastAPI, monitoring (Module 2+ / MLOps).

### What each tool is for, and what it is not

| Tool | Full name or meaning | What it does | What people mistakenly use it for |
|------|----------------------|--------------|-----------------------------------|
| **Python** | The language | Glue: load data, call libraries, serve results | A substitute for understanding the split and the metric |
| **venv** | Virtual environment | A private set of package versions for one project | Optional decoration. Without it, one project’s upgrade breaks another. |
| **conda** | A package and environment manager | Same job as venv, plus non-Python libraries | Required for every lab. Either tool is fine if the environment is recorded. |
| **NumPy** | Numerical Python | Arrays and vectorized math, the thing neural-net code sits on | A spreadsheet. You do not clean messy CSV column names here. |
| **Pandas** | Panel data library | Tables (`DataFrame`), CSV, missing values, group-bys | Training the model. Pandas prepares rows; sklearn learns weights. |
| **CSV** | Comma-separated values | A plain-text table | A safe production store. Fine for class labs; Module 2 moves to Parquet and S3. |
| **Matplotlib / Seaborn** | Plotting libraries | See distributions, missingness, and errors | Proof the model works. A pretty chart is not a test metric. |
| **scikit-learn** | The classical ML library, imported as `sklearn` | Splits, preprocessing, models, metrics | Deep learning or LLM training. |
| **Jupyter** | Notebook interface | Run a cell, look at the table, change one thing | A production deploy. Notebooks hide state. Pipelines rerun from the top. |
| **Git** | Version control | History of code | A place to dump a 2 GB dataset. Data versions belong to DVC or object storage later. |

**Why this order:** You cannot interpret a model you cannot load, and you cannot trust a model whose tables you have never plotted. Algorithms come after you can open the file and see a surprising value.

**Where the later tools fit:** **PyTorch** and **TensorFlow** train neural networks. **MLflow** records experiments. **Docker** packages the environment so another machine runs the same code. **FastAPI** serves predictions over HTTP. You do not need them to finish Module 1, and you should know their names so the roadmap is not a surprise.

---

## 🟢 Core sklearn pattern

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

**What `max_iter=2000` is:** The maximum number of optimization steps. It is a **hyperparameter**, not a learned weight. If the solver stops early you see a convergence warning. Raising it lets the fit finish. It does not make a wrong problem right.

**Why `fit` then `predict` are separate:** `fit` is allowed to read `y_train`. `predict` must not read `y_test`. Keeping them as two calls makes that wall visible.

**How to read the prints:** Accuracy is one number. The confusion matrix shows which species or which class was confused. The classification report gives precision, recall, and F1 per class, plus **support**, which means how many true rows of that class were in the test set. A class with tiny support can show a wild precision by chance.

**Where this snippet is incomplete for real tables:** It assumes `X` is already numeric. Cities, missing values, and income scales need a pipeline, below.

---

## 🟡 Preprocessing (insurance / have_insurance lab)

From your coding reading material, the **story** is:

1. Load CSV with `pandas`
2. Select features: age, income, education
3. Scale income (÷ 1000) so magnitudes match
4. **One-hot** education (`get_dummies`, `drop_first=True` avoids redundant columns)
5. Add **bias column** of ones for matrix math (if coding from scratch)
6. Train logistic regression with **gradient descent** OR use sklearn

**Why drop_first?** Prevents “dummy variable trap” (columns sum to 1).

### What the insurance story is teaching

The target `have_insurance` is binary. The features are a mix of a number (age, income) and a category (education). The lab exists so you see the same logistic regression from Module 1’s neuron, either written as a loop or called from sklearn.

**Dividing income by 1000** is a simple scale so income and age are similar sizes. A z-score is the more general version. Both are preprocessing, and both must be defined from training data only. Dividing by a constant you chose (1000) is safe. Computing a mean on the full file is not.

**`get_dummies`:** Pandas’ one-hot encoder. Education levels become columns. **`drop_first=True`** drops one level so the remaining columns are comparisons against that reference level. If Bachelor, Master, and PhD columns are all 0, the person is the dropped level.

**Dummy variable trap:** If the columns sum to 1 for every row, they are a perfect copy of a column of ones (the bias). The optimizer then cannot tell bias and education apart. Coefficients become unstable. Dropping one column removes the perfect copy.

**Bias column of ones:** When you code the math yourself, \(z = w \cdot x + b\) can be written as one dot product by appending a 1 to `x` and treating `b` as just another weight. Sklearn already includes the intercept. Adding a column of ones *and* using sklearn’s intercept double-counts it. Know which implementation you are in.

**Where the two implementations should agree:** Same split, same encoding, enough iterations. The weights will match in direction even if the solver is not identical. If handmade gradient descent diverges, the learning rate is too big or the features are unscaled.

---

## 🔴 Professional pipeline pattern

**Never** scale or encode **before** split.

Correct pattern:

1. Split train/test
2. `fit()` imputer/scaler **on train only**
3. `transform()` train and test
4. Bundle in **Pipeline** + **ColumnTransformer**

**Must-know checklist from student guide:**

- `train_test_split`
- `Pipeline`, `ColumnTransformer`
- `SimpleImputer`, `OneHotEncoder`, `StandardScaler`
- `cross_val_score`
- `GridSearchCV` / `RandomizedSearchCV`
- `classification_report`, `confusion_matrix`

### What these objects protect you from

| Object | What it is | Why it exists |
|--------|------------|---------------|
| **`SimpleImputer`** | Fills missing values with a statistic such as the median | A blank is not a number the model can multiply. The median must come from train. |
| **`StandardScaler`** | Z-score: subtract mean, divide by standard deviation | Stops large-unit columns from dominating |
| **`OneHotEncoder`** | Sklearn’s version of dummy columns | Belongs in the pipeline so the category list is learned on train. An unseen city at test time has a defined policy (`handle_unknown`) |
| **`ColumnTransformer`** | Applies different steps to different columns | Scale income; one-hot city; leave an already-encoded flag alone |
| **`Pipeline`** | A chain of transform → transform → model | `fit` runs every step in order on train. `predict` runs the same transforms with frozen statistics, then the model. You cannot forget and scale the test set with the test mean. |
| **`cross_val_score`** | Repeats fit/score across folds | A more stable estimate, and it refits the whole pipeline per fold if you pass the pipeline |
| **`GridSearchCV`** | Tries a grid of hyperparameters with cross-validation | Picks settings using validation folds, not the final test set |
| **`RandomizedSearchCV`** | Tries a random subset of that grid | Same idea when the grid is too large to finish |

**`fit` vs `transform` vs `fit_transform`:** `fit` learns statistics. `transform` applies them. `fit_transform` does both, and it is correct on the training fold only. Calling `fit_transform` on the full dataset, or on the test set, refits the statistics in the wrong place.

**Where the bug looks innocent:**

```python
X_scaled = scaler.fit_transform(X)          # learns mean from future test rows
X_train, X_test = train_test_split(X_scaled)
```

The test rows helped decide the mean and variance. The model has peeked at the exam. Do this instead: split first, `fit` on train, `transform` both.

---

## 🟡 Parameters vs hyperparameters

| | Parameters | Hyperparameters |
|---|------------|-----------------|
| **Examples** | Learned weights | `max_iter`, tree depth, learning rate |
| **Set when?** | During `.fit()` | Before training |

**What a parameter is:** A number inside the model that gradient descent (or another solver) changes. You do not type the final weights.

**What a hyperparameter is:** A choice about the learning procedure or the model shape, made before `fit`. Tree depth, regularization strength `C` in logistic regression, and learning rate are hyperparameters.

**Why the distinction matters:** Searching hyperparameters on the test set turns the test set into training data. `GridSearchCV` keeps that search inside training folds. The final test score happens after the search is frozen.

**`C` in logistic regression:** Inverse of regularization strength. Smaller `C` means a stronger penalty on large weights, a simpler model, and less overfitting. The letter is historical. Remember the direction, not a magic value.

---

## 🔴 Critical pitfalls

| Pitfall | What goes wrong |
|---------|------------------|
| **Data leakage** | Test info in training → fake high scores |
| **Leakage via scaling** | Fit scaler on full dataset |
| **Repeated test tuning** | Test set becomes part of training |
| **Wrong metric** | Optimizing accuracy on imbalanced data |

**How repeated test tuning happens:** You train, look at the test accuracy, add a feature, look again, change depth, look again. After twenty looks, you have selected the model that got lucky on these particular test rows. Report that number as exploration, then evaluate once on a set you did not use to choose.

**Where wrong-metric leakage of attention happens:** The code is honest and the conclusion is not. Accuracy 0.99 on 1% fraud will pass a unit test you wrote against accuracy. Put the business metric in the evaluation cell so the notebook cannot “pass” while catching nothing.

---

## 🟡 Regression vs classification (sklearn)

| | Regression | Classification |
|---|------------|----------------|
| **Goal** | Number | Category |
| **Models** | LinearRegression, Ridge | LogisticRegression, RandomForest |
| **Metrics** | RMSE, R² | Accuracy, P/R/F1 |

**Threshold:** Default 0.5 for probability is not sacred — tune for business cost.

**RMSE** means **root mean squared error**. Take the differences between predicted and actual numbers, square them (so negatives count), average them, and take the square root so the unit matches the target (rupees, minutes, megawatts). Big mistakes count extra because of the square.

**R²** (R-squared) is the fraction of variance in `y` that the model explains compared with predicting the average every time. 1 is a perfect fit. 0 is no better than the average. Negative means worse than predicting the average. A high R² on training data alone is not a result.

**Ridge** is linear regression plus a penalty on large weights. It is the regression cousin of regularized logistic regression.

**RandomForest** is an ensemble of decision trees, each trained on a random subset of rows and features. The trees vote. It handles non-linear patterns and is still a classical ML model.

**Why `LogisticRegression` is classification:** The name says regression because it models the log-odds, a continuous number. The decision you take from it is a class. Do not put it in the regression column when the target is yes/no.

**`predict_proba`:** Returns the probability. `predict` applies the 0.5 threshold. For fraud you often call `predict_proba` and apply your own cutoff.

---

## 🟢 Reproducibility

- `random_state=42` (or any fixed seed)
- Same split, same preprocessing pipeline
- Document versions of data (Module 2: DVC/S3)

**What reproducibility means:** Another person, or you next month, can rerun the training and get the same split, the same metrics, and the same kind of model. It does not mean the model is correct.

**Why a seed:** `train_test_split` shuffles. Without a seed the exam changes every run and you cannot tell a real improvement from a lucky shuffle.

**Where code-only versioning fails:** Git remembers the notebook. It does not remember which CSV was on disk. Module 2’s answer is to version the dataset (DVC, S3 versions) and record that identifier beside the Git commit.

---

## Checkpoint

Why is `fit_transform` on all data before split a leakage bug?

**Answer:** `fit_transform` computes statistics such as means, medians, and category lists from every row it is given. If those rows include the test set, the test set helps define the transform that is later applied to itself. The model indirectly sees the exam while studying. Split first. Fit on train. Transform test with the frozen training statistics.

**Next:** [06-genai-prompts-responsible-ai.md](06-genai-prompts-responsible-ai.md)
