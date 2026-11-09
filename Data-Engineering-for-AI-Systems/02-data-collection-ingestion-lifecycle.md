# Class notes — How data arrives, and why we do not edit the original

*Module 2, Session 1, continued.*

Scoping told us what decision we are allowed to support. This class is about the path a fact takes from a station, a database, or a person into a table a model may later read. **Collection** is how the fact enters your control. **Ingestion** is how it lands in the platform without being silently rewritten. The **lifecycle** is everything after that, until you are required to delete it.

The model never sees “the weather.” It sees whatever this path kept.

---

## 🟢 The loop, and the 10× warning

```
Collect → Clean → Validate → Label → Version → Monitor for drift
```

Any box can send you backward. Monitoring finds a shift and sends you back to collection. Validation fails and sends you back to cleaning. This is not a school project you finish.

The principle on the slide: a model is only as good as its data, and a defect found one stage later costs on the order of ten times more. The ten is a rule of thumb, not a measurement we took on the weather file. Use it as a reason to detect early. A bad unit found in the raw file is a one-line repair. The same bad unit found after three models, a dashboard, and a farmer-facing service is an incident. Profile on arrival. Put the checks in the pipeline. Keep the raw file so you can redo a bad transform without asking the station to send the day again.

Cleaning and validation are the next class. Labels, versions, and search are the class after that. Drift is the trust class.

---

## 🟡 Where rows come from, and the way each source fails

Every source owes you four facts: where it came from, how often it updates, what licence applies, and who to call when a column changes meaning.

**Sensors, through IoT Core.** A weather station publishes a message. AWS IoT Core receives device messages. The failure is a replaced sensor that quietly starts sending a new unit. Nothing crashes. The column name still says Celsius.

**Databases and partner files.** **RDS** is the Relational Database Service, a managed SQL database. **AppFlow** moves data from software-as-a-service products into AWS. **DMS** is the Database Migration Service, used for ongoing replication. **Glue** runs serverless jobs and holds the catalog. The failure mode is an export that includes a column the model must not see, or a job that “succeeds” while writing an empty partition, or a schema change that breaks the reader on Tuesday.

**Logs and clickstreams, through Kinesis.** A clickstream is the sequence of events. Kinesis is the buffer that holds them. Logs were often designed for debugging a service, not for training a model, and they may contain PII nobody reviewed. The failure is a bug in the consumer and no way to replay the day.

**Open and purchased data.** The Registry of Open Data and AWS Data Exchange are catalogs. The failure is a licence that allows you to study the file and forbids you to ship a commercial model. Read the licence before you celebrate the download.

**People.** Surveys and annotations. **SageMaker Ground Truth** is the managed labeling workforce. **Label Studio** is the open-source app you host yourself. The failure is a guide that two honest annotators read differently. That is a whole class of its own.

**Synthetic data.** Artificial rows or images, used to fill a gap or to avoid using real people. The failure is a model that learned the habits of the generator and fails on a real station. Synthetic data is not ground truth. The test set stays real.

---

## 🟡 Batch and streaming, and the zone both of them land in

| | Batch | Streaming |
|--|-------|-----------|
| The picture | A CSV arrives once a day | A sensor speaks every minute |
| How late the decision may be | Hours | Seconds |
| The AWS shape | A scheduled Glue job, or DMS | IoT Core into Firehose |
| Where the first copy lives | The raw zone, never edited | The raw zone, never edited |

**Batch** means the data accumulates and is processed as a set. A nightly file of station readings is batch. Hours of latency are acceptable if the farmer needs the score at 6 a.m.

**Streaming** means each event is handled soon after it happens. An alert that must fire this minute is a stream. **Firehose** collects those events into files on S3, so the rest of the tools can read objects rather than hold a socket open forever.

Both land in **raw**, and raw is **immutable**. Immutable means you do not overwrite the evidence. You write a new object or a new version. If cleaning has a bug, you rerun it from raw. If you “fixed” the raw object in place, you have destroyed the ability to tell a sensor fault from a pipeline fault. Clean copies go to `clean/`. Feature-ready copies go to `curated/`.

