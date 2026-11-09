# Session notes — Problem scoping & architecture

*Module 2 Session 1 — Week 3 / BITS deck.*

Architecture is the set of choices that are expensive to undo: what decision you serve, which data you trust, which systems sit between them, and which metric says the system may go live. This file explains those choices before any cleaning code.

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

### What each question forces you to specify

**Who.** Name the person who acts and the person who is hurt by a miss. For WeatherWise the actor might be an irrigation planner. The harmed person is the farmer whose crop dries out after a false “no rain” or whose water is wasted after a false “rain.” Airlines are a second actor with a different harm: delays and fuel. One model score can feed both only if you write both decisions down.

**What.** The prediction must be specific enough to label. “P(rain > 1 mm tomorrow at station S01)” says the event (rain above 1 millimetre), the place, and the time. “Understand the weather” cannot be labeled. **AUROC ≥ 0.75** is the model quality bar in the example. AUROC is the area under the ROC curve: the probability that a random rainy day is scored higher than a random dry day. 0.5 is a coin toss. 1.0 is perfect ranking. 0.75 is a course threshold, not a universal definition of a good model.

**Why.** Tie the score to an action with a benefit. If nobody skips an irrigation cycle or reroutes a flight, the AUROC never becomes value.

**Where.** The runtime. **Batch** means score a file of stations overnight and deliver it by 6 a.m. An **API** (application programming interface) means another system calls you on demand. **Edge** means the model runs on a small device near the station, useful when the network is unreliable. The same math can sit in any of the three; the data pipeline cannot.

**Why the rule is strict:** Without a metric you cannot tell whether cleaning helped. Without a decision you do not know which columns are legal features, because you do not know what “at prediction time” means.

---

## 🟢 WeatherWise data (three types, one question)

| Modality | Data | Model family (start simple) |
|----------|------|-----------------------------|
| Tabular time series | Temp, humidity, pressure, wind | Gradient boosting / small NN |
| Text | Forecaster bulletins | Embeddings + classifier; LLM for summaries |
| Images | Radar | CNN / fine-tuned vision transformer |

**Multimodal fusion:** one encoder per type → concatenate or attention → P(rain).

**Modality** means the kind of signal. A table, a paragraph, and an image are three modalities even though they describe one storm.

**Time series** means measurements indexed by time. Order matters. Yesterday explains today more than a random other day does.

**Gradient boosting** builds trees one after another, each correcting the last. It is a strong default for tables. A **small NN** (neural network) is the alternative when you already expect interactions the trees are not worth tuning for. Start with boosting so you have a baseline.

**Embedding:** A list of numbers that places similar texts near each other. A classifier on those numbers can label a bulletin “fog” without an LLM writing a paragraph. Use the LLM when the product needs a new sentence, such as a summary for a pilot.

**CNN** means **convolutional neural network**, a vision model that scans local patterns such as edges of a rain band. **ViT** means **vision transformer**, a vision model that uses the transformer attention idea on image patches. Fine-tuning means you start from a model pretrained on many images and continue training on your radar.

**Fusion:** Combine the three encoded vectors. **Concatenate** means stick them end to end and let a final model weigh them. **Attention** means the model learns when to trust the radar more than the bulletin. Fusion is optional until a single modality is not enough. Shipping three pipelines badly is worse than shipping one well.

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

The fifth V is sometimes listed as **value** in older “big data” slides. This course uses **rights**. Learn the course’s five.

| V | What it means | Why it changes the design | Where it shows up |
|---|---------------|---------------------------|-------------------|
| **Volume** | How much data per day | A laptop CSV dies at terabytes. You move to columnar files and distributed jobs. | S3 storage class and Athena cost |
| **Velocity** | How fast new data arrives and how fast a decision is needed | A daily file can be batch. A sensor every second needs a stream. | Kinesis, IoT Core, Firehose |
| **Variety** | How many shapes | One parser cannot clean radar and a CSV. | Separate raw prefixes per modality |
| **Veracity** | How much you can believe the values | Noise and bias are properties of the data, not of the model. | Profiling before feature work |
| **Rights** | Whether you may use it | A dataset without a licence or consent is not a shortcut. | Contracts, retention dates, PII review |

**MB** is megabyte, **TB** is terabyte (about a million megabytes). **IoT** is the Internet of Things, small connected sensors. **Kinesis** is AWS’s streaming service.

**Ownership** means who is allowed to change a dataset and who answers when it is wrong. **Export path** means how a partner can leave with a copy, and what you must delete when they do.

---

## 🟡 User stories with data acceptance criteria

Example story:

