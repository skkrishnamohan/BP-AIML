# Session notes — Collection, ingestion, lifecycle

*Module 2 Session 1.*

---

## 🟢 Data lifecycle (loop)

```
Collect → Clean → Validate → Label → Version → Monitor (drift)
```

**Principle:** A model is only as good as its data; late defects cost ~10× more.

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

---

## 🟡 Batch vs streaming

| | Batch | Streaming |
|---|-------|-----------|
| **Pattern** | Daily CSV drop | Sensors every minute |
| **Latency** | Hours | Seconds |
| **AWS** | Scheduled Glue, DMS | IoT Core + Firehose |
| **Landing** | S3 **raw** zone (immutable) |

**Never edit raw** — it’s your audit trail. Clean copies go to `clean/` and `curated/`.

---

## 🟡 Lake zones (S3 convention)

| Zone | Contents | Who writes |
|------|----------|------------|
| `raw/` | As received | Ingestion |
| `clean/` | Validated, typed Parquet/CSV | Pipelines + DQ rules |
| `curated/` | Feature-ready, versioned | ML teams |

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

---

## 🟡 Time-series rules (WeatherWise)

1. **Sort by time**; store **UTC**  
2. **Resample** to regular spacing; mark gaps  
3. **Seasonality features** — hour, day-of-year  
4. **Lag features** — yesterday’s pressure  
5. **No leakage** — future values must not enter features or scalers  
6. **Chronological split** — train on past, test on future (**not random shuffle**)

---

## 🟡 Data augmentation & synthetic

- **Images:** rotate, flip, brightness (labels must stay valid)  
- **Tabular synthetic:** SDV-style generators  
- **Text synthetic:** LLM-generated examples — **human spot-check**  
- **Synthetic ≠ truth** — always evaluate on real held-out data  

---

## 🟡 safetensors vs pickle

| | Pickle | safetensors |
|---|--------|-------------|
| Risk | Can execute code on load | Tensors + metadata only |
| Use | Legacy hubs | Preferred for sharing models |

Treat model files like **executables**.

---

## 🟡 Annotation overview (preview Session 2)

- **CVAT** — vision/video; model-assisted boxes  
- **Label Studio** — flexible, self-hosted  
- **Ground Truth** — managed AWS workforce  

**Model-assisted labeling:** model proposes → human corrects → faster than draw-from-scratch.

---

## Checkpoint

Why is chronological split mandatory for rain prediction?

**Next:** [03-cleaning-and-validation-weatherwise.md](03-cleaning-and-validation-weatherwise.md)
