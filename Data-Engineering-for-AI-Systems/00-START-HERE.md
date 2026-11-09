# Module 2 — Start here

**Data Engineering for AI Systems** is about the **ingredients** and **plumbing** before and after the model. If Module 1 was “how learning works,” Module 2 is “how we trust data at scale.”

## Case studies (use them as a spine)

| Case | What it teaches |
|------|-----------------|
| **WeatherWise** | Multimodal weather data → clean → label → version → rain prediction |
| **LoanLens** | Fairness, proxies, disparate impact, trust gates |

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

- 🟢 Basics — read first even if labs confused you  
- 🟡 Intermediate — connects AWS services to concepts  
- 🔴 Advanced — PSI math, mitigation tradeoffs, cost tuning  

## Pipeline map (put on your wall)

```
Collect → Clean → Validate → Label → Store/Version
    → Train (later modules) → Bias/OOD checks → Explain → Serve → Monitor
```

**Start:** [01-problem-scoping-and-architecture.md](01-problem-scoping-and-architecture.md)