- **As** irrigation planner **I want** rain probability by 6am
- **So that** I skip watering
- **Accept:** AUROC ≥ 0.75, completeness ≥ 95%, humidity 0–100, bias report attached

**Tools mentioned:** Jira/Azure Boards + Confluence data contracts.

**What a user story is:** A one-sentence requirement in the form “as a role, I want an outcome, so that a benefit.” It keeps the who and the why visible.

**What an acceptance criterion is:** A test that says the story is done. AUROC is a model test. Completeness and humidity range are data tests. The bias report is a release test. If any one fails, the story is not done, even if the notebook ran.

**Completeness ≥ 95%** means at least 95% of the required values are present. It is a data-quality metric, not a model metric.

**Data contract:** A written promise between the team that produces a table and the team that trains on it. It names columns, types, ranges, freshness, and what a breaking change is. Confluence is the document store named in class. Jira or Azure Boards tracks the work. The tool is not the point; the promise is.

**Why stories beat “build a weather model”:** The acceptance line tells the cleaning lab which checks to automate. Humidity above 100 fails the story even if the model still trains.

---

## 🟡 Architecture frameworks (checklists, not religion)

| Framework | Use |
|-----------|-----|
| **AWS Well-Architected** (ML + GenAI lenses) | Security, cost, reliability review |
| **CRISP-DM** | Classic data mining phases |
| **MLOps maturity** | Notebooks → pipelines → CT/CD |
| **NIST AI RMF** | Govern, map, measure, manage |

**What a framework is here:** A checklist written by someone who has seen the failure before. It does not pick your model.

| Name | Expansion | What you actually do with it |
|------|-----------|------------------------------|
| **AWS Well-Architected** | A review method with pillars such as security, reliability, cost, and operations. The ML and GenAI lenses add model-specific questions. | Ask “who can read the raw bucket?” and “what does a bad endpoint cost per hour?” before launch |
| **CRISP-DM** | Cross-Industry Standard Process for Data Mining | Business understanding → data understanding → preparation → modeling → evaluation → deployment. It is the older sibling of the AI lifecycle in Module 1. |
| **MLOps maturity** | How repeatable the machine-learning operation is | Level 0 is a notebook. Later levels are scheduled pipelines, tests, and continuous training. |
| **CT** | Continuous training | Retraining when data or schedule says so, not only when a human remembers |
| **CD** | Continuous delivery | A tested path from a new model version to a deployed endpoint |
| **NIST AI RMF** | National Institute of Standards and Technology AI Risk Management Framework | Govern (policies), map (context and risks), measure (metrics and tests), manage (act on the measurements) |

**Where students overuse them:** Quoting the framework name in a design doc without answering who, what, why, and where. Use the checklist to find a hole, then write the decision in plain language.

---

## 🟡 Batch ML reference architecture (AWS)

```
Sources → Ingest (Glue/Firehose) → S3 raw/clean/curated + Glue Catalog
      → Prepare (DataBrew, Data Wrangler) → SageMaker Train
      → Model Registry → Endpoint/Batch Transform → Monitor
```

**Data engineering owns the top; MLOps owns the bottom; they meet at curated zone.**

| Piece | What it is | Why it is in the picture |
|-------|------------|--------------------------|
| **Glue** | Serverless data integration and a data catalog | Scheduled transforms and a list of tables other tools can query |
| **Firehose** | A delivery stream into S3 | Lands streaming records as files without you running a server |
| **Glue Catalog** | The metadata directory | Athena and others need names, columns, and locations |
| **DataBrew** | A visual data-prep tool | Cleaning without writing every step from scratch |
| **Data Wrangler** | SageMaker’s data-prep interface | The same job closer to the training environment |
| **SageMaker** | AWS’s model training and hosting platform | Train, register, and serve |
| **Model Registry** | A versioned list of models and their approval state | So production points at a specific model, not “whatever was in the notebook” |
| **Endpoint** | A live API that scores one request or a small batch | Real-time path |
| **Batch Transform** | Score a whole file, then stop | The 6 a.m. weather path |
| **Curated zone** | The feature-ready, versioned data | The handoff. Data engineering says “this table meets the contract.” MLOps says “we trained on this version.” |

**Why the meeting point matters:** If training reads from `raw/`, every model reruns yesterday’s defects and the two teams cannot blame the same contract. Curated is the agreed table.

---

## 🟡 GenAI RAG architecture

```
Docs in S3 → chunk + embed → vector store → user query → retrieve top-k → LLM + guardrails → answer with citations
```

**Bedrock Knowledge Bases** packages much of the left side — you still own **data quality in**.