The design mistake is using a stream because it sounds modern, and then waiting a day to act. Or using a nightly file for a fraud decision that has 15 milliseconds. Velocity follows the decision from the scoping class, not the novelty of the service.

---

## 🟡 The three zones, said as permissions

A **data lake** is a store of files, here on S3, in both original and refined forms, that many tools can read. A **lakehouse** adds a catalog and SQL on top of those files. In this course that is the Glue Catalog plus Athena.

| Zone | What you may assume | What you may not do |
|------|---------------------|---------------------|
| `raw/` | “This is what the source sent.” | Train on it, or edit it to hide a defect |
| `clean/` | Types are parsed, duplicates are handled, validation has run | Hide a feature that only one model wants |
| `curated/` | The feature table matches the contract, and it is versioned | Silently overwrite last month, because last month’s model still needs to be explainable |

**Parquet** is a columnar file format. Each column is stored together and compresses well. CSV stores a row as a line of text. The clean zone often converts to Parquet because later queries read less. The scale class explains the bill. For now, Parquet means “the efficient copy,” not “the original.”

Who writes curated? The notes say the ML teams, because feature choices are model choices. In many companies a data team writes it and ML consumes it. The lesson is the handoff, not the org chart. Someone owns the contract, and training does not get to wander into raw.

---

## 🟡 What we do to text before it is allowed to train or to be searched

Text is not “just a column.” The bulletin class needs a sequence of decisions.

**Filter.** Drop the wrong language, the empty notes, and the spam. The risk is dropping a short but real severe-weather bulletin because a length rule was lazy.

**Redact.** Remove emails, phone numbers, and names. **Amazon Comprehend** is a natural-language API that can detect entities such as names. It is a detector, not a policy. You still decide what must be removed, and you still assume it will miss some.

**Normalise.** Make “Rain” and “rain” and stray spaces comparable, so duplicate detection and string rules work. Do not lowercase a station code if the case is meaningful. Apply normalisation to the column that needs it, not to the whole row by reflex.

**Split.** Break long text into tokens or chunks. Models and indexes have length limits. The risk, which we will meet as a labeling bug, is cutting the word “no” away from the word “rain.”

**Deduplicate.** Remove exact copies and near copies. Duplicates inflate accuracy, and if you split into train and test after duplicating, the same bulletin sits on both sides. The risk is deleting a warning that was genuinely issued twice. Exact duplicates and true repeated events are different. Keep a timestamp so you can tell them apart.

**Embed.** Turn the text into a vector so similar bulletins can be found. Do not embed text you would not be allowed to search later. The vector is a copy of the information.

Keep a raw text copy and a task-specific copy. A classifier wants normalised, deduplicated, labeled sentences. A model that must quote the forecaster should see the original wording, with PII still removed.

---

## 🟡 Six rules for weather in time

This is the section the rain model lives or dies on.

**1. Sort by time, and store UTC.** **UTC** is Coordinated Universal Time, the clock without a local timezone. Store timestamps in UTC. Convert to local time only when a person looks at the screen. A file that mixes “5:30 p.m. in India, no offset stored” with UTC will shift every “tomorrow” by several hours, and the shift will look like a small cleaning nuisance.

**2. Resample onto a regular grid, and mark the gaps.** Stations do not all speak on the same second. Resampling buckets them, for example one row per station per hour. A missing hour stays missing and is marked. Copying the previous hour into the gap hides an outage. Sometimes “the sensor was down” is the signal.

**3. Seasonality features.** Hour of day and day of year let a tree express “afternoons are warmer” and “June behaves like June” without a special architecture.

**4. Lag features.** A lag is an earlier value, such as yesterday’s pressure. Lags are legal because they are known before the decision. A **lead**, tomorrow’s pressure, is not legal when the decision is about tomorrow.

**5. No leakage into features or scalers.** A daily mean that includes hours after the decision is future information. Filling a gap with the next valid reading is future information. Fitting the scaler on the whole period, including the test months, is future information of a quieter kind.

