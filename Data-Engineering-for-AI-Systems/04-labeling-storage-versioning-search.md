# Session notes — Labels, lakes, versioning, vectors

*Module 2 Session 2 + Labs 3, 4a, 4b.*

A cleaned table still needs a meaning for “the right answer,” a place to live, a version you can roll back, and sometimes a way to search by meaning rather than by exact words. This file covers those four jobs.

---

## 🟢 Labels are a product

Ground truth = **supervised learning fuel**. Bad labels → bad model, no matter how fancy the algorithm.

**What a label is:** The `y` you are willing to treat as correct for training. For rain it might be “measured rainfall tomorrow exceeded 1 mm.” For a bulletin it might be the category an expert assigned. The label is not the prediction, and it is not automatically true just because a column exists.

**Why labels are a product:** Someone must define them, write instructions, measure agreement, and version them. That is design and operations work. Calling the column `label` does not make it ground truth.

**Ground truth** means the label you accept as the reference. Some targets are measurements (a rain gauge). Some are judgments (a forecaster’s category). Judgments need an agreement process because two honest experts can differ.

**How bad labels show up:** The loss decreases, the test score looks stable, and the errors are systematic. The model copied the labeling mistake. Changing the algorithm does not fix an instruction that says “mark ‘no rain expected’ as rain.”

**Where this connects to Module 1:** Supervised learning is defined by having `y`. Unsupervised learning looks for structure without `y`. Most of this course’s decisions are supervised, so the label workflow is part of the model.

---

## 🟡 Four labeling methods

| Method | Quality | Speed | When |
|--------|---------|-------|------|
| **Manual (experts)** | Highest | Slowest | Gold sets, hard cases |
| **Crowd** | Medium | Fast | Clear guidelines + redundancy |
| **Synthetic** | Varies | Fast | Augmentations, renders, LLM text |
| **Programmatic** | Noisy | Very fast | Rules / weak supervision |

**Real projects:** hybrid — automation for volume, humans for judgment gaps.

| Method | What it means | Why the quality column looks like that |
|--------|---------------|----------------------------------------|
| **Manual experts** | A person who knows the domain labels the item | Slow and expensive, and still needs instructions. This is how you build a **gold set**, a small trusted set used to score everyone else. |
| **Crowd** | Many non-experts follow a guide | Fast if the task is obvious. Quality collapses when the guide is vague. Give the same item to more than one person (**redundancy**) and compare. |
| **Synthetic** | A program or a generator creates labeled examples | The label is only as true as the generation rule. A rotated image may keep its label. An LLM sentence may not. |
| **Programmatic** | Rules or heuristics vote on a label. Also called **weak supervision** because the rules are noisy. | You can label a million rows today and be systematically wrong wherever the rule is naive. |

**Why hybrid is the real design:** Rules label the easy 80%. Humans label the abstentions and a random audit. Experts adjudicate disagreements and refresh the gold set. You do not pick one cell of the table for the whole project.

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

**What guidelines contain:** The definition of each class, positive examples, and the edge cases that cause fights (“no rain expected,” “drizzle,” “rain nearby but not at the station”). If it is not in the guide, annotators will invent a policy, and they will not invent the same one.

**What a pilot is:** A small paid rehearsal. You are testing the guide, not the annotators. Twenty to fifty items is enough to see confusion before you spend the full budget.

**Cohen’s kappa:** A measure of agreement between two annotators that subtracts the agreement you would expect by chance. Raw percent agreement lies when one class is common: two people who always say “clear” agree often and know nothing. Kappa near 0 means they agree about as often as chance. Kappa near 1 means they agree even after chance is removed. Below about 0.6, the course says to repair the instructions.

**Adjudicate** means a more expert person resolves the items where annotators disagreed. Those resolutions become new examples in the guide.

**Gold questions:** Items with a known answer slipped into the work so you can see who is clicking randomly. **Spot audits** are random reviews after scale-up. Both exist because quality drifts once the pilot is over.

**What the lab numbers mean:** Kappa around 0.78 is usable and not perfect. About 13.5% of bulletins still need an expert. Budget for that fraction instead of assuming automation or a single pass is enough.

---

## 🟡 Programmatic labeling (bulletins)

Pattern:

- Several **labeling functions** vote (`storm`, `fog`, `rain`) or **abstain**
- Majority vote → label
- **Abstain** rows → humans

**Bug example:** `lf_rain` matching “**No** rain expected” → add **negation handling** (lab task).

**Hybrid savings (illustrative):** ~80% rows by rules @ ~94.6% accuracy; rest to humans; large cost reduction vs all-human.

