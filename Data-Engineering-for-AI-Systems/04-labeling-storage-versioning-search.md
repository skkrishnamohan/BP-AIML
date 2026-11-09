# Session notes — Labels, lakes, versioning, vectors

*Module 2 Session 2 + Labs 3, 4a, 4b.*

---

## 🟢 Labels are a product

Ground truth = **supervised learning fuel**. Bad labels → bad model, no matter how fancy the algorithm.

---

## 🟡 Four labeling methods

| Method | Quality | Speed | When |
|--------|---------|-------|------|
| **Manual (experts)** | Highest | Slowest | Gold sets, hard cases |
| **Crowd** | Medium | Fast | Clear guidelines + redundancy |
| **Synthetic** | Varies | Fast | Augmentations, renders, LLM text |
| **Programmatic** | Noisy | Very fast | Rules / weak supervision |

**Real projects:** hybrid — automation for volume, humans for judgment gaps.

---

## 🟡 Label quality workflow

1. **Guidelines** with examples + edge cases  
2. **Pilot** 20–50 items, 2+ annotators  
3. **Measure agreement** (Cohen’s **kappa**)  
4. **Adjudicate** disagreements with expert  
5. **Scale** with gold questions + spot audits  

**Kappa intuition:**

- 1.0 = perfect beyond chance  
- 0 = chance agreement  
- < 0.6 → fix guidelines, not blame people  

**Lab 3 numbers (simulated):** kappa ~0.78; ~13.5% bulletins need adjudication.

---

## 🟡 Programmatic labeling (bulletins)

Pattern:

- Several **labeling functions** vote (`storm`, `fog`, `rain`) or **abstain**  
- Majority vote → label  
- **Abstain** rows → humans  

**Bug example:** `lf_rain` matching “**No** rain expected” → add **negation handling** (lab task).

**Hybrid savings (illustrative):** ~80% rows by rules @ ~94.6% accuracy; rest to humans; large cost reduction vs all-human.

---

## 🟡 Label Studio vs Ground Truth

| | Label Studio | SageMaker Ground Truth |
|---|--------------|------------------------|
| Hosting | Your laptop/EC2 | Managed AWS |
| Strength | Flexible configs | Workforce + IAM integration |
| Pay | You manage infra | Per labeled object |

**Config = UI + label schema** (XML for bulletin categories: clear/cloudy/rain/fog/storm).

---

## 🟡 Storage map for AI

| Need | Store type |
|------|------------|
| Files, lake | **S3** zones |
| SQL analytics on lake | **Glue Catalog + Athena** (lakehouse) |
| Heavy BI concurrency | **Redshift Serverless** |
| Relationships | **Graph** (Neptune, Neo4j) |
| Similarity search | **Vector** DB / OpenSearch k-NN |

**Start lakehouse;** add warehouse when many users hammer same aggregates daily.

---

## 🟡 Ingestion production habits

| Property | Meaning |
|----------|---------|
| **Idempotent** | Re-run job ≠ duplicate rows |
| **Schema evolution** | New columns don’t break readers (Glue Catalog) |
| **Late data** | Event time vs arrival time + lateness window |
| **Backfills** | Replay into **new version**, not in-place overwrite of history |

---

## 🟡 Versioning: S3 + DVC

**S3 versioning:** every overwrite keeps history; list versions, restore old object.

**DVC pattern:**

- Git stores **pointers** (`.dvc` files)  
- S3 stores **blobs**  
- Repro experiment = `git commit` + `dvc push` + training config  

```
Reproducible run = code commit + data version + hyperparameters
```

---

## 🟡 Lab 4a — Athena lakehouse

**Goals:**

1. Enable bucket versioning  
2. Upload two versions of `weather_clean.csv`  
3. Restore older version  
4. Create Athena table via Glue Catalog  
5. Run SQL; note **data scanned** (cost driver)

**Checkpoint:** count ≈ **5,466** rows; can download prior version.

**Cost tip:** Athena bills per TB scanned — **Parquet + partitions** reduce scan later (Session 3).

---

## 🟡 Embeddings & semantic search (Lab 4b)

Pipeline:

```
Text → embedding model (Titan/Cohere) → vector → nearest-neighbor index
```

**Why:** “hard to see the road” matches **fog** bulletins without shared keywords (unlike grep).

**CLIP idea:** image + text encoders map to **shared vector space** (multimodal search).

---

## 🟢 Graph + GraphRAG (concept)

Nodes = entities (station, alert, route); edges = relationships.

Query paths like: *storm alert → routes affected → customers*.

**GraphRAG:** retrieval follows **edges**, not just similar text chunks.

---

## Session 2 remember list

1. Labels need guidelines + agreement  
2. Hybrid labeling (rules + humans) wins on cost  
3. Pick store by question; **version everything**

**Next:** [05-trust-bias-ood-explainability.md](05-trust-bias-ood-explainability.md)
