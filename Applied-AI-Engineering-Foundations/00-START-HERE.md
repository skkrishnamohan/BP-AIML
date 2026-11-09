# Module 1 — Start here (you’re not behind)

If class felt like jargon soup, that’s normal. This module stack is designed so you can go **slow first**, then **deepen**. Each later file explains a topic as **what**, **why**, **how**, **where**, plus the **definition**, everyday **meaning**, and **abbreviations**.

## What Module 1 is really about

You are learning to think like an **AI engineer**, not a mathematician.

**What:** An AI engineer turns a business decision into data, a model, a measurement, and a running system.

**Why:** A clever model that answers the wrong question, leaks future data, or cannot be explained will fail in production even if a notebook looks perfect.

**How:** You choose among rules, machine learning, and generative AI; you split data honestly; you measure the errors that actually cost money; you keep humans responsible for the outcome.

**Where:** Banks (loan and fraud), weather products, insurance pricing, support chatbots, and any team that says “we should add AI.”

- When to use **rules** vs **ML** vs **GenAI**
- How **data (X)** and **answers (y)** become a **model**
- How to **evaluate** without fooling yourself
- How **AI SDLC** differs from normal software

## Words you will meet in every file

| Term | Expansion | Plain meaning |
|------|-----------|---------------|
| **AI** | Artificial Intelligence | Software that performs a task we associate with judgment |
| **ML** | Machine Learning | The program learns a pattern from examples instead of only following rules you typed |
| **DL** | Deep Learning | ML that uses many stacked layers, usually a neural network |
| **GenAI** | Generative AI | Models that create new text, images, audio, or code |
| **LLM** | Large Language Model | A large GenAI model specialized in language |
| **SDLC** | Software Development Life Cycle | The path from idea to running software and back |
| **MLOps** | Machine Learning Operations | The engineering practice of training, deploying, and watching models the way DevOps watches apps |
| **X** | Features | The inputs the model is allowed to see |
| **y** | Label or target | The answer you want the model to predict |
| **KPI** | Key Performance Indicator | The business number that tells you the system helped (approvals, fraud caught, water saved) |

**Definition vs meaning:** A definition is the precise classroom meaning. The meaning is what you should picture. Example: the definition of a **feature** is an input variable. The meaning is “a fact the model is allowed to look at when it decides.”

## Read in this order

| # | File | You’ll understand… |
|---|------|-------------------|
| 1 | [01-from-zero-ai-ml-dl-genai.md](01-from-zero-ai-ml-dl-genai.md) | The “concentric circles,” history, industry problems |
| 2 | [02-rules-learning-and-ai-sdlc.md](02-rules-learning-and-ai-sdlc.md) | Traditional vs AI lifecycle, 9 phases |
| 3 | [03-how-a-model-learns.md](03-how-a-model-learns.md) | Neuron, weights, logistic regression, gradient descent |
| 4 | [04-classification-regression-and-metrics.md](04-classification-regression-and-metrics.md) | Task types, accuracy trap, precision/recall, confusion matrix |
| 5 | [05-python-ml-workflow-and-pitfalls.md](05-python-ml-workflow-and-pitfalls.md) | sklearn pipeline, train/test, leakage, overfitting |
| 6 | [06-genai-prompts-responsible-ai.md](06-genai-prompts-responsible-ai.md) | LLMs, prompting, responsible AI pillars |
| 7 | [07-case-studies-quizzes-and-coding-lab.md](07-case-studies-quizzes-and-coding-lab.md) | Bank loan case, quizzes, insurance coding problem |

## Level guide

- **Basics** — sections marked 🟢. Read the definition and the analogy. Skip formulas if they blur; come back after the story makes sense.
- **Intermediate** — 🟡. This is where engineering choices live: which metric, which split, which tool.
- **Toward advanced / exam-ready** — 🔴. Leakage, cross-validation, and production failure modes.

Skip 🔴 on first pass if overwhelmed; come back after Module 2.

## How to read one file

1. Read the opening idea out loud in your own words.
2. For each table, cover the right column and say the meaning before you look.
3. Answer the checkpoint without scrolling to the next file.
4. If a short form appears (TP, RMSE, RLHF), expand it before you move on.

## One sentence to remember

> **AI learns patterns from past examples; your job is to define the problem, protect the data, and measure whether the pattern is real.**

Next file: [01-from-zero-ai-ml-dl-genai.md](01-from-zero-ai-ml-dl-genai.md)