**What a labeling function is:** A small rule that either returns a class or abstains. `lf_fog` might look for “visibility poor” and otherwise refuse to vote. Abstaining is a feature. A function that always votes spreads its mistakes to every row.

**Why several functions:** One rule is brittle. A vote lets fog words and rain words compete. Where they conflict or all abstain, a human decides. That human queue is the point of the design.

**What the negation bug teaches:** The substring “rain” appears inside “no rain expected.” A naive match labels the opposite of the text. Negation handling means the rule understands “no,” “not,” and “without,” at least well enough to abstain. This is also why you do not fully trust programmatic labels without a gold set.

**How to read the savings numbers:** They are illustrative lab math. If rules cover about 80% of rows at about 94.6% accuracy, humans only read the hard remainder, and you still audit. The cost drops because expert time is the expensive part, not because the rules became ground truth. Keep the gold set real.

---

## 🟡 Label Studio vs Ground Truth

| | Label Studio | SageMaker Ground Truth |
|---|--------------|------------------------|
| Hosting | Your laptop/EC2 | Managed AWS |
| Strength | Flexible configs | Workforce + IAM integration |
| Pay | You manage infra | Per labeled object |

**Config = UI + label schema** (XML for bulletin categories: clear/cloudy/rain/fog/storm).

**EC2** means **Elastic Compute Cloud**, a virtual machine you manage. **IAM** means **Identity and Access Management**, AWS’s permissions system. Ground Truth can use the same identities and private data locations as the rest of the account. Label Studio is flexible and will happily run wherever you install it, including a laptop that is a poor place for a production labeling workforce.

**Label schema:** The list of allowed labels and the type of task (one class, many classes, a box, a text span). The **UI** is how that schema is shown to the annotator. **XML** is the configuration format used in the lab, a markup language of tags. The format is incidental. What matters is that the schema matches the guidelines: the same five bulletin categories, with no free-text surprise class.

**Where the choice is made:** A course lab or a company already inside AWS leans on Ground Truth. A research setup that must leave AWS leans on Label Studio. The quality workflow above does not change with the tool.

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

**What each store is for:**

| Store | What it is | The question it answers | Do not use it to |
|--------|------------|-------------------------|------------------|
| **S3** | Object storage | “Where is the file, and which version?” | Serve thousands of tiny row updates per second like a transactional database |
| **Glue Catalog** | Table metadata | “What columns does `weather_clean` have, and where is the data?” | Store the rows themselves |
| **Athena** | Serverless SQL over files in S3 | “What was the mean humidity in May?” without loading a cluster | Power a hundred-person dashboard that reruns the same scan all day |
| **SQL** | Structured Query Language | The language of that question | — |
| **Lakehouse** | Files plus a catalog plus SQL | The combination of the three rows above | — |
| **Redshift Serverless** | A cloud data warehouse that starts without you managing servers | The same aggregates, many times a day, by many BI users | The first place you land raw files |
| **BI** | Business intelligence | Dashboards and reports | Training-time feature logic you do not version |
| **Graph** | Nodes and edges. **Neptune** is AWS’s graph database. **Neo4j** is a widely used graph database. | “Which routes are connected to this station’s alert?” | A plain daily average |
| **Vector database / OpenSearch k-NN** | An index of embeddings. **k-NN** means k nearest neighbors. | “Which bulletins are closest in meaning?” | Exact financial totals |

**Why start with the lakehouse:** The lab data is files, the teams are small, and Athena answers the audit questions. Add Redshift when concurrency and repeated aggregates make scanning files wasteful. Starting with a warehouse does not remove the need for raw and clean zones.

---

## 🟡 Ingestion production habits

| Property | Meaning |
|----------|---------|
| **Idempotent** | Re-run job ≠ duplicate rows |
| **Schema evolution** | New columns don’t break readers (Glue Catalog) |
| **Late data** | Event time vs arrival time + lateness window |
| **Backfills** | Replay into **new version**, not in-place overwrite of history |

**Idempotent** means doing the job twice has the same result as doing it once. If a Glue job fails halfway and you rerun it, you must not append a second copy of Monday. The usual method is to write a partition deterministically (replace that day’s output prefix) or to deduplicate on the primary key.

**Schema evolution** means the columns change over time. A new `wind_gust` column should not make old queries crash. The catalog records the new schema. Readers that select named columns keep working. Readers that assume a fixed position in a CSV do not, which is another reason to prefer Parquet and named columns.

**Event time** is when the weather happened. **Arrival time** is when the file landed. A station that was offline can upload yesterday’s hours today. A **lateness window** says how long you wait before you consider a day’s partition closed. Without it, late rows are either dropped or they rewrite history quietly.

