# Class notes — Generative models, prompts, and what you still own

*Student guide on GenAI, RLHF, prompting, and the responsible-AI pillars.*

The earlier classes produced a probability you could put next to a label. This class produces a paragraph. The paragraph can be useful and it can be fiction. The engineering question does not change: what was the system allowed to know, why should anyone trust the output, and who answers when it is wrong?

---

## 🟢 What the model was actually trained to do

A generative model creates new content by repeating statistical patterns it saw in a huge collection of text, images, or audio. A large language model’s native act is narrower than the marketing. Given the tokens so far, predict the next token. A token is a chunk, often part of a word. Do that many times and you have a paragraph. Nowhere in that act is “consult a system of record.” If the weights happen to reproduce a fact that appeared often in training, the paragraph may be true. If they do not, the paragraph may still sound true. That failure has a name, **hallucination**: fluent text that is not grounded in a source you trust.

There are three stages people blur into “the AI.”

**Pretraining.** The model reads a massive corpus and practices next-token prediction. It picks up grammar, associations, and facts that were repeated. It does not know your company’s policy, and it does not know what happened this morning unless that text is placed in front of it. Pretraining is expensive and, for you, usually already done by the time you call an API.

**RLHF — reinforcement learning from human feedback.** People rank several answers to the same prompt. A reward model learns those preferences. The language model is then updated to prefer answers the reward model scores highly. This is why a chat model follows “be brief” and “refuse this” better than a raw pretrained model does. **Reinforcement learning** means the training signal is a reward, not only a single correct label. RLHF does not turn the model into a verified database. It turns it into a model that produces the kind of answer humans marked as better.

**Guardrails.** Filters and policies around the model, on the way in and on the way out. They refuse some requests and block some outputs. They are a layer, like a threshold on a fraud score. They are not a proof that everything which passes is acceptable. You still test them.

A worked distinction the class needed. Ask “what is the cooling-off period in the attached policy?” with the policy pasted underneath. The model can quote. Ask the same question with no policy, and the model will often invent a number and a section heading. The second answer is a sample from the weights, not a lookup. Low temperature makes the invention more repeatable. It does not make it a lookup. The lookup is a separate system, retrieval-augmented generation, which Module 2 builds properly. Until then, treat an ungrounded number as unverified.

**Context length** is how many tokens fit in front of the model at once. A 60-page circular may not fit. The practical consequence is chunking: you break the document into pieces and decide which pieces to show. Cutting a sentence in half can separate the word “no” from the word “rain.” Chunking is an engineering choice, not a formatting detail.

---

## 🟢 The three things a person still owns

The slide says the model does not take the job. A person who can aim it might. The line is easy to mock and worth unpacking, because “aim it” is three concrete duties.

**You own the ask.** The model completes a pattern. A vague pattern completes into a vague answer. Specifying the task, the audience, the format, and the constraint is the same kind of work as defining y in supervised learning. If you do not define the output, the model will, and it will sound finished.

**You own the trust decision.** The model does not know when it is guessing. Verification is outside it: run the code, open the circular, compare the number with the policy, or have a named reviewer sign. “It sounded right” is not a control.

**You own compliance.** **PII** is personally identifiable information: names, account numbers, phone numbers, anything that identifies a person. Pasting a customer’s full record into a public chatbot can be an incident even if the reply is helpful. A summary of an RBI circular is not itself the regulation. A loan denial still needs the approved decision process, not only a well-written paragraph.

Write the duties next to the fraud lesson. There, a human owned the threshold. Here, a human owns the source and the sign-off. The tool changed. The ownership did not.

---

## 🟢 A prompt, taken apart the way we did in class

The bad prompt was “write something about AI.” There is no task you can mark, no reader, and no constraint. The model must invent all three, and it will.

The structure from the student guide:

| Piece | What it is doing | The workshop example |
|-------|------------------|----------------------|
| Task | The verb and the artifact, specific enough to mark | Produce an agenda |
| Context | The facts that would change the artifact | Aerospace engineering leads, four hours, no vendor pitches |
| Clarifying invitation | Permission to refuse to guess | Ask me what you need before you draft |

A fuller prompt adds three more lines when the stakes are real.

**Format.** “A table with time, topic, and the deliverable of that session.” Format is how you stop a wandering essay.

**Evidence.** “Use only the attached brief. If a fact is not in the brief, write ‘not in the brief’ rather than supplying one.” This is the prompt version of forbidding leakage. You are telling the model which features exist.

**A check.** “Every session has one verb and one deliverable. There are no more than six sessions.” Now you can mark the output without rereading it for vibes.

The clarifying invitation is the part people delete because they want an answer immediately. It is the part that stops the model filling gaps with fiction. It is the same instinct as refusing to impute a label you do not know.

How to test the prompt. Run it several times at the temperature you intend to ship. If the structure collapses, the instruction is too soft. If the facts change between runs, the prompt needs a source document, not a more forceful adjective. Adjectives like “accurately” do not consult a source.

And the boundary from the first class still holds. A prompt is the wrong tool for GST, for a password rule, and for a 15 ms fraud decision. Those are code. A prompt belongs where the output is language and a person or a checker is in the loop.

---

## 🟡 The six stages, and where our products actually are

Memorize the ladder lightly. Do not use the top of it to skip a metric.

