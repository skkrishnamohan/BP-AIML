# Session notes — Collection, ingestion, lifecycle

*Module 2 Session 1.*

**Collection** is how a fact enters your control. **Ingestion** is how it lands in your platform without being silently edited. **Lifecycle** is the rest of the path until the data is retired. A model never sees “the weather.” It sees whatever this path kept.

---

## 🟢 Data lifecycle (loop)

```
Collect → Clean → Validate → Label → Version → Monitor (drift)
```

**Principle:** A model is only as good as its data; late defects cost ~10× more.

**What the loop is saying:** Every box can send you backward. Monitoring finds drift, which sends you back to collection. Validation fails, which sends you back to cleaning. The arrows are not a single school project you finish.

**Why late defects cost more:** A bad unit found in the raw file is a one-line fix. The same bad unit found after three models, a dashboard, and a farmer-facing API is a product incident. The “10×” figure is an engineering rule of thumb for the cost of fixing a defect one stage later, not a measured constant from the weather file. Use it as a warning to detect early.

**How you honor the principle:** Profile on arrival. Put checks in the pipeline, not in someone’s memory. Keep the raw file so you can repair a bad transform without asking the station to resend.

**Where each box is taught:** Cleaning and validation are the next file. Labels, versions, and search are the file after that. Drift monitoring is the trust file.

---

## 🟡 Collection techniques

| Source | Examples | AWS patterns |
|--------|----------|--------------|
| Sensors / IoT | Weather stations | IoT Core |
| APIs / DB exports | Partner CSV, RDS | AppFlow, DMS, Glue |
| Logs / events | Clickstream | Kinesis |
| Open / purchased | Registry of Open Data, Data Exchange | Check licence |
| Human | Surveys, annotations | Ground Truth, Label Studio |
| Synthetic | Fill gaps, privacy | Validate vs real data |

**What you owe every source:** where it came from, how often it updates, what licence applies, and who to call when a column changes.

| Source | What it is | The failure mode |
|--------|------------|------------------|
| **IoT Core** | AWS service that receives device messages | A replaced sensor sends a new unit and the old pipeline never notices |
| **RDS** | Relational Database Service, a managed SQL database | An export that includes columns the model must not see, or a query that hits production too hard |
| **AppFlow** | Managed transfer from SaaS apps into AWS | Easy to turn on, easy to forget as a data contract |
| **DMS** | Database Migration Service | Good for ongoing replication; schema changes still break downstream jobs |
| **Glue** | Serverless jobs and the catalog | A job that “succeeds” while writing an empty partition |
| **Kinesis** | Streaming buffer | Records you never replay after a bug in the consumer |
| **Clickstream** | The sequence of clicks or events | Logs that were designed for debugging, not for labels, with no consent review |
| **Registry of Open Data / Data Exchange** | Catalogs of public or paid datasets | A licence that allows study and forbids commercial models |
| **Ground Truth** | SageMaker’s managed labeling | You still write the instructions; the service does not know your edge cases |
| **Label Studio** | An open-source labeling app | You operate it |
| **Synthetic data** | Artificial rows or images made to fill a gap or avoid using real people | Models that learn the generator’s habits and fail on real stations |

**Why synthetic data is last:** It is useful when real examples are rare or sensitive. It is not ground truth. Always score the model on real held-out data.

---

## 🟡 Batch vs streaming

| | Batch | Streaming |
|---|-------|-----------|
| **Pattern** | Daily CSV drop | Sensors every minute |
| **Latency** | Hours | Seconds |
| **AWS** | Scheduled Glue, DMS | IoT Core + Firehose |
| **Landing** | S3 **raw** zone (immutable) | S3 **raw** zone (immutable) |

**Never edit raw** — it’s your audit trail. Clean copies go to `clean/` and `curated/`.

**What batch means:** Data accumulates and is processed as a set. A nightly file of station readings is batch. The decision latency is hours, which is acceptable if the farmer needs the score at 6 a.m.

**What streaming means:** Each event is handled soon after it happens. A station that reports every minute, and an alert that must fire this minute, is a stream. **Firehose** batches those events into files on S3 so later tools can read objects rather than a never-ending socket.

**Why both land in immutable raw:** The first stored copy is the evidence of what arrived. If cleaning has a bug, you rerun it from raw. If you “fixed” the raw object in place, you destroyed the evidence and cannot tell a sensor fault from a pipeline fault.

