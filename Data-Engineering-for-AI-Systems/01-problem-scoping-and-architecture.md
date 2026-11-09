# Class notes — Scoping the problem before anyone opens a file

*Module 2, Session 1. The Week 3 / BITS deck.*

Module 1 ended with “define the problem.” This class refuses to let you skip it. Architecture, here, means the choices that are expensive to undo: which decision you serve, which data you are allowed to trust, which systems sit between them, and which number has to pass before anything goes live. A model name is not an architecture.

---

## 🟢 Four questions, answered for WeatherWise

We do not open the CSV until these four have answers.

**Who.** The person who acts, and the person who is hurt if the action is wrong. For WeatherWise the actor might be an irrigation planner. The person hurt is the farmer: a false “no rain” dries the crop, a false “rain” wastes the water. Airlines are a second actor with a different hurt, delays and fuel. One score can feed both only if both decisions are written down. “Users” is not an answer.

**What.** The prediction, specific enough that a label exists. “Understand the weather” cannot be labeled. The class example is: the probability that rainfall at station S01 tomorrow exceeds 1 millimetre, and a quality bar such as AUROC at least 0.75.

Say AUROC in a full sentence so it is not a badge. You take a random day that really rained and a random day that did not. AUROC is the probability that the model scored the rainy day higher. 0.5 means the ranking is chance. 1.0 means every rainy day outranks every dry day. 0.75 is the example bar in the story, not a law of nature. A product can need 0.9, or it can ship at 0.7 if the action is cheap to undo. The bar is part of the scope. It is not discovered after training.

**Why.** The action and the benefit. Fewer wasted irrigation cycles. Fewer delays. If nobody’s morning changes, the AUROC never becomes value.

**Where.** The runtime. **Batch** means score a file of stations overnight and deliver it by 6 a.m. An **API**, an application programming interface, means another system calls you at the moment of need. **Edge** means the model runs on a small computer near the station, which matters when the network fails. The same math can live in any of the three. The pipeline cannot. A stream of per-minute scores is wasted money if the farmer acts once at dawn.

The rule from the front of the room: no metric and no decision means you are not ready to collect data. Without a metric you cannot tell whether cleaning helped. Without a decision you do not know which columns are legal, because you do not know what “at prediction time” means.

---

## 🟢 Three kinds of weather data, and the temptation to fuse them on day one

**Modality** means the kind of signal. WeatherWise has three, and they are not interchangeable.

The **table** is a time series: temperature, humidity, pressure, wind, indexed by time. Order matters. Yesterday explains today more than a random other day does. The starting model family is gradient boosting or a small neural network. **Gradient boosting** grows trees one after another, each tree correcting the last. It is the strong default for a table, and it gives you a baseline a fancier model has to beat.

The **text** is the forecaster’s bulletin. If the output you need is a category such as fog or storm, start with an **embedding** and a classifier. An embedding is a list of numbers that places similar sentences near each other. You can measure precision and recall. If the output you need is a new paragraph for a pilot, then you have a writing task and a language model is appropriate, with a person or a citation on the result. Do not start with the language model for a labeled classification. Generation is the wrong first tool when the label already exists.

The **image** is the radar. The starting families are a **CNN**, a convolutional neural network, which scans local patterns such as the edge of a rain band, or a **ViT**, a vision transformer, which applies the transformer idea to patches of the image. **Fine-tuning** means you start from a model that has already seen many images and continue training on your radar, rather than from empty weights.

**Fusion** means combining the three. Each modality has an encoder. You either **concatenate**, stick the vectors end to end, or use **attention**, let the model learn when the radar deserves more trust than the bulletin. Fusion is how you get one P(rain) from three signals. It is also how you get three half-finished pipelines and no baseline. Ship one modality well before you fuse. The class example starts with the table, because the label and the metric are easiest to define there.

---

## 🟡 The five V’s, using the course’s fifth

Older “big data” talks list volume, velocity, variety, veracity, and value. This course replaces value with **rights**. Learn the course’s five. Value still matters. It is the “why” you already wrote.

**Volume.** How much arrives per day. A megabyte fits in a laptop CSV. A terabyte does not. Volume pushes you to columnar files and to jobs that do not need the whole file in memory. It also shows up later as the Athena bill.

