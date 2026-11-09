# Session notes — Scale, integration, capstone

*Module 2 Session 3 (part 7) + lab integrations.*

Scale here means the same pipeline still works when the file is large and the query is asked often. Integration means the services agree on a dataset version. The capstone is the whole WeatherWise path in one picture, not a new algorithm.

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

### What, why, how, where

**Parquet.** A columnar file. CSV stores a row as one line of text, so reading three columns means reading every line. Parquet stores each column in its own compressed block and keeps statistics such as min and max. A query that needs humidity can skip the radar-id block and can skip row groups whose statistics prove they are irrelevant. **Why it matters:** Athena charges by bytes scanned, and training jobs shuffle less data. **Where:** the clean and curated zones, after you have finished the human-readable CSV lab.

**Partitions.** Folders that encode a column, such as `year=2024/month=05/`. The catalog knows that layout. A query with `WHERE year = 2024` does not open the other years. This is **partition pruning**. **Predicate pushdown** is the broader idea: apply the filter as early as possible, inside the scan, rather than after reading everything. **Why:** cost and time. **Where it fails:** partitioning by a column you rarely filter, or by a column so fine (every station-hour) that you create millions of tiny files. Tiny files cost more to list than you save in scans. Partition by something you always filter, often date.

**Pipelines.** A **pipeline** is a defined sequence of steps that can be rerun. **SageMaker Pipelines** chains data prep, training, evaluation, and registration. **Step Functions** is a general AWS workflow service that can include Glue jobs, gates, and approvals. **Why:** a notebook run by hand is not repeatable and has no record of which step failed. **How:** each step reads an explicit input version and writes an explicit output version. **Where:** the path from curated data to a registered model, including the trust gate.

**Cost.** The bill is a design constraint from the scoping notes. Scanning a terabyte to answer “mean humidity last May” is a modeling choice, not an unavoidable cloud tax. Select only the columns you need, filter on partition keys, and store curated tables as Parquet.

**Column prune** means the engine reads only the columns named in the query. It is the columnar twin of partition pruning.

---

## 🟡 Integration points

- **Glue Catalog** — single metadata layer for lake
- **Athena** — SQL on S3 without servers
- **SageMaker** — train, clarify, monitor, endpoints
- **EventBridge** — schedule retraining / drift checks
- **CloudWatch** — metrics + alarms (trust gate publishes here)

**Data engineering + MLOps handoff:** curated S3 path + schema contract + version ID → training job.

| Service | What it contributes | What breaks if it is missing |
|---------|---------------------|------------------------------|
| **Glue Catalog** | One dictionary of table names, columns, and S3 locations | Every team hard-codes a path and a schema, and they drift apart |
| **Athena** | Ad-hoc SQL for profiling and audit, no cluster to manage | Quality investigations become “download the CSV and hope it fits” |
| **SageMaker** | Training, Clarify bias and explanation reports, Model Monitor, and endpoints | Models stay in notebooks and never meet the gate |
| **EventBridge** | A clock and an event bus. It can start a drift check every morning or after a new curated version lands. | Checks run only when someone remembers |
| **CloudWatch** | Time-series metrics and alarms | A failed PSI sits in a JSON object nobody reads |

**What the handoff must contain:**

| Piece | Why the training job cannot guess it |
|-------|--------------------------------------|
| **Curated S3 path** | Raw and clean are not the contract for features |
| **Schema contract** | Column names, types, ranges, and which nulls are allowed. A renamed humidity column should fail the job, not silently train on an empty feature. |
| **Version ID** | The model card has to name the data it used. “Latest” changes under you during training. |

**How you know the handoff worked:** A second person can start the training job from the version id alone and get the same row count the data team published (about 5,466 in the lab).

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

**What each step is responsible for, in one line:**

| Step | Owns | Does not own |
|------|------|--------------|
| 1 Collect | Landing source bytes unchanged in `raw/` | Deciding the model family |
| 2 Clean + validate | Types, units, keys, ranges, and a failing gate | Fairness of historical loan labels |
| 3 Label | A defined `y`, guidelines, and agreement | The serving latency |
| 4 Store + version | Zones, catalog, and a version you can restore | Whether the metric matches the business |
| 5 Bias + OOD | Fit-for-use checks and quarantine | Writing the farmer a paragraph |
| 6 Explain | Why a row was scored, and a card a stranger can read | Collecting the next day’s sensors |
| 7 Scale + integrate | Repeatable, affordable, monitored runs | Inventing a new label definition quietly |

**Why the table is the capstone:** Module 2’s exam questions jump between rows. A Fahrenheit bug is row 2. A proxy that survives column deletion is row 5. An Athena bill is row 7. Being able to point at the row is the skill.