**What RAG is:** Retrieval-augmented generation. The LLM is not asked to remember your policy. A search step finds the relevant chunks, and the prompt includes those chunks. The model writes from them.

**Chunk:** A piece of a document small enough to embed and to fit in a prompt, often a few hundred tokens with some overlap so a sentence cut in half is still found.

**Embed:** Turn the chunk into a vector. Similar meanings land nearby.

**Vector store:** A database that can answer “which stored vectors are closest to this query vector?”

**Top-k:** Return the k nearest chunks. k that is too small misses context. k that is too large buries the model in noise and cost.

**Citation:** A link or passage id so a human can check the claim. Without it, RAG collapses back into an ungrounded paragraph.

**Guardrails:** Filters on the question and the answer, plus a rule to say “not in the documents” instead of inventing.

**Bedrock** is AWS’s managed foundation-model service. **Knowledge Bases** is its packaged RAG path: ingest, chunk, embed, and retrieve. Packaging does not clean a wrong bulletin or a duplicated policy. Garbage in still means a confident, cited, wrong answer.

---

## 🟡 LLM → agent ladder

1. **LLM** — answers from training
2. **+ RAG** — looks up your docs
3. **+ Tools** — calls APIs
4. **Agent** — plans multi-step loops
5. **Multi-agent** — specialists collaborate

Each step adds power **and risk** — only move right when needed.

**MCP** = agent ↔ tools; **A2A** = agent ↔ agent.

| Rung | What it can do | The new risk | Stay here if |
|------|----------------|--------------|--------------|
| LLM | Continue a prompt | Hallucination | The task is drafting and a human reviews |
| RAG | Answer from your files | Retrieves the wrong chunk and sounds sourced | The facts live in documents |
| Tools | Call a calculator, a database, or a weather API | A bad argument changes real data | You need a live number, not a remembered one |
| Agent | Decide the next tool call in a loop | Runaway loops, hidden actions, hard-to-test plans | The steps are not known in advance |
| Multi-agent | Split research, critique, and execution | Cost, contradiction, and nobody accountable | One agent’s context is genuinely overloaded |

**MCP** means **Model Context Protocol**, a standard way for an agent to discover and call tools. **A2A** means **agent-to-agent**, a way for agents to hand work to each other. They are interfaces. They do not make the underlying action safe.

**Why “only move right when needed”:** Each rung is harder to evaluate. A single prompt has one input and one output. An agent has a tree of actions. Do not take that test burden on to summarize a PDF you could have chunked.

---

## 🟡 Cloud strategy (three patterns)

1. **One vertical stack** (e.g. all AWS) — fast, more lock-in
2. **Tools-first** — pick cloud that supports required services
3. **Open-source core** (K8s, MLflow, Airflow, DVC) — portable, you operate it

| Pattern | What you gain | What you pay |
|---------|---------------|--------------|
| **Vertical stack** | Services that already connect: S3 to Glue to SageMaker | Switching clouds means rewriting the pipeline. That cost is **lock-in**. |
| **Tools-first** | You start from a required capability (“we need managed labeling”) and pick the cloud that has it | A mixed estate to operate |
| **Open-source core** | Kubernetes, MLflow, Airflow, and DVC run on more than one cloud | Your team runs upgrades, scaling, and security |

**K8s** is Kubernetes, a system for running containers. **Airflow** schedules pipelines. **MLflow** tracks experiments and models. **DVC** versions large files with Git pointers.

**Where the course labs sit:** They are pattern 1 so you can finish a lab. The notes still name pattern 3 so you recognize a portable design when you see one.

---

## 🟢 Model family chooser (shape of data)

Pick from **data shape**, not hype:

- Tables → classical ML / boosting
- Sequences → LSTM/transformer if needed
- Text → embeddings; LLM for generation/Q&A
- Images → CNN/ViT

**LSTM** means **long short-term memory**, an older neural network for sequences. Transformers have replaced LSTMs for many language tasks. For a short weather table, neither is the starting point; a boosted tree with lag features is.

**Why shape beats hype:** The shape tells you what a row means and which leakage is possible. A headline does not.

---

## Checkpoint

For WeatherWise bulletins only, would you start with an LLM or embeddings + classifier?

**Answer:** Start with embeddings plus a classifier if the output is a category such as fog, rain, or storm. You have labels, you can measure precision and recall, and the result is stable. Use an LLM when the output must be a new paragraph, such as a briefing, and keep a human or a citation check on it. Generation is the wrong first tool for a labeled classification task.

**Next:** [02-data-collection-ingestion-lifecycle.md](02-data-collection-ingestion-lifecycle.md)
