# Session notes — Case studies, quizzes, coding lab tie-in

*Consolidates Week 1 slides + coding reading.*

---

## 🟢 Fraud detection blueprint (features → label)

Each transaction row:

- **X:** amount, distance from home, time of day, …  
- **y:** 1 = confirmed fraud, 0 = legit  

Training = find weights that separate patterns. Inference = score new txn in milliseconds.

---

## 🟢 Quiz #1 answers (rules / ML / GenAI)

| Case | Answer |
|------|--------|
| GST 18% | Rules |
| UPI fraud shield | Predictive ML |
| RBI policy brief | GenAI |
| Password validator | Rules |
| Grid load forecast | Regression ML |

---

## 🟢 Quiz #2 answers (classification vs regression)

| Case | Answer |
|------|--------|
| Bengaluru flat price ₹ | Regression |
| Spam vs inbox | Binary classification |
| Normal / pre-diabetic / diabetic | Multi-class |
| Delivery ETA minutes | Regression |
| Churn yes/no | Binary classification |

---

## 🟡 Bank auto-loan inference story

1. New customer features scaled  
2. Plug into **frozen** weights from training  
3. Get P(buy) = 88%  
4. Business rules: ≥75% → pre-approved offer  

**Latency target in story:** < 10 ms for mobile banking.

---

## 🔴 Insurance dataset (class + homework)

**Features:** age, income, education → **y:** have_insurance  

Walkthrough covered:

- Log loss measures wrong confident predictions  
- Gradient updates weights toward less loss  

**Your coding PDF** implements this with pandas + manual gradient descent OR sklearn equivalent.

**Study tip:** Run the insurance notebook once line-by-line with print statements after each step.

---

## Module 1 “you are here” in program roadmap

1. **Applied AI Foundations** ← you completed concepts here  
2. **Data Engineering for AI** ← Module 2 notes  
3. Model Engineering  
4. AI Quality  
5. Platform & MLOps  
6. Responsible AI & Capstone  

---

## Module 1 mastery checklist

- [ ] Explain AI ⊃ ML ⊃ GenAI in one diagram  
- [ ] Choose rules vs ML vs GenAI for 3 scenarios  
- [ ] Write formulas for precision & recall from a confusion matrix  
- [ ] Describe train/test split and one leakage example  
- [ ] Sketch neuron → sigmoid → threshold  
- [ ] List 3 responsible AI pillars and what they mean in practice  

When most boxes are checked, start **Data Engineering** folder in order.

**Next module:** [../Data-Engineering-for-AI-Systems/00-START-HERE.md](../Data-Engineering-for-AI-Systems/00-START-HERE.md)
