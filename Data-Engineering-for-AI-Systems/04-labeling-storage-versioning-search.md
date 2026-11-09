# Class notes — Labels, the lake, versions, and finding text by meaning

*Module 2, Session 2, and Labs 3, 4a, and 4b.*

A cleaned table still has no meaning for “the right answer,” no promise you can restore yesterday, and no way to find a bulletin that does not share your keywords. This class treats those as products. A column named `label` is not automatically ground truth. A file you overwrote is not a version. A keyword search is not an understanding of the sentence.

---

## 🟢 A label is something you manufacture on purpose

In supervised learning, the label is the `y` you are willing to treat as correct. For rain it might be “the gauge exceeded 1 mm the next day,” which is a measurement. For a bulletin it might be the category an expert assigned, which is a judgment. Measurements and judgments both get called ground truth, and they do not deserve the same trust. A gauge can be broken. Two experts can disagree. The work of this class is to make the judgment reliable enough to train on, and to know how reliable it was.

If the labels are systematically wrong, the loss still falls. The model is a diligent student of the mistake. Changing the algorithm does not fix an instruction that marks “no rain expected” as rain. You fix the instruction, then you retrain. That is why labels are a product: someone defines them, writes the guide, measures agreement, and versions the result. Unsupervised learning, which looks for structure without a `y`, is not the path for these decisions. Most of this course is supervised, so the label workflow is part of the model.

---

## 🟡 Four ways to produce a label, and why real projects mix them

**Manual experts.** A person who knows the domain labels the item. Slow, expensive, and still dependent on a written guide, because experts disagree too. This is how you build a **gold set**, a small trusted set used later to score everyone else, including the rules.

**A crowd.** Many non-experts follow the guide. Fast when the task is obvious, and noisy when the guide is vague. Give the same item to more than one person. That repetition is **redundancy**. If they disagree, you have found a hole in the guide or a hard case, which is what you wanted to find.

**Synthetic.** A program creates a labeled example: a rotated image, a rendered scene, an LLM sentence. The label is only as true as the generation rule. Rotation often preserves a class. A generated sentence often does not.

**Programmatic, also called weak supervision.** Rules vote. You can label a million rows today. You can also be systematically wrong on every row the rule misunderstands. The speed is the attraction and the danger.

The design that actually gets used is a hybrid. Rules take the easy bulk. Humans take the cases where the rules abstain or conflict. Experts adjudicate disagreements and refresh the gold set. Picking one cell of the table for the whole project is how the budget and the quality both fail.

---

## 🟡 The quality workflow, and what kappa is doing

Five steps.

1. Write **guidelines** with the definition of each class, positive examples, and the edge cases that cause fights. “No rain expected.” “Drizzle.” “Rain nearby but not at this station.” If it is not in the guide, each annotator invents a policy.
2. Run a **pilot** of 20 to 50 items with at least two annotators. You are testing the guide, not the people.
3. Measure agreement with Cohen’s **kappa**.
4. **Adjudicate**: a more expert person resolves the disagreements, and those resolutions become new examples in the guide.
5. **Scale**, with gold questions hidden in the work and with spot audits afterwards. Quality drifts once the pilot is over. A gold question is an item whose answer you already know, slipped into the queue so you can see who is clicking without reading. A spot audit is a random review after you have scaled.

**Why not raw percent agreement?** Suppose 90% of bulletins are “clear.” Two annotators who always say “clear” agree 90% of the time and have demonstrated nothing about the rare storm. Percent agreement is flattered by the majority, the same way accuracy is flattered by the majority. Kappa subtracts the agreement you would expect from chance.

The intuition we need, not the full formula: 1.0 means they agree beyond what chance would produce. 0 means they agree about as often as chance. Below about 0.6, the course says to repair the guidelines rather than to blame the annotators. The pilot exists so you discover that 0.6 before you have paid for the full set.

Lab 3, on the simulated bulletins, lands near kappa **0.78**, with about **13.5%** of bulletins still needing adjudication. 0.78 is usable and not perfect. Budget for that 13.5%. Do not plan a project that assumes one pass and no expert queue.