**Velocity.** How fast data arrives, and how fast the decision is needed. Those are different clocks. A station can report every minute while the farmer acts once a day. Matching the pipeline to the station and ignoring the farmer is how teams build a stream nobody waits for. **Kinesis** is the AWS streaming service. **IoT** is the Internet of Things, the small connected sensors. IoT Core is where those messages enter AWS.

**Variety.** Tables, text, and images need different parsers. One raw prefix per modality keeps a broken radar file from being fed to the CSV cleaner.

**Veracity.** How much you believe the values. Noise, bias, and sentinels are properties of the data. Profile them before you design features. A model will treat −999 as a temperature if you let it.

**Rights.** Licence, consent, PII, and how long you may keep the file. A dataset you are not allowed to train on is not a shortcut. Also write the owner, the person who answers when a column changes, and the export path, how a partner leaves with a copy and what you must delete when they do.

---

## 🟡 A user story that includes the data

The story form is: as a role, I want an outcome, so that a benefit.

As an irrigation planner, I want the rain probability by 6 a.m., so that I can skip watering.

The **acceptance criteria** are the tests that say the story is done. The class example stacks four different kinds of test on purpose:

- AUROC at least 0.75. That is a model test.
- Completeness at least 95%. **Completeness** is the share of required values that are present. That is a data test.
- Humidity between 0 and 100. That is a contract test, and it is the reason the cleaning lab has a range rule.
- A bias report attached. That is a release test.

If the notebook ran and the humidity rule failed, the story is not done.

A **data contract** is the written promise between the people who produce the table and the people who train on it: column names, types, ranges, freshness, and what counts as a breaking change. The slides mention Jira or Azure Boards for the work and Confluence for the document. The tool is not the lesson. The promise is. A renamed humidity column should fail a job, not silently train on an empty feature.

---

## 🟡 Frameworks are checklists

A framework will not pick your model. It reminds you of a hole.

**AWS Well-Architected**, with the machine-learning and generative-AI lenses, is a review across security, reliability, cost, and operations, plus questions those lenses add for models. Use it to ask who can read the raw bucket, and what a bad endpoint costs per hour, before launch.

**CRISP-DM** is the Cross-Industry Standard Process for Data Mining: business understanding, data understanding, preparation, modeling, evaluation, deployment. It is the older sibling of the AI lifecycle from Module 1. If you have done Module 1, you have already met it under another name.

**MLOps maturity** describes how repeatable the operation is. The bottom is a notebook on one laptop. Higher up are scheduled pipelines, tests, and continuous training. **CT** is continuous training, retraining when data or a schedule says so. **CD** is continuous delivery, a tested path from a new model version to a deployed endpoint. Do not claim a maturity level you do not have. “We have a notebook” is a respectable level 0 if you say it plainly.

**NIST AI RMF** is the National Institute of Standards and Technology AI Risk Management Framework. Four verbs: govern (write the policy), map (describe the context and the risks), measure (the metrics and the tests), manage (act on the measurements). It is the responsible-AI pillars turned into a management cycle.

Quoting a framework in a design document, without answering who, what, why, and where, is how this class fails. Use the checklist to find a missing answer. Then write the answer in ordinary language.

---

## 🟡 The batch picture on AWS, and who owns which half

```
Sources → Ingest (Glue or Firehose) → S3 raw / clean / curated, plus the Glue Catalog
       → Prepare (DataBrew or Data Wrangler) → SageMaker training
       → Model Registry → Endpoint or Batch Transform → Monitor
```

Data engineering owns the top, through the curated zone. MLOps owns the bottom, from training onward. They meet at curated data. If training reads from `raw/`, every model reruns yesterday’s defects, and the two teams do not share a contract.

The names, once each:

| Name | What it is for |
|------|----------------|
| Glue | Serverless data jobs, and the catalog of tables |
| Firehose | Delivers a stream into S3 as files, without you running a server |
| Glue Catalog | The directory: table name, columns, and the S3 location. Athena and others need it. |
| DataBrew | Visual data preparation |
| Data Wrangler | SageMaker’s data-preparation interface, closer to the training environment |
| SageMaker | Train, register, and serve models |
| Model Registry | A versioned list of models and whether each one is approved. Production points at a version, not at “whatever was in the notebook.” |
| Endpoint | A live API that scores a request |
| Batch Transform | Score a whole file, then stop. This is the 6 a.m. weather path. |
| Curated zone | The feature-ready, versioned table. The handoff. |

