# Class notes — Scale, the handoff, and the whole pipeline on one page

*Module 2, Session 3, the last part, plus the lab integrations.*

Nothing new is introduced about what rain means or what fairness means. This class asks whether the path you have designed still works when the file is large, when the same question is asked all day, and when a second person has to run it without you in the room. Scale is a design constraint. Integration is an agreement about a version. The capstone is the ability to point at one picture and say which box a defect belongs in.

---

## 🟡 Why the Athena bill is a data-structure question

Athena charges by the amount of data a query reads. The exam line is: Parquet plus partitions cuts the bill because **less data is scanned**. Here is the mechanism, so the line is not a slogan.

A **CSV** stores each row as a line of text. A query that wants humidity in May still reads every character of every column, for every row it opens. A **Parquet** file stores each column in its own compressed block and keeps statistics such as the minimum and maximum of that block. The query can skip the blocks that are not humidity. It can also skip row groups whose statistics prove they are outside the filter. That skipping of columns is **column pruning**.

A **partition** is a folder that encodes a column, such as `year=2024/month=05/`. The catalog knows the layout. A query with `WHERE year = 2024` does not open the other years. That skipping of files is **partition pruning**. **Predicate pushdown** is the general name for applying the filter as early as possible, during the scan, rather than after you have read everything into memory. The slide’s answer, “less data scanned,” is this pair of skips.

Partition on a column you actually filter, usually a date. Partition on something you rarely use and you have only made the folder tree pretty. Partition down to every station-hour and you create millions of tiny files. Listing those files can cost more than the scan you saved. Small files are a design bug, not a virtue.

**Pipelines** are how the path becomes rerunnable. **SageMaker Pipelines** chains preparation, training, evaluation, and registration. **Step Functions** is a general AWS workflow service: Glue jobs, a trust gate, an approval, in an order you can replay. A notebook run by hand has no record of which step failed and cannot be the thing an alarm starts at 2 a.m. Each step reads an explicit input version and writes an explicit output version. “Latest” is not a version. Latest changes while you are training.

Cost belongs in the scoping class as well as here. Scanning a terabyte to answer “mean humidity last May” is a choice about file layout. Select the columns you need, filter on the partition key, and store the curated table as Parquet. The bill is the feedback that the layout is wrong.

---

## 🟡 The services as a single handoff

| Service | What it contributes | What you will feel if it is missing |
|---------|---------------------|--------------------------------------|
| Glue Catalog | One dictionary of names, columns, and locations | Every team hard-codes a path, and the paths drift apart |
| Athena | SQL for profiling and audit, with no cluster to keep alive | Investigations become “download it and hope it fits” |
| SageMaker | Training, Clarify, Model Monitor, endpoints | The model stays in a notebook and never meets the gate |
| EventBridge | A clock and an event bus. It can start a drift check every morning, or start one when a new curated version lands. | Checks happen when someone remembers |
| CloudWatch | Metrics and alarms | A failed PSI sits in a JSON object and pages nobody |

**The handoff from data engineering to MLOps** is three things, and the training job is not allowed to guess them.

The **curated path**, not the raw path. Raw is the evidence. Clean is the repaired evidence. Curated is the feature table the contract allows training to read.

The **schema contract**: names, types, ranges, and which blanks are allowed. A renamed humidity column fails the job. It does not become a silent column of empties that the model treats as a real pattern.

The **version id**. The model card has to name the data. If you train on “whatever is in the prefix right now,” the card is already a lie by the time the job finishes.

You know the handoff worked when a second person can start training from the version id alone and get the row count the data team published. In the lab that count is about 5,466. Matching it is not superstition. It is the check that they read the same bytes.

An **endpoint** is the live scoring service. WeatherWise may not need one. A batch transform that finishes before 6 a.m. matches the decision. Either way, monitoring sits after serving, and a bad PSI can stop the next version from being promoted. Deployment remains the beginning of the loop.

---

## 🟢 The capstone picture

