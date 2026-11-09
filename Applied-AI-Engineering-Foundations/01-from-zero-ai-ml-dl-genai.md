# Class notes — From zero: AI, ML, DL, GenAI

*Week 1. Student guide, slides, and the readings “AI vs ML vs DL,” “Historical Context,” and “Current AI Landscape.”*

Today’s class is a sorting class. People walk in using the word AI for five different machines, and then they pick a tool because it is fashionable. By the end you should be able to take a one-line business request and say which machine it is, why the others would be irresponsible, and what kind of answer that machine is even capable of giving.

---

## 🟢 The picture we drew first

Draw four nested circles. Do not draw them as a timeline. A timeline makes students think the old circles died. They did not. GST is still a rule. A credit score is still classical ML. A chatbot is GenAI. They are alive on the same day, and the engineer’s first job is to point at the right circle.

```
+------------------------------------------------------+
| AI: a machine doing a task that looks like judgment  |
|   +----------------------------------------------+   |
|   | ML: the rule was estimated from examples     |   |
|   |   +--------------------------------------+   |   |
|   |   | DL: many layers learn the clues      |   |   |
|   |   |   +------------------------------+   |   |   |
|   |   |   | GenAI: the output is new     |   |   |   |
|   |   |   | content, not only a label    |   |   |   |
|   |   |   |   +----------------------+   |   |   |   |
|   |   |   |   | LLM: GenAI for words |   |   |   |   |
|   |   |   |   +----------------------+   |   |   |   |
|   |   |   +------------------------------+   |   |   |
|   |   +--------------------------------------+   |   |
|   +----------------------------------------------+   |
+------------------------------------------------------+
```

Read the picture from the outside.

**Artificial Intelligence** is the campus. A recommendation, a fraud alert, a chatbot, and a hand-written eligibility rule that a product manager calls “our AI” can all sit on this campus. The word alone does not tell you how the decision is made. If a vendor says “AI-powered GST calculator,” ask them to open the circle. You will often find a multiplication.

**Machine Learning** is one department. The machine learns from data instead of only following rules a person typed. “Learns” is not a metaphor. It means there is a set of old examples, each example has an input and a known answer, and a procedure adjusts internal numbers until the mistakes on those examples get smaller. No examples, no learning. A coin toss has inputs and has no learnable answer.

**Deep Learning** is a method inside that department, not a synonym for “good ML.” Many layers of a neural network build the clues themselves. A classical model wants you to hand it a column called “distance from home.” A deep vision model starts from pixels and discovers edges, then parts, then objects, because you could not write those columns by hand. Deep learning is hungry: it wants large data and, usually, a GPU. On a small table of age and income it is often a worse engineering choice than a simple model you can explain.

**Generative AI** is the lab that creates a new artifact. This is the circle students collapse. A model that says “spam” or “not spam” is doing discrimination in the technical sense: it discriminates one class from another. A model that writes the email is generative. Most modern GenAI is deep learning, but plenty of deep learning is not generative. A face detector does not need to invent a new face.

**Large Language Models** are generative models for language. The usual architecture is the **Transformer**, from the 2017 paper *Attention Is All You Need*. You do not need the paper today. You need the consequence: the model’s native act is “given the text so far, what piece of text comes next?” It is not “look up the paragraph in a trusted file.” If you want the trusted file, you have to build the lookup around the model. That lookup is RAG, and it belongs to Module 2.

Memory line from the board: AI is the campus, ML is one department, deep learning is a method, GenAI is the lab that makes new things, and an LLM is one bench in that lab.

### Two words we stopped to define because the class mixed them up

**Discriminative** means the model answers “which bucket is this?” Fraud or legitimate. Rain or no rain. Species of iris.

**Generative** means the model answers “what new thing could come next?” A sentence, a picture, a SQL query.

A weather product can contain both. The number P(rain) is discriminative. The paragraph “a band of rain reaches the station after 4 p.m.” is generative, and somebody still has to check it against the number.

---

## 🟢 Three ways a model decides

The word **model** is used so loosely that we pinned it. A model is a function from an input to an output. Input, process, output. The only interesting question is where the process lives.