---

## 🟡 Programmatic labeling, including the negation bug

A **labeling function** is a small rule that either returns a class or **abstains**. Abstaining is a feature. A function that always votes spreads its mistakes onto every row. `lf_fog` might look for “visibility poor” and otherwise refuse to vote. Several functions vote. A majority becomes the label. Rows where everyone abstains, or where the votes conflict, go to a person.

The bug in the lab is the lesson. `lf_rain` matches the substring “rain,” so it matches “**no** rain expected” and votes the opposite of the sentence. Negation handling means the rule understands “no,” “not,” and “without,” at least well enough to abstain. This is also why a gold set stays in the loop. You do not discover this bug by staring at the rule’s accuracy on rows the rule itself labeled.

The illustrative savings: rules cover about 80% of rows at about 94.6% accuracy against the gold checks, and the rest go to humans. The cost falls because expert time is the expensive input, not because the rules became ground truth. Treat the percentages as lab math that shows the shape of the trade, and keep the audit.

---

## 🟡 Two labeling products

| | Label Studio | SageMaker Ground Truth |
|--|--------------|------------------------|
| Where it runs | Your laptop or an EC2 machine you manage | Managed inside AWS |
| Why you pick it | Flexible configurations, and it can leave AWS | A workforce, and the same permissions system as the rest of the account |
| What you pay | You operate the machines | You pay per labeled object |

**EC2** is Elastic Compute Cloud, a virtual machine. **IAM** is Identity and Access Management, AWS’s permissions. Ground Truth can use the same identities and the same private buckets. Label Studio will run wherever you install it, including a laptop that should not be the production labeling service.

The **config** is the user interface plus the **label schema**, the allowed labels and the type of task. In the lab the schema is the bulletin categories clear, cloudy, rain, fog, and storm, written as **XML**, a markup of tags. The format is incidental. What matters is that the schema matches the guide. A free-text box where the guide promised five categories will produce a sixth category called whatever the annotator typed.

---

## 🟡 Which store answers which question

Start with a lakehouse. Add a warehouse when many people run the same heavy aggregates every day. Do not start with the warehouse because it sounds more serious.

| You need | You use | The question it answers | The question it is bad at |
|----------|---------|-------------------------|---------------------------|
| Files and the zones | S3 | Where is this version of the file? | Thousands of tiny row updates per second |
| A directory of tables | Glue Catalog | What are the columns of `weather_clean`, and where do they live? | Storing the rows themselves |
| SQL on the files | Athena | What was the mean humidity in May, without running a cluster? | A hundred-person dashboard that repeats the same scan all day |
| Heavy, repeated BI | Redshift Serverless | The same aggregates, many times a day | The first landing place of raw files |
| Relationships | A graph: Neptune or Neo4j | Which routes connect to this alert? | A plain monthly average |
| Similarity | A vector index, or OpenSearch k-NN | Which bulletins are closest in meaning? | An exact total |

**SQL** is Structured Query Language. A **lakehouse** is files plus a catalog plus SQL. **BI** is business intelligence, the dashboards. **Redshift Serverless** is a data warehouse that starts without you managing servers. **Neptune** is AWS’s graph database. **Neo4j** is a widely used graph database. **k-NN** is k nearest neighbors. The store follows the question. Embedding a station id in order to compute a monthly mean is the wrong store and the wrong representation.

---

## 🟡 Four habits that separate a lab upload from a pipeline

**Idempotent.** Running the job twice has the same result as running it once. If a Glue job fails halfway and you rerun it, Monday must not be appended twice. Write a day’s output to a deterministic partition and replace that partition, or deduplicate on the primary key. “Append forever” is not idempotent.

**Schema evolution.** Columns change. A new `wind_gust` should not break old queries. The catalog records the new schema. Readers that name their columns keep working. Readers that assume “the fifth field in the CSV” do not. This is another reason the curated form is Parquet with names, not a positional text file.

