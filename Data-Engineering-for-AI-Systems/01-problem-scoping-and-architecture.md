# Session notes — Problem scoping & architecture

*Module 2 Session 1 — Week 3 / BITS deck.*

---

## 🟢 Frame before you build

Answer **Who? What? Why? Where?** before touching data.

| Question | WeatherWise example |
|----------|---------------------|
| **Who** | Farmers, airlines — who is harmed if wrong? |
| **What** | P(rain > 1mm tomorrow at station S01) + metric (e.g. AUROC ≥ 0.75) |
| **Why** | Fewer wasted irrigation cycles, fewer delays |
| **Where** | Cloud batch vs real-time API vs edge |

**Rule:** No metric + no decision = not ready to collect data.

---

## 🟢 WeatherWise data (three types, one question)

| Modality | Data | Model family (start simple) |
|----------|------|-----------------------------|
| Tabular time series | Temp, humidity, pressure, wind | Gradient boosting / small NN |
| Text | Forecaster bulletins | Embeddings + classifier; LLM for summaries |
| Images | Radar | CNN / fine-tuned vision transformer |

**Multimodal fusion:** one encoder per type → concatenate or attention → P(rain).

---

## 🟡 5 Vs of data requirements

| V | Question |
|---|----------|
| **Volume** | MB/day vs TB/day → cost & tools |
| **Velocity** | Batch file vs stream → Kinesis/IoT |
| **Variety** | Table vs text vs image → different pipelines |
| **Veracity** | How noisy/biased? Profile first |
| **Rights** | Licence, consent, PII, retention |

Also analyze: **sources**, ownership, export path.

---

## 🟡 User stories with data acceptance criteria

Example story:

- **As** irrigation planner **I want** rain probability by 6am  
- **So that** I skip watering  
- **Accept:** AUROC ≥ 0.75, completeness ≥ 95%, humidity 0–100, bias report attached  

**Tools mentioned:** Jira/Azure Boards + Confluence data contracts.

---

## 🟡 Architecture frameworks (checklists, not religion)

| Framework | Use |
|-----------|-----|
| **AWS Well-Architected** (ML + GenAI lenses) | Security, cost, reliability review |
| **CRISP-DM** | Classic data mining phases |
| **MLOps maturity** | Notebooks → pipelines → CT/CD |
| **NIST AI RMF** | Govern, map, measure, manage |

---

## 🟡 Batch ML reference architecture (AWS)

```
Sources → Ingest (Glue/Firehose) → S3 raw/clean/curated + Glue Catalog
      → Prepare (DataBrew, Data Wrangler) → SageMaker Train
      → Model Registry → Endpoint/Batch Transform → Monitor
```

**Data engineering owns the top; MLOps owns the bottom; they meet at curated zone.**

---

## 🟡 GenAI RAG architecture

```
Docs in S3 → chunk + embed → vector store → user query → retrieve top-k → LLM + guardrails → answer with citations
```

**Bedrock Knowledge Bases** packages much of the left side — you still own **data quality in**.

---

## 🟡 LLM → agent ladder

1. **LLM** — answers from training  
2. **+ RAG** — looks up your docs  
3. **+ Tools** — calls APIs  
4. **Agent** — plans multi-step loops  
5. **Multi-agent** — specialists collaborate  

Each step adds power **and risk** — only move right when needed.

**MCP** = agent ↔ tools; **A2A** = agent ↔ agent.

---

## 🟡 Cloud strategy (three patterns)

1. **One vertical stack** (e.g. all AWS) — fast, more lock-in  
2. **Tools-first** — pick cloud that supports required services  
3. **Open-source core** (K8s, MLflow, Airflow, DVC) — portable, you operate it  

---

## 🟢 Model family chooser (shape of data)

Pick from **data shape**, not hype:

- Tables → classical ML / boosting  
- Sequences → LSTM/transformer if needed  
- Text → embeddings; LLM for generation/Q&A  
- Images → CNN/ViT  

---

## Checkpoint

For WeatherWise bulletins only, would you start with an LLM or embeddings + classifier?

**Next:** [02-data-collection-ingestion-lifecycle.md](02-data-collection-ingestion-lifecycle.md)
