# Module 2 — Start here

**Data Engineering for AI Systems** is about the **ingredients** and **plumbing** before and after the model. If Module 1 was “how learning works,” Module 2 is “how we trust data at scale.”

**What data engineering means here:** Design the path that collects, cleans, labels, stores, and versions data so a later model can be trained and audited.

**Why it is its own module:** Most failed models are not failed math. They are bad targets, leaked columns, unlabeled edge cases, or a file nobody can reproduce.

**How you will practice:** Two running stories, WeatherWise and LoanLens, plus AWS labs. Learn the idea before you replay the notebook.

**Where the work sits:** Object storage, catalogs, labeling tools, and checks that run before training is allowed to start.

## Case studies (use them as a spine)

| Case | What it teaches |
|------|-----------------|
| **WeatherWise** | Multimodal weather data → clean → label → version → rain prediction |
| **LoanLens** | Fairness, proxies, disparate impact, trust gates |

**WeatherWise** is a rain-prediction product. The data is sensor tables, forecaster text, and radar images. It teaches the happy-path pipeline and the ways weather data lies (units, time order, missing values).

**LoanLens** is a lending dataset with a historically advantaged group and a proxy feature. It teaches that a model can be technically accurate and still unfair, and that dropping a sensitive column does not remove the information if another column carries it.

## Words that unlock this module

| Term | Expansion | Meaning |
|------|-----------|---------|
| **S3** | Simple Storage Service | AWS object storage. Files live in buckets and prefixes such as `raw/` and `clean/`. |
| **AWS** | Amazon Web Services | The cloud used in the labs. The ideas also exist on other clouds. |
| **DQ** | Data quality | Rules that say a dataset is fit to use |
| **PII** | Personally identifiable information | Data that identifies a person; restrict and redact it |
| **ETL** | Extract, transform, load | Move data out of a source, change it, and store it |
| **RAG** | Retrieval-augmented generation | Find relevant documents, then let an LLM answer from those documents |
| **OOD** | Out of distribution | An input unlike what the model was trained on |
| **PSI** | Population Stability Index | A score of how much a feature’s distribution has shifted |
| **SHAP** | SHapley Additive exPlanations | A way to attribute a prediction to features |
| **DVC** | Data Version Control | Git-like pointers for large data files stored elsewhere |
| **AUROC** | Area under the ROC curve | A ranking metric; 0.5 is chance, 1.0 is perfect separation |

## Read in this order

| # | File | Sessions |
|---|------|----------|
| 1 | [01-problem-scoping-and-architecture.md](01-problem-scoping-and-architecture.md) | Session 1 (scope, RAG, agents) |
| 2 | [02-data-collection-ingestion-lifecycle.md](02-data-collection-ingestion-lifecycle.md) | Session 1 |
| 3 | [03-cleaning-and-validation-weatherwise.md](03-cleaning-and-validation-weatherwise.md) | Session 1 lab |
| 4 | [04-labeling-storage-versioning-search.md](04-labeling-storage-versioning-search.md) | Session 2 |
| 5 | [05-trust-bias-ood-explainability.md](05-trust-bias-ood-explainability.md) | Session 3 |
| 6 | [06-scale-integration-and-labs.md](06-scale-integration-and-labs.md) | Session 3 labs + capstone |

## Level guide

- 🟢 Basics — read first even if labs confused you. Definitions and the case story live here.
- 🟡 Intermediate — connects AWS services to concepts. Know what the service is for, not every console click.
- 🔴 Advanced — PSI math, mitigation tradeoffs, cost tuning. Come back after you can draw the pipeline.

## Pipeline map (put on your wall)

```
Collect → Clean → Validate → Label → Store/Version
    → Train (later modules) → Bias/OOD checks → Explain → Serve → Monitor
```

**How to read the map:** Each arrow is a place defects enter. Cleaning does not fix an unfair label. A fairness check does not fix a Fahrenheit value stored as Celsius. Do the step that matches the defect.

**Start:** [01-problem-scoping-and-architecture.md](01-problem-scoping-and-architecture.md)