| Kind | Where the logic lives | The picture |
|------|----------------------|-------------|
| Classic rules | In source code a human can read | A traffic cop with a written manual. Red means stop. The cop does not learn red. |
| Classical ML | In numbers estimated from examples, using clues a human designed | A child who has been shown many dogs and can now say “dog” on a new street. Someone still chose which photos to show. |
| Deep learning | In numbers across many layers that build the clues | Vision assembled in stages: edges, then shapes, then a face. You did not name the edge. |

A **rule** is an instruction you can finish writing. “If the password has fewer than 12 characters, reject it.” Same password, same answer, next year, unless a person edits the code. That property is called **deterministic**.

**Learning** is what happens when you cannot finish the instruction, but you can show old cases. The internal numbers are **weights**. They are not opinions you type. A training procedure moves them.

A **feature** is a clue you designed: distance from home, hour of day, income. A **representation** is a clue the network discovered: “this patch of pixels is an edge.” Classical ML lives or dies by the features. Deep learning earns its cost when the representation cannot be designed by hand.

We stopped on a question from the second row. “If deep learning is more advanced, why does the faculty keep saying start simple?” Because advanced describes the machinery, not the fitness for the decision. A table with five columns and a legal need to explain a rejection is a bad place for a network you cannot read. A radar image is a bad place for five hand-written if statements. The shape of the data chooses the machine. The brochure does not.

---

## 🟢 The five situations from the quiz, taught slowly

The faculty rule, which is worth copying in the margin: do not use ML when about five IF/ELSE rules would do the job. Use ML when the pattern is too tangled, or changes too quickly, to keep writing by hand.

### 1. GST is 18% of the price

**GST** is Goods and Services Tax. The rate is published. The output is `price × 0.18`. There is no example to learn from, because a different answer is not a pattern. It is a bug. An LLM that says 18 today and 12 tomorrow is not being creative. It is failing a deterministic law. This is rules. It stays rules even if someone wraps the multiplication in a chat window.

### 2. A UPI fraud shield in 15 milliseconds

**UPI** is the Unified Payments Interface, the real-time payment rail. **ms** is a millisecond, a thousandth of a second. Fifteen of them is the budget for the whole score, because the payment is waiting.

Why not rules? A criminal who reads “block amounts above ₹50,000 from a new device” will send ₹49,000 from a device that is two days old. The signal is fifty weak clues at once: amount, hour, distance from the customer’s usual place, device age, merchant category. No one can write the combination, and the combination moves every month. That is predictive ML. The model emits a probability. A threshold, which is still a rule, turns the probability into block, review, or allow.

Why not an LLM? It does not produce a calibrated probability on a 15 ms budget, and a fluent sentence is not a control. You may later ask a model to draft the analyst’s note. The decision itself stays a small scored model.

### 3. Summarize a 60-page RBI circular

**RBI** is the Reserve Bank of India. The input is long unstructured text. The useful output is new text: a brief a person will read. There is no single numeric y. This is GenAI, and the class of GenAI is an LLM. The engineer’s leftover duty is that the brief can be wrong in a confident voice. A person who owns the policy still checks it, or the system is built to quote the circular rather than remember it.

### 4. Password must be at least 12 characters and contain a digit

You can write this in one boolean expression. Learning it from examples would mean hoping the model notices the rule, and then auditing a probability instead of a line of code. Rules.

### 5. Tomorrow’s grid load in megawatts

**MW** is megawatt, a quantity of electrical power. The answer is a number on a scale, not a bucket. Predicting a number is **regression**. The pattern lives in history, weather, and the calendar. It is too messy for a formula you would trust, and it is not a writing task. Regression ML.

The habit to take out of these five: read the output first. A rate in a statute, a yes/no policy, a probability, a quantity, or a new paragraph. The output picks the circle. The industry (banking, weather, government) does not.

---

## 🟡 Why the same prompt gives two different paragraphs

An LLM does not store your answer. It stores, in a very large set of weights, a talent for guessing the next **token**. A token is a chunk of text, often a piece of a word. “Unbelievable” may be three tokens. The model looks at the tokens so far and produces a probability for every token in its vocabulary. Then it picks one. That pick is called **sampling**. Then it does it again, and again, until it stops.