**Immutable** means you do not overwrite. You write a new object or a new version. S3 versioning, covered later, is the mechanism.

**Where people mix them up:** Using a stream because it sounds modern, then waiting a day to act. Or using batch for fraud that must be decided in 15 ms. Match velocity to the decision from the scoping notes.

---

## 🟡 Lake zones (S3 convention)

| Zone | Contents | Who writes |
|------|----------|------------|
| `raw/` | As received | Ingestion |
| `clean/` | Validated, typed Parquet/CSV | Pipelines + DQ rules |
| `curated/` | Feature-ready, versioned | ML teams |

**What a data lake is:** A store that keeps data in files, often S3, in its original and refined forms, and lets many tools read it. A **lakehouse** adds table management and SQL on top of those files (Glue Catalog plus Athena in this course).

**Why three zones exist:**

| Zone | What you are allowed to assume | What you must not do |
|------|--------------------------------|----------------------|
| `raw/` | “This is what the source sent.” | Train on it, or edit it to hide a defect |
| `clean/` | Types are parsed, duplicates handled, validation has run | Add business-specific features that only one model wants |
| `curated/` | The feature table matches the contract for a decision | Silently overwrite last month’s version |

**Parquet** is a columnar file format. It stores each column together and compresses well. CSV stores rows as text. The clean zone often converts to Parquet because later scans are cheaper. The idea is explained again in the scale notes; you only need the meaning here.

**Who writes curated:** The notes say ML teams because feature choices are model choices. In many companies a data team writes curated and ML consumes it. The course’s point is the handoff, not the org chart. Someone owns the contract.

---

## 🟡 Text data treatment

| Step | Purpose |
|------|---------|
| Filter | Language, length, spam |
| Redact | PII (e.g. Comprehend) |
| Normalise | Case, whitespace, encoding |
| Split | Tokens/chunks for LLM/RAG |
| Deduplicate | Exact & near-dup |
| Embed | Vectors for search |

Different downstream tasks need different treatment (classifier vs raw LLM).

**What each step is protecting:**

| Step | What | Why | Where it can go wrong |
|------|------|-----|------------------------|
| **Filter** | Drop the wrong language, empty notes, and spam | A classifier learns the junk you leave in | Dropping rare but real severe-weather bulletins because they are short |
| **Redact** | Remove emails, phone numbers, names | Bulletins and logs often contain PII you do not need for rain | A redaction miss that ships into a prompt |
| **Comprehend** | AWS natural-language API that can detect entities such as names | A ready-made detector, not a policy. You still decide what to remove. | Trusting it as complete |
| **Normalise** | Make “Rain” and “rain” and stray spaces comparable | Otherwise duplicates and string rules miss | Lowercasing a station code you needed to keep distinct, if you apply it to the wrong column |
| **Split** | Break long text into chunks or tokens | Models and embedding indexes have length limits | Cutting a negation (“no rain”) away from the word “rain” |
| **Deduplicate** | Remove exact copies and near copies | Duplicates inflate accuracy and leak across train and test if you split after | Deleting a repeated warning that was genuinely issued twice |
| **Embed** | Convert text to a vector | Enables “find similar bulletins” | Embedding text you would not be allowed to search later |

**Why the downstream task changes the treatment:** A classifier may want normalized, deduplicated, labeled sentences. An LLM that must quote the forecaster should see the original wording, with PII still removed. Keep a raw text copy and a task-specific copy.

---

## 🟡 Time-series rules (WeatherWise)

1. **Sort by time**; store **UTC**
2. **Resample** to regular spacing; mark gaps
3. **Seasonality features** — hour, day-of-year
4. **Lag features** — yesterday’s pressure
5. **No leakage** — future values must not enter features or scalers
6. **Chronological split** — train on past, test on future (**not random shuffle**)

**UTC** means **Coordinated Universal Time**, the timezone-free clock. Store timestamps in UTC and convert to local time only for display. Mixing IST timestamps that have no offset with UTC timestamps looks like a small cleaning issue and shifts every “tomorrow.”

**What resample means:** Stations do not all report on the same second. Resampling buckets them onto a regular grid, such as one row per station per hour. A missing hour stays missing and is marked. Filling it silently with a copy of the previous hour hides an outage.