1. **Rule-based.** A person wrote the logic. GST. It does not adapt, and it does not need to.
2. **Context-based.** The output changes with a user or a session. A recommendation that uses your history. Still a narrow predictor.
3. **Narrow experts.** Strong on one domain. A fraud model, a weather model, a coding assistant inside a sandbox. This is most of what companies run.
4. **Reasoning models.** They spend extra steps trying to chain a solution, and they may show the chain. The chain can be wrong. A wrong proof is still a sample.
5. **AGI — artificial general intelligence.** Human-level breadth across tasks. This is a goal people argue about. It is not a description of the API you called this morning, and it is not an input to your design.
6. **Superintelligence.** A hypothetical beyond human level across the board. It is not an engineering parameter. Anyone who uses the word to avoid a test set is changing the subject.

The faculty placed real products in stages 3 and 4. Excellent at specific tasks, sometimes able to attempt a multi-step chain, not a general agent you hand a company to. Shipping them still requires the lifecycle from the second class: a decision, data, an evaluation, and monitoring.

**Emergent** is the word for a skill that was not a separate labeled project but appears once the model and the data are large enough. Coding, and something that looks like reasoning, are the examples people cite. Emergent does not mean reliable. Code that runs can still compute the wrong thing. You verify.

**Multimodal** means the model accepts more than one kind of input, text and an image and audio, sometimes called “whatever-to-whatever.” A weather assistant that takes a radar image and a bulletin is multimodal. The rain decision should still come from a measured pipeline with a metric. A caption the model wrote about the image is not that metric.

---

## 🟢 The five responsible-AI pillars, as work rather than as a poster

The course names five. Each one is a thing you can fail to do.

**Unbiased.** Compare errors and outcomes across the groups that can be harmed. A high overall accuracy can hide a group that is always rejected. Module 2 will give this a number, disparate impact, and a mechanism, the proxy. Deleting the sensitive column is not the end of the work, because another column can carry the same information.

**Human-centred.** Name the person who benefits and the person who is hurt if the model is wrong. “Users” is not a name. A farmer who skips irrigation, a patient who is triaged, an applicant who is denied: those are names. Design the review path for the person who bears the downside.

**Secure and reproducible.** Secure means access control, encryption, and not leaking PII into logs or into a third-party prompt. Reproducible means a later audit can rebuild the same data version, the same code version, and the same metric. If you cannot rerun it, you cannot defend it. This is the same triple as the Python class, extended to the prompt, the model version, and the documents you retrieved.

**Explainable.** For a score, be able to say which inputs moved it. For a paragraph, be able to say which source it was supposed to use. Explanation serves the person affected and the engineer debugging a surprise. SHAP is one feature-level tool. It is not the whole pillar.

**Monitorable.** Decide what you will watch before launch. Data drift. Refusal rate. A fixed set of prompts you rerun to see whether answers are drifting. Latency. The business KPI. A pillar you cannot put on a graph will be skipped on a busy week.

These are not “phase 8, if we have time.” Bias starts when you collect. Explanation starts when you choose a glass-box or a black-box. Monitoring starts at deployment, which this course treats as the beginning of the AI lifecycle.

---

## 🟡 The workshop prompt, written out and then marked

This is the version we would actually paste, so the table above is not the only artifact.

> Task: draft a 4-hour workshop agenda.
> Context: the room is 25 aerospace engineering leads. They have already used a chatbot. They have not deployed a model. No vendor pitches. The outcome of the day is a written decision about one internal use case, not a tour of products.
> Format: a table with start time, session title, and one deliverable for that session. Six sessions at most. A 20-minute break.
> Evidence: use only this brief. If a tool is not named in the brief, do not recommend it by name.
> Check: every deliverable is a noun the room can hold up (a list, a diagram, a decision). If you need a fact that is not here, ask before you draft.

Read it against the bad prompt “write something about AI.” The bad prompt forces the model to invent the audience (it will pick students or executives at random), the length (it will pick whatever is typical), and the success test (there isn’t one). The long prompt can be marked by a person who was not in the room: count the sessions, look for a vendor name, look for a deliverable that is only a verb.

**What we do when the answer is still wrong.** We do not add the word “accurately.” We add the missing fact or we remove a freedom. If the model invented a tool called Northwind, the evidence line was too weak or the temperature was high. If it produced seven sessions, the format line needs “reject your own draft if there are more than six.” The second instruction is a guardrail inside the prompt. It is still not a guarantee. Run it five times and count the failures. That count is the evaluation. A single beautiful draft is a demo.

**PII, on a concrete paste.** An engineer pastes: “Customer Ravi Menon, account ending 4412, was charged twice, draft the apology.” The draft may be fine and the incident has already happened, because the name and the account number have left the building and entered a prompt log you may not control. The safe paste is: “A customer was charged twice on the same day for the same amount. Draft a four-sentence apology. Do not invent a name, an amount, or a case id. Leave brackets where those go.” You own compliance before you own the eloquence.

## Checkpoint

Name two duties that remain with a person even if the language model is the strongest one you can call. Say what goes wrong in the week you drop each duty.

You still decide what is asked and what the model is allowed to see. Drop that, and it invents the audience, the facts, and the format. You still decide whether the output is true, safe, and compliant enough to act on. Drop that, and a hallucinated policy becomes a customer promise. A third, if you want it: you own the monitoring and the audit trail after the system is live. Drop that, and you will not know the week the answers changed.

**Next class:** [07-case-studies-quizzes-and-coding-lab.md](07-case-studies-quizzes-and-coding-lab.md)