**Where the endpoint fits:** An **endpoint** is the live scoring service. WeatherWise’s 6 a.m. decision might instead be batch transform. Either way, monitoring sits after serving, and a bad PSI can stop promotion of the next version. Deployment is the beginning of the loop, as in Module 1.

---

## 🟡 Warm-up questions (from slides — self-test)

1. **Fahrenheit in Celsius column** — why fix units before range checks?
2. Name **3 architecture decisions** from Session 1 (e.g. batch vs stream, RAG vs fine-tune, cloud strategy).
3. Why is **safetensors** safer than pickle?

**Answers you should be able to say:**

1. A range check written for Celsius will delete obviously hot Fahrenheit values and keep Fahrenheit values that happen to sit inside the Celsius band. The file then looks valid and is still in the wrong unit. Convert first, then test the range, and log how many cells were converted.
2. Any three real decisions. Examples: nightly batch versus a per-minute stream, because the farmer acts at 6 a.m.; embeddings plus a classifier versus an LLM, because bulletins need a category before they need a paragraph; one AWS stack versus a portable open-source core, trading speed against lock-in; RAG versus fine-tuning, because the facts live in documents that change and should be cited; raw / clean / curated zones, because training must not read the audit copy.
3. Loading a pickle can run arbitrary Python stored in the file. Safetensors stores numeric tensors and metadata and does not execute code. A model file is an artifact you load into a process, so it has the risk profile of a program.

---

## 🟡 Exit ticket style Q&A

| Question | Short answer |
|----------|--------------|
| Remove `group` but bias remains? | **Proxy features** (postcode) |
| PSI 0.4 on humidity? | **Investigate drift**, don’t blindly retrain; check sensor/world change |
| Parquet partitions? | **Predicate pushdown** → less scan |

**Longer answers, so the short ones are not slogans:**

**Proxy.** Group B is still predictable from `postcode_score` (correlation about −0.61 in LoanLens). The historical labels are already uneven (disparate impact about 0.59). Deleting the group column leaves both the stand-in feature and the skewed teacher signal. The trust gate’s proxy check exists for this case.

**PSI 0.4.** That is above the 0.25 major-shift band. The inputs no longer look like training. Retraining immediately can bake in a broken sensor or a one-off storm. Check calibration, station, and time window first. If the world truly changed and the labels are real, then retrain on a declared new version and keep the old model card pointing at the old data.

**Parquet and partitions.** The engine skips files whose partition does not match the filter and reads only the columns the query names. Fewer bytes scanned means a smaller Athena bill and a faster question. The phrase on the slide is predicate pushdown.

---

## 🔴 Module 2 mastery checklist

- [ ] Write Who/What/Why/Where for a project you care about
- [ ] List 9 cleaning defect types and fix order reasoning
- [ ] Explain kappa vs raw agreement
- [ ] Draw raw → clean → curated S3 flow
- [ ] Define DI and PSI thresholds from course
- [ ] Describe trust gate: bias + drift + routing
- [ ] Name three explainability levels (sample/feature/distribution)

**What “checked” means for each box:**

| Box | You can say |
|-----|-------------|
| Who/What/Why/Where | The actor, the exact prediction, the business result, and batch versus API versus edge |
| Nine defects | Station case, date formats, duplicates, sentinels, hidden units, impossible ranges, missing values, target leakage, and a validation gate. Units before ranges. |
| Kappa | Percent agreement ignores chance. Kappa removes chance. Under about 0.6, fix the guide. |
| Zones | Raw is immutable intake, clean is validated, curated is the versioned feature table training is allowed to read |
| Thresholds | DI warning below 0.8. PSI stable below 0.10, watch to 0.25, major above 0.25. Proxy alarm at absolute correlation 0.50 in the lab. |
| Trust gate | Representation, disparate impact, and proxy before training; PSI for drift; pass to curated, fail to quarantine; alarm on CloudWatch |
| Three levels | Similar examples (sample), which inputs matter (feature), whether today matches training (distribution) |

---

## What to study next (Module 3 preview)

**Model Engineering** — training pipelines, architecture choices, tuning beyond baselines.

**What that module will assume you already know:** which target you are predicting, which features were legal at decision time, which data version you used, and which metric and fairness check the model must pass. Model engineering tunes a model that those choices made meaningful. It does not choose them for you.

Keep these notes open when replaying:

- `Weather_Data_Cleaning_Walkthrough_SageMaker.ipynb`
- `Session3_Lab_Trust_Gate.ipynb`
- `04a_Lab4a_Versioning_and_Athena.ipynb`

**Back to Module 1 review:** [../Applied-AI-Engineering-Foundations/00-START-HERE.md](../Applied-AI-Engineering-Foundations/00-START-HERE.md)