**Backfill** means reprocessing an old period, often because the cleaning rule improved. Write it as a new version. Overwriting last quarter’s curated table erases the data your previous model was trained on, and you can no longer explain that model.

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

**What S3 versioning does:** When versioning is enabled on the bucket, a new upload of `weather_clean.csv` does not destroy the previous bytes. It adds a version id. You can list versions and download the old one. It is not a substitute for zones. It is a safety net and an audit trail.

**What DVC adds:** Git becomes slow and wrong if you commit a multi-gigabyte CSV. DVC stores a small pointer file in Git and the bytes in S3. Checking out an old Git commit, then pulling DVC, restores the data that commit was made with.

**Blob** means binary large object, the file contents. **Hyperparameters** are the settings you chose before training, from Module 1.

**Why all three pieces:** Code without data can be a different experiment. Data without code is a file nobody can rebuild. Hyperparameters without either is a number in a slide. The triple is the minimum description of a run.

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

**What you are proving in the lab:**

| Step | What it teaches |
|------|-----------------|
| Enable versioning | The bucket can remember overwrites. Doing this later does not recover files already replaced. |
| Upload twice | Two version ids exist for one key |
| Restore | You can get the previous bytes back on purpose |
| Glue Catalog table | SQL needs a schema and a location, not only a file |
| Athena query | The lakehouse works, and the bytes scanned are the bill |

**Why “data scanned” is the lesson:** Athena’s price follows how much data the query reads, not how many rows it returns. `SELECT *` on a wide CSV reads everything. A later Parquet file with a partition on year lets the engine skip whole files. The lab wants you to see the scanned number while the data is still small.

**The 5,466 checkpoint** matches the cleaning lab. If Athena counts something else, the table is pointing at the wrong key or the wrong version.

---

## 🟡 Embeddings & semantic search (Lab 4b)

Pipeline:

```
Text → embedding model (Titan/Cohere) → vector → nearest-neighbor index
```

**Why:** “hard to see the road” matches **fog** bulletins without shared keywords (unlike grep).

**CLIP idea:** image + text encoders map to **shared vector space** (multimodal search).

**What an embedding model does:** It maps a piece of text to a vector of a few hundred or a few thousand numbers. Texts with similar meaning have vectors with a high **cosine similarity** (they point in a similar direction), even if they share no words.

**Titan** and **Cohere** are embedding-model providers available through Bedrock in the lab. The pipeline does not depend on those brand names. It depends on using the same model for the stored chunks and the query. Vectors from two different models are not comparable.

**Nearest-neighbor index:** A structure that finds the closest stored vectors without comparing the query to every row one by one. **k-NN** asks for the k closest.

**Why this beats grep:** `grep` finds characters. A person writing “hard to see the road” will not match a bulletin that says “dense fog, visibility 50 m.” The embedding space groups those meanings. It can also group things you did not intend, so you still evaluate search quality.

**CLIP** means **Contrastive Language–Image Pretraining**. One encoder reads images, another reads text, and training pulls matching image–text pairs together. A radar image and the phrase “band of heavy rain” can then live in one space. That is the search version of multimodal fusion.

**Where embeddings are the wrong tool:** Exact ids, sums, and thresholds. Do not embed a station id to compute a monthly mean. Use SQL for that.

---

## 🟢 Graph + GraphRAG (concept)

Nodes = entities (station, alert, route); edges = relationships.

Query paths like: *storm alert → routes affected → customers*.

**GraphRAG:** retrieval follows **edges**, not just similar text chunks.

**What a node and an edge are:** A node is a thing. An edge is a typed relationship: “station S01 issued alert A17,” “alert A17 affects route R3.” The query is a path, which is a poor fit for a single blob of text and a poor fit for a flat table when the connections hop several times.

**What GraphRAG adds to RAG:** Classic RAG retrieves chunks that are similar to the question. GraphRAG also walks relationships. “Who is affected by this storm?” may be badly answered by similar paragraphs and well answered by alert → route → customer. You still want the source text for the words of the alert.

**Why this is only as good as the edges:** A missing “affects route” link is invisible to similarity search too, but the graph makes the gap look like a confident empty path. Building the graph is a data-engineering job: extracting entities, deduplicating stations, and versioning the edges.

---

## Session 2 remember list

1. Labels need guidelines + agreement
2. Hybrid labeling (rules + humans) wins on cost
3. Pick store by question; **version everything**

**Next:** [05-trust-bias-ood-explainability.md](05-trust-bias-ood-explainability.md)