**Late data.** **Event time** is when the weather happened. **Arrival time** is when the file landed. A station that was offline uploads yesterday this morning. A **lateness window** is how long you wait before you call a day’s partition closed. Without it, late rows are dropped, or they rewrite history without a version.

**Backfills.** Reprocessing an old period, usually because the cleaning rule improved, writes a new version. Overwriting last quarter’s curated table erases the data the previous model was trained on. You can no longer explain that model. The model card has to point at a version that still exists.

---

## 🟡 Versions: the bucket remembers, and Git does not hold the bytes

Turn on **S3 versioning** before you need it. After it is on, a new upload of `weather_clean.csv` does not destroy the previous bytes. It adds a version id. You can list versions and download the old one. Turning it on later does not resurrect what you already overwrote. Versioning is a safety net and an audit trail. It is not a substitute for raw, clean, and curated.

**DVC**, Data Version Control, solves a different problem. Git becomes the wrong tool if you commit a multi-gigabyte CSV. DVC puts a small pointer file in Git and the **blob**, the file contents, in S3. Checking out an old commit and then pulling DVC restores the data that commit was made with.

The reproducibility line, which is the same line as Module 1 with the data filled in:

```
a rerunnable experiment = a code commit + a data version + the hyperparameters
```

Any one of the three, alone, is a rumor.

Lab 4a asks you to prove the mechanism. Enable versioning. Upload `weather_clean.csv` twice. Restore the older version. Create an Athena table through the Glue Catalog. Run a count. Note how much data was scanned. The count checkpoint is about **5,466** rows, the same rows the cleaning lab produced. If Athena disagrees, the table points at the wrong key or the wrong version. The scanned-bytes number is the lesson hiding in a small file: Athena’s price follows how much data the query reads, not how many rows come back. `SELECT *` on a wide CSV reads everything. Parquet and partitions, next class, are how that number falls. See the number while the data is still small enough that the bill cannot distract you from the mechanism.

---

## 🟡 Search by meaning

The pipeline of Lab 4b:

```
text → an embedding model → a vector → an index of nearest neighbors
```

**Titan** and **Cohere** are embedding models available through Bedrock in the lab. The brand is not the lesson. The lesson is that the stored chunks and the query must be embedded with the same model. Vectors from two models do not live in the same space. You cannot mix them and call the nearest neighbor meaningful.

**Cosine similarity** says two vectors point in a similar direction, which for embeddings means the texts are similar in the way the model was trained to notice. The index finds the nearest ones without a linear scan of every row. This beats `grep` on the bulletin “dense fog, visibility 50 m” when the user typed “hard to see the road.” The characters do not match. The meaning does. Embeddings also group things you did not intend. You evaluate search quality. You do not assume closeness is relevance.

**CLIP**, Contrastive Language–Image Pretraining, trains an image encoder and a text encoder so that a matching image and caption land near each other. A radar image and the phrase “band of heavy rain” can then share a space. That is multimodal search, the retrieval version of the fusion idea from the first class.

---

## 🟢 A graph, when similarity is the wrong hop

A **node** is a thing: a station, an alert, a route, a customer. An **edge** is a typed relationship: this station issued this alert, this alert affects this route. The useful query is a path. “Who is affected by this storm?” may be badly answered by similar paragraphs and well answered by alert, then route, then customer.

**GraphRAG** retrieves by walking edges, not only by finding similar chunks. You still want the source text, because the words of the alert matter. The graph is only as good as the edges. A missing “affects route” link looks like a confident empty answer. Building those edges, deduplicating stations, and versioning the graph is a data-engineering job, not a prompt.

---

## What to remember without the slides

Labels need a guide and a measurement of agreement that is not flattered by the majority class. Rules plus humans beat either one alone on cost and on the negation bugs. The store follows the question. Version the code, the data, and the settings together, or you cannot explain the model you shipped.

**Next class:** [05-trust-bias-ood-explainability.md](05-trust-bias-ood-explainability.md)
