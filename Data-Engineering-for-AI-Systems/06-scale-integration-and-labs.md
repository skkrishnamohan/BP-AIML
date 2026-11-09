# Session notes — Scale, integration, capstone

*Module 2 Session 3 (part 7) + lab integrations.*

---

## 🟡 Scalability themes

| Topic | Idea |
|-------|------|
| **Parquet** | Columnar, compressed — less scan than CSV |
| **Partitions** | e.g. `year=2024/` — Athena reads less data |
| **Pipelines** | SageMaker Pipelines, Step Functions orchestrate repeatable runs |
| **Cost** | Athena ~$ per TB scanned — partition + column prune = money saved |

**Exam-style question from slides:** Why does Parquet + partitions cut Athena bills?  
→ **Less data scanned per query.**

---

## 🟡 Integration points

- **Glue Catalog** — single metadata layer for lake  
- **Athena** — SQL on S3 without servers  
- **SageMaker** — train, clarify, monitor, endpoints  
- **EventBridge** — schedule retraining / drift checks  
- **CloudWatch** — metrics + alarms (trust gate publishes here)

**Data engineering + MLOps handoff:** curated S3 path + schema contract + version ID → training job.

---

## 🟢 End-to-end WeatherWise pipeline (module capstone)

| Step | AWS / practice |
|------|----------------|
| 1 Collect | IoT/API → raw S3 |
| 2 Clean + validate | Notebook/Glue, DQ rules |
| 3 Label | Ground Truth / Label Studio / programmatic |
| 4 Store + version | S3 versioning, DVC, Catalog |
| 5 Bias + OOD | Clarify, custom trust gate, PSI |
| 6 Explain | SHAP, Clarify, model cards |
| 7 Scale + integrate | Pipelines, Athena, monitored endpoint |

By Session 3 you’ve touched **every box** at introductory depth.

---

## 🟡 Warm-up questions (from slides — self-test)

1. **Fahrenheit in Celsius column** — why fix units before range checks?  
2. Name **3 architecture decisions** from Session 1 (e.g. batch vs stream, RAG vs fine-tune, cloud strategy).  
3. Why is **safetensors** safer than pickle?

---

## 🟡 Exit ticket style Q&A

| Question | Short answer |
|----------|--------------|
| Remove `group` but bias remains? | **Proxy features** (postcode) |
| PSI 0.4 on humidity? | **Investigate drift**, don’t blindly retrain; check sensor/world change |
| Parquet partitions? | **Predicate pushdown** → less scan |

---

## 🔴 Module 2 mastery checklist

- [ ] Write Who/What/Why/Where for a project you care about  
- [ ] List 9 cleaning defect types and fix order reasoning  
- [ ] Explain kappa vs raw agreement  
- [ ] Draw raw → clean → curated S3 flow  
- [ ] Define DI and PSI thresholds from course  
- [ ] Describe trust gate: bias + drift + routing  
- [ ] Name three explainability levels (sample/feature/distribution)  

---

## What to study next (Module 3 preview)

**Model Engineering** — training pipelines, architecture choices, tuning beyond baselines.

Keep these notes open when replaying:

- `Weather_Data_Cleaning_Walkthrough_SageMaker.ipynb`  
- `Session3_Lab_Trust_Gate.ipynb`  
- `04a_Lab4a_Versioning_and_Athena.ipynb`  

**Back to Module 1 review:** [../Applied-AI-Engineering-Foundations/00-START-HERE.md](../Applied-AI-Engineering-Foundations/00-START-HERE.md)
