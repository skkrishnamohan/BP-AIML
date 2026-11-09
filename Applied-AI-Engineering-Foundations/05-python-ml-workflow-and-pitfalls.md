# Session notes — Python ML workflow (NumPy → sklearn → production habits)

*Based on: Student Guide phases 18–20, Tools reading, insurance coding PDF.*

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

---

## 🟡 Parameters vs hyperparameters

| | Parameters | Hyperparameters |
|---|------------|-----------------|
| **Examples** | Learned weights | `max_iter`, tree depth, learning rate |
| **Set when?** | During `.fit()` | Before training |

---

## 🔴 Critical pitfalls

| Pitfall | What goes wrong |
|---------|------------------|
| **Data leakage** | Test info in training → fake high scores |
| **Leakage via scaling** | Fit scaler on full dataset |
| **Repeated test tuning** | Test set becomes part of training |
| **Wrong metric** | Optimizing accuracy on imbalanced data |

---

## 🟡 Regression vs classification (sklearn)

| | Regression | Classification |
|---|------------|----------------|
| **Goal** | Number | Category |
| **Models** | LinearRegression, Ridge | LogisticRegression, RandomForest |
| **Metrics** | RMSE, R² | Accuracy, P/R/F1 |

**Threshold:** Default 0.5 for probability is not sacred — tune for business cost.

---

## 🟢 Reproducibility

- `random_state=42` (or any fixed seed)
- Same split, same preprocessing pipeline
- Document versions of data (Module 2: DVC/S3)

---

## Checkpoint

Why is `fit_transform` on all data before split a leakage bug?

**Next:** [06-genai-prompts-responsible-ai.md](06-genai-prompts-responsible-ai.md)