**Seasonality:** Weather repeats daily and yearly. Hour of day and day of year let a tree or a linear model express “afternoons are warmer” without a special architecture.

**Lag feature:** A column that contains an earlier value, such as pressure 24 hours ago. Lags are legal because they are known before the decision. A **lead** (tomorrow’s pressure) is not legal when predicting tomorrow.

**Why random shuffle is forbidden:** If you shuffle, Tuesday’s row can land in training and Monday’s row in test. The model studies the future to predict the past. The test score is fiction. Split by time: train on history, test on the latest period, and do not fit scalers using the test period.

**How leakage hides in time series:** A “daily mean” computed with the whole day, including hours after the decision. A target `rain_tomorrow` built from a column that already peeked. A gap filled using the next valid reading (that is future information). Interpolation must only use the past, or it must be treated as a feature that would be computable at that time.

---

## 🟡 Data augmentation & synthetic

- **Images:** rotate, flip, brightness (labels must stay valid)
- **Tabular synthetic:** SDV-style generators
- **Text synthetic:** LLM-generated examples — **human spot-check**
- **Synthetic ≠ truth** — always evaluate on real held-out data

**Augmentation** means creating a new training example by a change that should not change the label. Flipping a radar image horizontally might be fine. Flipping it in a way that moves a storm onto a different city is not. The label must stay true after the change.

**SDV** means **Synthetic Data Vault**, a family of tools that learn a table’s distribution and sample new rows. The rows are plausible, not observed. They can fill a privacy or class-balance need. They also reproduce whatever bias was in the real table.

**Why humans spot-check LLM text:** A generated bulletin can say “no rain” in wording your labeler marks as rain, or it can invent a station. Synthetic text that was never reviewed becomes confident garbage with a label.

**Where the evaluation rule applies:** The test set is real. If you also test on synthetic data, you are partly grading the generator.

---

## 🟡 safetensors vs pickle

| | Pickle | safetensors |
|---|--------|-------------|
| Risk | Can execute code on load | Tensors + metadata only |
| Use | Legacy hubs | Preferred for sharing models |

Treat model files like **executables**.

**What pickle is:** Python’s built-in way to turn an object into bytes. Loading a pickle can run arbitrary code that was saved inside it. A model file from an untrusted URL can be a program, not only numbers.

**What safetensors is:** A format that stores tensors (the numeric arrays) and a small header. Loading it does not execute Python. That is why it is safer to share.

**Why “treat models like executables”:** Whoever can change the file can change the decisions, or worse, run code on the machine that loads it. Store model files in the same kind of controlled bucket as code artifacts, with versions and access control.

**Where this shows up:** Downloading a “helpful” preprocessor from a public hub, or emailing a `.pkl` between teammates. Prefer safetensors for weights, and job code that is reviewed like application code.

---

## 🟡 Annotation overview (preview Session 2)

- **CVAT** — vision/video; model-assisted boxes
- **Label Studio** — flexible, self-hosted
- **Ground Truth** — managed AWS workforce

**Model-assisted labeling:** model proposes → human corrects → faster than draw-from-scratch.

**Annotation** means a human (or a program with review) attaching the label: a class, a bounding box, a span of text.

| Tool | What it is best at | What you still own |
|------|--------------------|--------------------|
| **CVAT** | Computer Vision Annotation Tool. Boxes and tracks on images and video. | The label guide and the review |
| **Label Studio** | Many data types in one open-source app | Hosting, access, and the schema |
| **Ground Truth** | A managed workforce inside AWS | Instructions, gold questions, and quality checks |

**Why model-assisted labeling helps:** Drawing every radar box from scratch is slow. A model proposes a box, and the human fixes it. The risk is that humans accept a plausible wrong box. Keep a sample that is labeled from scratch so the model’s mistakes are visible.

---

## Checkpoint

Why is chronological split mandatory for rain prediction?

**Answer:** Rain is a time series. A random split puts future weather into training and past weather into test, so the model cheats by learning from later days. The live system can only see the past when it predicts tomorrow. Train on earlier dates, test on later dates, and fit every scaler only inside the training period.

**Next:** [03-cleaning-and-validation-weatherwise.md](03-cleaning-and-validation-weatherwise.md)