This is why two calls differ. The most likely token is not the only legal token. Sampling sometimes takes the second or the tenth.

**Temperature** is the dial on how peaked that list of probabilities is. It is not a dial on truth.

- Temperature near 0 means “almost always take the top token.” Use it when the output is SQL, JSON, or code, where a creative comma is a broken program. **SQL** is Structured Query Language.
- Temperature 0.7 to 1.0 means “let less likely tokens win more often.” Use it for a brainstorm, not for a refund policy that must be the same at 9 a.m. and at 9 p.m.

Worked picture. Suppose the next-token probabilities are: “rain” 0.60, “drizzle” 0.30, “snow” 0.10. At temperature 0 you will see “rain” almost every time. Raise the temperature and “drizzle” shows up often enough that two summaries of the same bulletin disagree. Neither temperature checks whether it rained.

The engineering sentence: ML and GenAI are **probabilistic**. They return a score or a sample, not a proof. Put three things beside them.

- A **threshold**: block only if the fraud probability is at least 0.92.
- A **guardrail**: reject the output if it is not valid JSON, if it has no citation, or if it is on a disallowed topic.
- A **human review** where the cost of being wrong is a person losing money, time, or a right.

**Hallucination** is the name for a fluent sentence that is not grounded in a source you trust. Low temperature does not cure it. A lookup into a real document, or a person, does.

---

## 🟢 Four milestones, and why the job became operations

The history lecture is not nostalgia. Each stage moved the logic out of a file a developer can read and into an artifact that has to be fed, deployed, and watched.

**1. Rule-based systems, roughly before 2010.** Predictable and explainable. You can point at the line. They become brittle when the world is mostly exceptions. Fraud rules are a clean example: each new attack adds a condition, the conditions interact, and nobody can tell you what the file will do on a Tuesday.

**2. Classical ML.** The exceptions are learned from rows. A person still decides which columns exist. That design work is **feature engineering**. Logistic regression, decision trees, and gradient boosting live here. They are the right start for tables.

**3. Deep learning.** The features for images, audio, and raw text are learned. The bill moves to hardware. A **GPU**, a graphics processing unit, does the huge matrix multiplications. A **CPU**, the ordinary processor, is often too slow to train a large network. You also need far more data, because the network is free to memorize if you do not give it enough reality.

**4. Foundation models and LLMs.** A **foundation model** is pretrained on a very broad collection so that many tasks can start from it instead of from empty weights. You either **prompt** it, which means giving instructions and evidence at the moment of the request, or you **fine-tune** it, which means continuing training on your smaller set. You usually do not train the base model yourself. The new operational problems are cost per call, leakage of private text into the prompt, and the fact that the model will answer even when your documents do not contain the answer.

**Drift** is why MLOps exists. Drift means the world moved away from the training examples. Last year’s spam no longer looks like this year’s spam. The Python file did not change. The correct answers did. A model that was approved in January can be quietly worse in July while the API still returns success. Watching for that movement is part of shipping the model, not a later hobby.

---

## 🟡 What we do before anyone says “which algorithm?”

The order on the board was:

**Problem → Decision → Data → System → Model**

AI is a candidate when the decision is high volume, repetitive, time-sensitive, expensive when wrong, or too tangled for a manual rule. Fraud sorting, clinical triage, demand forecasting, and defect detection fit. A one-off judgment with no past examples does not. A fair coin does not.

**Triage** means sorting cases so a human sees the dangerous ones first. The model ranks the queue. It does not become the clinician or the fraud analyst.

Walk a weather example in the order, so the order is not a slogan.

| Step | The question | A real answer |
|------|--------------|---------------|
| Problem | Whose pain? | A farmer waters, and then it rains. The water was wasted. |
| Decision | What action changes? | Skip irrigation when rain is likely enough. |
| Data | What can we know before the action? | Yesterday’s pressure, humidity, the radar. Not tomorrow’s rain. |
| System | Where does the score go, and by when? | A file by 6 a.m. Batch is enough. A chatbot is not the product. |
| Model | What family fits that data? | A tabular time-series model. The algorithm name comes last. |

If you name the algorithm at step 1, you will build a demo. The decision and the data are the parts you cannot casually replace.

