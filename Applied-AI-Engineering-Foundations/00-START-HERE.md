# Module 1 — Start here

If the first week felt like a list of words, that is because the slides are written for someone who already sat in the class. These notes are the class. Read them slowly. The later files assume you can say the words on this page in a full sentence.

## What this module is building

You are not being trained to invent a new algorithm. You are being trained to stand in front of a problem and say, in order:

1. Is this even a learning problem, or is it a rule somebody can write down?
2. If it is learning, what is the input, what is the answer, and what would count as a wrong answer that costs money?
3. How will we know the pattern is real, and not the model peeking at the answer?
4. Who is still responsible after the model gives a number?

That is the whole of Module 1. Everything else is a tool for one of those four questions.

An **AI engineer** here means the person who connects a business decision to data, a model, a measurement, and a running system. The mathematician cares whether the loss has a proof. You care whether a farmer, a bank, or a fraud analyst can act on the output tomorrow morning and whether you can explain the failure when they cannot.

## The words that will be used in every class

Write these out once. Later files will not stop to expand them every time.

**AI — Artificial Intelligence.** Software doing a task we would have called judgment if a person did it: flagging a payment, ranking a queue, drafting a paragraph. It is the name of the campus, not of one building.

**ML — Machine Learning.** A subset of AI. Instead of you typing every rule, the program estimates a rule from examples. The examples are old situations where you already know what the right answer was.

**DL — Deep Learning.** A subset of ML that stacks many simple calculations into layers, usually a neural network. It is how we deal with raw pictures, long text, and sound, where a human cannot sit and design every clue.

**GenAI — Generative AI.** Models whose job is to produce new text, images, audio, or code. A fraud score is not generative. A summary of a circular is.

**LLM — Large Language Model.** A large generative model for language. It predicts the next piece of text. It does not look a fact up in a book unless you build that lookup around it.

**SDLC — Software Development Life Cycle.** The path from “someone has a need” to “the software is running and we are watching it.” The AI version of this path is different because the behavior lives partly in the data.

**MLOps — Machine Learning Operations.** The discipline of training, releasing, and watching models with the same seriousness that operations teams watch an ordinary service.

**X — features.** The inputs the model is allowed to see at the moment of the decision.

**y — the label, also called the target.** The answer you attach to old examples so the model can learn. In production, y has not happened yet. That is the whole point of predicting it.

**KPI — Key Performance Indicator.** The business number that says the system helped: fraud losses, irrigation cycles skipped, applications reviewed in time. Accuracy is not automatically the KPI.

Keep one warning next to the letter **b**. In a neuron, bias means a starting score. In a dataset, bias means an unfair skew. In statistics, bias means a model that is systematically too rigid. They are three different complaints. Whenever a slide says “bias,” ask which one.

## The order of the classes

| Class | File | Leave the class able to… |
|-------|------|--------------------------|
| 1 | [01-from-zero-ai-ml-dl-genai.md](01-from-zero-ai-ml-dl-genai.md) | Place a problem in AI, ML, DL, or GenAI, and refuse the wrong tool |
| 2 | [02-rules-learning-and-ai-sdlc.md](02-rules-learning-and-ai-sdlc.md) | Tell a normal software lifecycle from an AI one, phase by phase |
| 3 | [03-how-a-model-learns.md](03-how-a-model-learns.md) | Walk one row through a neuron, a sigmoid, and a weight update |
| 4 | [04-classification-regression-and-metrics.md](04-classification-regression-and-metrics.md) | Build a confusion matrix and show why 99% can be a failure |
| 5 | [05-python-ml-workflow-and-pitfalls.md](05-python-ml-workflow-and-pitfalls.md) | Run the honest sklearn order and spot leakage in code |
| 6 | [06-genai-prompts-responsible-ai.md](06-genai-prompts-responsible-ai.md) | Say what an LLM was trained to do, and what you still own |
| 7 | [07-case-studies-quizzes-and-coding-lab.md](07-case-studies-quizzes-and-coding-lab.md) | Answer the Week 1 quizzes by pointing at the target, not the industry |

## How the notes are marked

- 🟢 is the part you copy even if you copy nothing else.
- 🟡 is the engineering choice: which metric, which split, which tool.
- 🔴 is the trap that makes a notebook look brilliant and a live system fail.

If you are tired, read every 🟢 section and every worked example. Come back for 🔴 before you touch a lab.

## One sentence from the front of the room

> AI learns patterns from past examples. Your job is to define the problem, protect the data, and measure whether the pattern is real.

Next class: [01-from-zero-ai-ml-dl-genai.md](01-from-zero-ai-ml-dl-genai.md)
