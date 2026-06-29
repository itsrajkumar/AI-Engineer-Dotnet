# Day 2: Supervised Fine-Tuning (SFT)

> **Type:** 📖 Theory | **Time:** ~2 hours
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand the goal of Supervised Fine-Tuning (SFT).
- Learn how to structure conversational datasets (ChatML format).
- Understand Chat Templates and why they matter.
- Evaluate the cost and data requirements for effective SFT.

---

## 📖 Turning a Predictor into an Assistant

A base model is a text-continuation engine.
If you input: *"How do I bake a cake?"*
A base model might output: *"How do I bake a pie? How do I bake cookies?"* (It thinks you are writing a list of questions).

**Supervised Fine-Tuning (SFT)** uses the exact same training loop we learned in Week 11 (Forward -> Loss -> Backward -> Step), but with very specific data.

We train the model on examples of **how we want it to behave**.

### 1. The Chat Format

To teach the model that a conversation is happening between a User and an Assistant, we inject special tokens into the text. This is called a **Chat Template**. 

A common format (ChatML) looks like this to the model:
```
<|im_start|>system
You are a helpful baking assistant.<|im_end|>
<|im_start|>user
How do I bake a cake?<|im_end|>
<|im_start|>assistant
First, preheat your oven...<|im_end|>
```

During SFT, we calculate the Loss *only* on the Assistant's response. The model learns that when it sees `<|im_start|>assistant`, it must generate a helpful answer and stop when it generates `<|im_end|>`.

### 2. Data Requirements for SFT

- **Quality > Quantity:** 1,000 highly accurate, beautifully formatted, grammatically perfect examples will result in a much better model than 100,000 scraped, messy Reddit conversations.
- **LIMA (Less Is More for Alignment):** Research showed that you can get an incredibly capable assistant with just 1,000 extremely high-quality, diverse prompts and responses.
- **Diversity:** If all your training data is short "Yes/No" answers, the model will forget how to write long essays (a phenomenon called "catastrophic forgetting"). The dataset must represent the full distribution of tasks you want the model to perform.

### 3. When should you use SFT?

You should fine-tune a model when:
1. You need the model to output a very specific, rigid JSON format 100% of the time, and prompt engineering is failing.
2. You want to teach the model a new "tone of voice" (e.g., talk exactly like a 19th-century pirate, or a hyper-professional medical assistant).
3. You want to distill the knowledge of a massive, expensive model (like GPT-5) into a tiny, cheap model (like Llama 4 Scout) to run on the edge.

You should **NOT** fine-tune a model to teach it new facts. (e.g., do not fine-tune a model on your company's HR handbook). Use **RAG** for knowledge injection. Use **Fine-tuning** for behavior/format injection.

---

## 🔑 Key Takeaways

- **SFT** teaches a base model the conversational Q&A format.
- Data quality is the single most important factor in SFT.
- Fine-tuning is for **form/behavior**, RAG is for **facts/knowledge**.

---

## ➡️ Next

Continue to **[Day 3: RLHF and DPO](../Day-03-RLHF-and-DPO/README.md)**
