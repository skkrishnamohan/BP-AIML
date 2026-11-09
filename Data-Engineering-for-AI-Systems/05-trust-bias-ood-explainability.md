# Session notes — Trust: bias, OOD, explainability

*Module 2 Session 3 + Labs 5, 6 + Trust Gate lab.*

---

## 🟢 Four kinds of bias

| Type | Example |
|------|---------|
| **Data** | Stations only near cities; loan history reflects past discrimination |
| **Algorithm** | Optimizing overall accuracy ignores rare group |
| **Model** | Uses **proxy** (postcode → group) even if group column removed |
| **Evaluation** | Average metrics hide group failure |

**Bias can enter at every stage** — check at each arrow in the pipeline, not once at the end.

---

## 🟡 LoanLens (human-centric case)

- **Task:** approve/decline loans  
- **12k** synthetic applicants; group B ~30%  
- **Historical labels** favored group A  
- Features: income, credit, debt ratio, **postcode score** (proxy corr ≈ **-0.61** with group B)

| Metric | Group A | Group B |
|--------|---------|---------|
| Historical approval | ~75.5% | ~44.2% |

**Fairness through unawareness fails** when proxies exist.

---

## 🟡 Fairness metrics (know tradeoffs)

| Metric | Meaning |
|--------|---------|
| **Disparate impact (DI)** | Approval rate B / rate A — **< 0.8** often “four-fifths” concern |
| **Statistical parity diff** | Rate B − rate A |
| **Equal opportunity diff** | TPR gap among **truly creditworthy** |

Metrics **conflict** — choose with stakeholders which harm matters.

---

## 🟡 Mitigation timeline

| Phase | Actions |
|-------|---------|
| **Pre-processing** | More data for underrepresented groups, reweigh/resample, fix labels, remove proxies |
| **In-processing** | Fairness constraints, adversarial debiasing, Fairlearn |
| **Post-processing** | Group thresholds, reject-option, human review band |

**Prefer fixing data first** — helps every future model.

**SageMaker Clarify:** bias reports pre/post training; wire as **pipeline gate**.

---

## 🟡 Trust Gate lab (data engineer view)

**Purpose:** decide if data is fit to use **before** modeling.

**Bias checks (loan CSV, no model needed):**

1. **Representation** — group B ≥ 10% of rows?  
2. **Disparate impact** on historical approvals ≥ 0.80?  
3. **Proxy** — any feature |corr| with group ≥ 0.50?  

**Course checkpoint:** DI ≈ **0.59** → **FAIL**; `postcode_score` proxy → **FAIL**.

> Removing `group_b` column does **not** fix bias if proxy remains.

**Drift check (weather):** **PSI** on features vs training reference.

| PSI | Interpretation |
|-----|----------------|
| < 0.10 | Stable |
| 0.10–0.25 | Watch |
| > 0.25 | Major shift |

**Routing:** pass → `curated/`; fail → `quarantine/`; publish metrics to **CloudWatch** + JSON on S3.

---

## 🟡 Out-of-distribution (OOD)

Model always outputs something — **OOD detection** says “I shouldn’t trust this.”

| OOD type | Example |
|----------|---------|
| Unseen population | Desert station, trained on temperate |
| Broken sensor | Stuck 870 hPa / 99% humidity |
| World change | New climate regime |
| Out-of-scope ask | Weather bot asked for legal advice |

**Detector families:**

1. **Output-based** — low max softmax probability (model-dependent)  
2. **Density-based** — Mahalanobis, Isolation Forest  
3. **Distribution-based** — **PSI**, KS test (Model Monitor)

**Production pattern:** OOD gate → abstain / human / fallback → log for future labels.

---

## 🟡 Explainability — three lenses

| Lens | Question | Tools |
|------|----------|-------|
| **Sample** | Why this row? | Nearest neighbors, prototypes |
| **Feature** | What inputs matter? | Permutation importance, **SHAP**, PDP |
| **Distribution** | Does today match training? | **PSI**, KS |

**Weather model insight:** shuffling **pressure** hurt AUROC most — matches meteorology (sanity check).

---

## 🟡 SHAP (intuition)

From game theory: fairly split “payout” (prediction) among features.

- **Local:** why **this** day got high P(rain)  
- **Global:** average |SHAP| across rows  

**TreeSHAP** fast on tree models; **SageMaker Clarify** at scale.

**Cookie vs dog story:** explanation shows model used **eye/nose shape** — fix with hard negatives + augmentation.

---

## 🟡 Transparency artifacts

| Document | Contents |
|----------|----------|
| **Data card** | Source, labeling, gaps, version, intended use |
| **Model card** | Metrics per group, limits, training data version |
| **Lineage** | Which data + code + params → this artifact |
| **ADR** | Architecture decisions & rejected options |

---

## Lab 5 & 6 checkpoints

- **Lab 5:** baseline DI flagged; Mahalanobis catches desert/broken sensor better than naive softmax  
- **Lab 6:** PSI flags temp/humidity/pressure; wind/cloud stable → targeted investigation  

---

## Checkpoint

Why did removing `group` from features not remove bias in LoanLens?

**Next:** [06-scale-integration-and-labs.md](06-scale-integration-and-labs.md)