---

## 🟡 RAG, drawn as a path rather than as a product

Retrieval-augmented generation is what we promised in Module 1 when we said a language model is not a database.

```
Documents in S3 → cut into chunks → embed each chunk → store the vectors
User question → embed the question → retrieve the top k chunks
→ prompt the LLM with those chunks and a guardrail → answer, with citations
```

A **chunk** is a piece small enough to embed and to fit in the prompt, often a few hundred tokens, with some overlap so a sentence cut in half can still be found. **Top-k** means “return the k nearest chunks.” Too small and you miss the paragraph that had the exception. Too large and you bury the model in noise and cost. A **citation** is the passage id or the link. Without it, the answer collapses back into an ungrounded paragraph that happens to sit next to real documents.

**Guardrails** filter the question and the answer, and they include the instruction to say “not in the documents” rather than invent. **Bedrock** is AWS’s managed foundation-model service. **Knowledge Bases** is its packaged version of the left half of the diagram: ingest, chunk, embed, retrieve. Packaging does not clean a duplicated policy or a wrong bulletin. A cited wrong sentence is still wrong. You own the quality of what goes in.

---

## 🟡 The ladder from a language model to a crowd of agents

Move one rung to the right only when the rung you are on cannot do the job. Each rung adds a behavior you must test.

1. **The LLM alone** answers from its training. The risk is hallucination. Stay here to draft, with a reviewer.
2. **Plus RAG** answers from your files. The new risk is retrieving the wrong chunk and sounding sourced.
3. **Plus tools** calls a calculator, a database, or a weather API. The new risk is a bad argument that changes real data.
4. **An agent** chooses the next tool call in a loop, because the steps are not known in advance. The new risks are runaway loops and actions nobody can replay.
5. **Several agents** split research, critique, and execution. The new risks are cost, contradiction, and no single owner.

**MCP**, the Model Context Protocol, is a way for an agent to discover and call tools. **A2A**, agent-to-agent, is a way for agents to hand work to each other. They are interfaces. They do not make the action safe. Do not take an agent loop on to summarize a PDF you could have chunked.

---

## 🟡 Three ways to sit on a cloud

**One vertical stack**, for example all of AWS. You move fast because S3, Glue, and SageMaker already know each other. You pay in **lock-in**: leaving means rewriting the pipeline. The labs are this pattern, so you can finish them.

**Tools first.** You start from a required capability, such as managed labeling, and you pick the place that has it. You then operate a mixed estate.

**An open-source core.** Kubernetes, MLflow, Airflow, and DVC run on more than one cloud. **Kubernetes** (K8s) runs containers. **Airflow** schedules pipelines. **MLflow** tracks experiments and models. **DVC** versions large files. You gain portability and you pay by operating the upgrades, the scaling, and the security yourself.

None of the three is a moral position. Write down which one you chose and what you are giving up. That note is an architecture decision.

---

## 🟢 Picking a model family from the shape, one more time

Tables start with classical ML or boosting. Sequences, if a table with lag features is not enough, can use an **LSTM** (long short-term memory, an older sequence network) or a transformer. Text uses embeddings for classification and an LLM for writing or for question answering over documents. Images use a CNN or a ViT.

For a short weather table, the LSTM is not the start. Lag features and a boosted tree are the start. Shape beats the headline, because the shape tells you what a row means and which leakage is possible.

---

## Checkpoint

You have only the forecaster bulletins, and the product needs each bulletin tagged clear, cloudy, rain, fog, or storm. Would you start with a language model or with embeddings plus a classifier? Say what you would be unable to measure if you chose the other one.

Start with embeddings and a classifier. You have labels, so you can measure precision and recall and you can compare annotators. A language model is the right tool when the deliverable is a new paragraph, such as a briefing, and even then a person or a citation stays on it. If you start with generation, you have nothing stable to score against the five classes. You have a different paragraph every time.

**Next class:** [02-data-collection-ingestion-lifecycle.md](02-data-collection-ingestion-lifecycle.md)