---

## 🟢 Glossary we pinned before leaving

| Term | The sentence we wrote |
|------|------------------------|
| Feature, X | A fact the model is allowed to see when it decides. |
| Label or target, y | The correct answer on a historical row. Fraud or not. A price. A species. |
| Weight, w | A learned multiplier. It says how hard that fact pushes the decision. |
| Bias, b | The starting score before any fact is considered. Not social bias. |
| Training | The phase that is allowed to look at y and move the weights. |
| Inference | Using the frozen weights on a new row, when y has not happened. |
| Ground truth | The label you have decided to trust. A rain gauge, or an expert’s judgment after a written guide. |
| Algorithm | The procedure that moves the weights. |
| Architecture | The shape: one neuron, a tree, a transformer. |
| Parameter | A number the procedure learns. Weights and the bias term. |

The difference the exam likes: training may take an hour and is allowed to see the answers. Inference takes milliseconds and must not see the answer, because in production the answer is in the future.

---

## 🟡 A second pass: one product, three machines, so the circles stay separate

Take a bank’s mobile app on a Monday morning. Three features ship in the same release, and students will call all three “the AI.”

The GST line on a receipt is `amount × 0.18`. We can write the test before we write the code. If the amount is ₹1,000, the tax is ₹180, every time, in every city. There is no training set. There is no temperature. Wrapping it in a chat prompt would add a way for the answer to be ₹170 and sound confident. The machine is a rule. It lives in the outer circle only.

The “you may like this credit card” ranking is ML. Historical customers who looked like this applicant either took the card or did not. That yes/no is y. Age, income band, and recent product views, if they exist before the offer is shown, are X. The model returns a probability. A cutoff, still a rule, decides whether to show the banner. This is the ML circle. It is not generative. Nothing new is being written. A person is being scored.

The “explain this charge in two sentences” button is GenAI. The useful output is new language. The model must be handed the transaction row. If it is not handed the row, it will invent a merchant and a reason, because inventing the next tokens is the only act it has. Temperature near 0, a format (“two sentences, no advice”), and a human-visible source line are the controls. This is the inner circle. It does not replace the fraud score sitting next to it.

Write the three side by side in the margin:

| Button in the app | Output | Machine | What would make it the wrong machine |
|-------------------|--------|---------|--------------------------------------|
| Tax on the receipt | A legally fixed number | Rule | Any sampling |
| Card offer | A probability, then a yes/no | Predictive ML | Judging it by how pleasant the wording is |
| Explain this charge | A short paragraph | LLM, grounded on the row | Letting it answer with no row attached |

If you can fill that table for a new app without looking back at the five quiz rows, this class has landed.

## 🟡 Words students used loosely, and the sentence that replaces each one

**“The AI decided.”** An AI system produced a score or a paragraph. A person, or a rule someone wrote, turned it into an action. Keep those two steps separate or you will not know what to change when the action is wrong.

**“We will train ChatGPT on our data.”** Usually you will not. You will prompt a model, or you will retrieve your documents and put them in the prompt, or you will fine-tune. Training from empty weights is a different budget. Say which of the three you mean.

**“It is accurate, so it understands.”** Accuracy is a fraction of labels matched. Understanding is not what the metric measures. A model can match labels by using a leaked column and understand nothing you could defend.

**“Deep learning is the upgrade.”** It is a tool for raw signals where you cannot design the clues. On a five-column table it is often a more expensive way to overfit.

## Checkpoint

Answer these before you open the next file. Say them in sentences.

1. Is “predict an apartment price in rupees” classification or regression?
2. Why is 99% accuracy a misleading celebration when only 1% of payments are fraud?
3. Name one task that should stay a rule, and one that is genuinely GenAI.

When you have tried: (1) Regression, because rupees are a quantity, not a bucket. (2) A model that calls every payment legitimate is right on the 99% and catches no fraud. The easy majority fills the score. (3) GST or the password length is a rule. Summarizing the circular is GenAI. The full arithmetic of (2) is the metrics class.

**Next class:** [02-rules-learning-and-ai-sdlc.md](02-rules-learning-and-ai-sdlc.md)