| Step | What this course used | The step owns | The step does not own |
|------|----------------------|---------------|------------------------|
| 1. Collect | IoT or an API, landing in raw S3 | An unchanged copy of what arrived | The choice of model family |
| 2. Clean and validate | The notebook or Glue, and DQ rules | Types, units, keys, ranges, a gate that can fail | Whether historical loan labels are fair |
| 3. Label | Ground Truth, Label Studio, or rules plus humans | A defined y, a guide, and an agreement number | The serving latency |
| 4. Store and version | S3 versions, DVC, the catalog | Zones and a version you can restore | Whether AUROC is the right business bar |
| 5. Bias and OOD | Clarify, the trust gate, PSI | Fit-for-use checks and a quarantine path | A paragraph for the farmer |
| 6. Explain | SHAP, Clarify, model cards | Why a row was scored, and a card a stranger can read | Tomorrow’s sensor readings |
| 7. Scale and integrate | Pipelines, Athena, a monitored endpoint or batch job | A repeatable, affordable, watched run | A quiet change to the meaning of the label |

By the end of Session 3 you have touched every row at introductory depth. The exam jumps between rows. A Fahrenheit value is row 2. A proxy that survives a deleted column is row 5. An Athena bill is row 7. Being able to point at the row is the skill the module is marking.

---

## 🟡 The warm-up, answered in full sentences

**Why fix Fahrenheit before the range check?** A range written for Celsius deletes the obviously hot Fahrenheit values and keeps the Fahrenheit values that happen to sit inside the Celsius band, such as a stored 40. The file looks clean and is still the wrong unit. Convert \(C = (F - 32) \times 5/9\) first, then test the range, and log how many cells moved.

**Three architecture decisions from Session 1.** Any three you can defend. Nightly batch rather than a per-minute stream, because the farmer acts at 6 a.m. Embeddings and a classifier rather than a language model, because a bulletin needs a category before it needs a paragraph. One AWS stack rather than a portable open-source core, trading speed against lock-in. RAG rather than fine-tuning, because the facts live in documents that change and should be cited. Raw, clean, and curated as separate zones, because training must not read the audit copy.

**Why is safetensors safer than pickle?** Loading a pickle can run arbitrary Python that was saved in the file. Safetensors stores numeric tensors and metadata and does not execute code. You load a model into a process. The file has the risk of a program. Treat it as one.

---

## 🟡 The exit ticket, longer than the slide’s three words

**We removed `group` and the bias stayed.** Group B is still predictable from `postcode_score`, correlation about −0.61. The historical labels are already uneven, disparate impact about 0.59. Deleting the column leaves both the stand-in feature and the skewed teacher. The trust gate’s proxy check is this exact case.

**PSI on humidity is 0.4.** That is above the 0.25 major-shift band. The inputs no longer look like training. Retraining immediately can memorize a broken sensor or one storm. Check the instrument, the station, and the window first. If the world truly changed and the labels are real, retrain on a declared new version, and leave the old model card pointing at the old data.

**Why do Parquet and partitions help?** The engine skips files whose partition fails the filter, and it reads only the named columns. Predicate pushdown. Fewer bytes. A smaller bill and a faster question.

---

## The mastery checklist, as board work

You are ready to leave the module when you can do these without the slides.

- Write who, what, why, and where for a project you care about, including the decision time and the metric.
- List the nine cleaning defects and explain why units come before ranges, using one Fahrenheit number that would survive a reversed order.
- Explain why two annotators who agree 90% of the time may still have a useless kappa, using a majority class.
- Draw raw, clean, and curated, and say which one training is allowed to read.
- State the course thresholds: disparate impact’s warning at 0.8, PSI stable under 0.10, watch through 0.25, major above 0.25, and the lab’s proxy alarm at absolute correlation 0.50.
- Describe the trust gate: representation, disparate impact, and proxy before training; PSI for drift; curated on pass; quarantine on fail; an alarm, not only a file.
- Name the three explanation lenses: a similar example, the features that moved this score, and whether today matches training.

---

## What Module 3 will assume

**Model engineering** is training pipelines, architecture choices, and tuning beyond a baseline. It assumes you already know the target, which features were legal at the decision time, which data version you used, and which metric and fairness check the model must pass. Tuning does not choose those. It spends effort inside them.

When you replay the labs, keep these notes open beside:

- `Weather_Data_Cleaning_Walkthrough_SageMaker.ipynb`
- `Session3_Lab_Trust_Gate.ipynb`
- `04a_Lab4a_Versioning_and_Athena.ipynb`

The notes are the lecture. The notebooks are the worked practical. If the notebook and the notes disagree about a threshold, the lab’s printed checkpoint wins for the lab, and you write the disagreement in the margin rather than silently “fixing” either one.

Back to the first week: [../Applied-AI-Engineering-Foundations/00-START-HERE.md](../Applied-AI-Engineering-Foundations/00-START-HERE.md)
