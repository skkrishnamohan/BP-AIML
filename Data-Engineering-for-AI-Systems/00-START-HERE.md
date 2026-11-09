# Module 2 — Start here

Module 1 asked how a model learns. This module asks whether the data deserves to be learned from. Most failed models in the labs are not failed math. They are a bad target, a leaked column, a label two annotators would not agree on, or a file nobody can restore.

**Data engineering**, in this course, is the design of the path that collects, cleans, labels, stores, and versions data so a later model can be trained and audited. The path is the product. The model is a consumer of the path.

## Two stories, used for the whole module

**WeatherWise** predicts rain. The data is a table from stations, text bulletins from forecasters, and radar images. It is the happy path that keeps going wrong in ordinary ways: mixed units, mixed date formats, a target that peeks at tomorrow, a file that cannot be restored. When a lecture says “the station,” it means WeatherWise.

**LoanLens** is a synthetic lending table. One group was approved far more often in the historical labels, and a postcode feature still carries group membership after you delete the group column. It is here so you can see unfairness with numbers, without using real applicants. When a lecture says “the applicant,” it means LoanLens.

Keep both stories distinct. A Fahrenheit bug is not a fairness bug. A proxy is not fixed by a date parser.

## Words you will need before Session 1

| Term | Say it in full once | The meaning in this module |
|------|---------------------|----------------------------|
| AWS | Amazon Web Services | The cloud the labs use. The ideas exist on other clouds. |
| S3 | Simple Storage Service | Object storage. A bucket holds files. Prefixes such as `raw/` and `clean/` are the folders. |
| DQ | Data quality | Rules that say a table is fit to use |
| PII | Personally identifiable information | Data that identifies a person. Restrict it and redact it. |
| ETL | Extract, transform, load | Take data from a source, change it, put it somewhere |
| RAG | Retrieval-augmented generation | Find the relevant documents first, then let a language model answer from those documents |
| OOD | Out of distribution | An input unlike anything the model was trained on |
| PSI | Population Stability Index | How far a feature’s distribution has moved from the training distribution |
| SHAP | SHapley Additive exPlanations | A way to give each feature a share of one prediction |
| DVC | Data Version Control | Small pointers in Git, with the large files stored elsewhere |
| AUROC | Area under the ROC curve | A ranking score. 0.5 is chance. 1.0 is perfect separation. |

## The order of the classes

| Class | File | Leave able to… |
|-------|------|----------------|
| 1 | [01-problem-scoping-and-architecture.md](01-problem-scoping-and-architecture.md) | Write who, what, why, and where, and draw the system around them |
| 2 | [02-data-collection-ingestion-lifecycle.md](02-data-collection-ingestion-lifecycle.md) | Land data without editing the evidence, and split time honestly |
| 3 | [03-cleaning-and-validation-weatherwise.md](03-cleaning-and-validation-weatherwise.md) | Name the nine defects and say why the order of fixes matters |
| 4 | [04-labeling-storage-versioning-search.md](04-labeling-storage-versioning-search.md) | Treat labels, versions, and search as products |
| 5 | [05-trust-bias-ood-explainability.md](05-trust-bias-ood-explainability.md) | Compute a disparate impact and say what a trust gate does |
| 6 | [06-scale-integration-and-labs.md](06-scale-integration-and-labs.md) | Put the whole pipeline in one picture and explain the cloud bill |

🟢 is the definition and the worked example. 🟡 connects the idea to a service or a design choice. 🔴 is the trap.

## The pipeline, which is also the map of the module

```
Collect → Clean → Validate → Label → Store and version
    → Train (later modules) → Bias and OOD checks → Explain → Serve → Monitor
```

Each arrow is a door for a different defect. Cleaning does not fix an unfair label. A fairness report does not convert Fahrenheit to Celsius. Do the step that matches the defect. If you remember only this diagram, you can still place every later lecture on it.

First class: [01-problem-scoping-and-architecture.md](01-problem-scoping-and-architecture.md)