**6. Split by time.** Train on the past. Test on a later period. Do not shuffle. If you shuffle, Tuesday can land in training and Monday in the test. The model studies the future to predict the past. The score is fiction. The live system, on Monday morning, cannot see Tuesday.

Do the check by hand on one station. Print three consecutive days. Confirm that Monday’s features contain Sunday and not Tuesday, and that the label “did it rain on Tuesday?” is the label, not a feature.

---

## 🟡 Augmentation, synthetic data, and model files that execute

**Augmentation** creates a new training example by a change that must not change the label. Rotating a photo or changing its brightness can be fine. Flipping a radar image so that a storm moves onto another city is not. The label has to remain true after the change, or you have manufactured a lie.

**SDV**, the Synthetic Data Vault, is a family of tools that learn a table and sample new rows. The rows are plausible. They were not observed. They also reproduce whatever bias was in the real table. LLM-written bulletins need a human spot check, because the generated sentence can say “no rain” in wording your labeler marks as rain, or it can invent a station. The evaluation set is real. If you also score the model on synthetic data, you are partly grading the generator.

**Pickle** is Python’s way of turning an object into bytes. Loading a pickle can run code that was saved inside it. A model file from a link can be a program. **safetensors** stores numeric tensors and a small header, and loading it does not execute Python. Prefer it when you share weights. Treat every model file like an executable: controlled bucket, access control, versions. Emailing a `.pkl` across a team is how this lesson gets learned the hard way.

---

## 🟡 Labeling tools, only as a preview

**Annotation** means attaching the label: a class, a box around an object, a span of text. **CVAT**, the Computer Vision Annotation Tool, is built for boxes and tracks on images and video. Label Studio is flexible and self-hosted. Ground Truth is the managed AWS workforce. In all three, you still own the guide.

**Model-assisted labeling** means the model proposes a box or a class and a person corrects it. It is faster than drawing from scratch. The risk is that people accept a plausible wrong proposal. Keep a sample that is labeled from scratch, so the model’s mistakes stay visible. The next class turns this preview into a workflow with a number for agreement.

---

## 🟡 Monday morning at one station, so the rules are not abstract

Station S01 reports hourly. On Sunday the link fails from 18:00 to 23:00. Those five hours arrive on Monday at 07:00, after the farmer’s 06:00 decision has already been made.

Event time is Sunday evening. Arrival time is Monday morning. If the pipeline uses arrival time as if it were weather time, Monday morning looks like a burst of Sunday’s climate and Sunday evening looks like the station did not exist. Seasonality features will be wrong because the hour is wrong. If the 06:00 batch closed the Sunday partition with no lateness window, the late hours are dropped and Sunday’s mean is biased toward the hours that happened to arrive. If the batch rewrites Sunday in place, last week’s model card now points at a file whose contents changed. The lateness window plus a new version is the grown-up behavior: Sunday’s curated partition stays the one the Monday score used, and a later version may include the late hours for the next training run, clearly labeled.

The same morning, someone shuffles the last 90 days to “make a random test.” Sunday’s late rows, once they are in the table, can train a model that is scored on Saturday. The live system on Saturday could not see Sunday. The score is a rehearsal of a power the product will not have. Chronological split is the repair. It is not a preference for tidiness.

Raw stays the Sunday messages as they arrived, including the late ones, with their arrival timestamps. Cleaning does not edit those objects. It writes a new version in clean. If the hour parsing is wrong, you rerun from raw. If you had “fixed” the raw messages, the rerun would preserve the mistake and call it the source.

## Checkpoint

Why is a random shuffle an invalid test for rain prediction? Describe one specific way the future would leak into the past.

Because the decision on Monday morning can only see Sunday and earlier. A shuffle can place Tuesday’s row in the training set and Monday’s row in the test set, so the model is examined on a day it has already, indirectly, studied past. The same leak happens if a scaler is fit on the whole year, including the test months, or if a gap on Monday is filled using Tuesday’s reading.

**Next class:** [03-cleaning-and-validation-weatherwise.md](03-cleaning-and-validation-weatherwise.md)
