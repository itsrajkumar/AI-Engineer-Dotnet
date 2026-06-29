# Day 1: Pretraining & Datasets

> **Type:** 📖 Theory | **Time:** ~2 hours
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand the lifecycle of an LLM (Pretraining → SFT → Alignment).
- Explore modern pretraining datasets (FineWeb, RefinedWeb).
- Understand data deduplication and contamination.
- Learn how Tokenizers work (BPE, Tiktoken).

---

## 📖 The Three Stages of LLM Training

Training a modern AI model like Llama 4 or GPT-5 happens in three distinct phases:

1. **Pretraining (The Knowledge Engine):** 
   - **Goal:** Learn the structure of language, facts about the world, and basic reasoning.
   - **Data:** Massive scrapes of the entire internet (10-15 Trillion tokens).
   - **Cost:** Tens of millions of dollars. Thousands of GPUs for months.
   - **Output:** A "Base Model". It knows a lot, but it is not an assistant. It just predicts the next word.

2. **Supervised Fine-Tuning (SFT) (The Format):**
   - **Goal:** Teach the model how to act like an assistant (User asks a question, Assistant answers).
   - **Data:** 10,000 to 100,000 high-quality, human-written instruction-response pairs.
   - **Cost:** Thousands of dollars. A few GPUs for days.

3. **Alignment (RLHF/DPO) (The Personality & Safety):**
   - **Goal:** Teach the model what answers humans *prefer* (safe, polite, accurate, refusing dangerous requests).
   - **Data:** Human preference data (e.g., Prompt A, Response 1 is better than Response 2).
   - **Cost:** Very expensive human labeling, but cheap to train compared to pretraining.

### The Importance of Data Quality

In the pretraining phase, the old adage is true: **Garbage In, Garbage Out**. 
Modern models are trained on curated datasets like **FineWeb**. The process involves:
- **Filtering:** Removing toxic, low-quality, or machine-generated spam.
- **Deduplication:** If the model sees the same paragraph 1,000 times, it will memorize it and overfit. Data must be rigorously deduplicated.
- **De-contamination:** Ensuring benchmark tests (like the bar exam or coding tests) are absolutely completely removed from the training data, so the model isn't "cheating" on its exams.

### Tokenization (Byte-Pair Encoding)

Models do not read words; they read numbers (tokens). 

A tokenizer breaks text into chunks. 
- "apple" -> `[4398]`
- "unbelievable" -> `[un]`, `[believ]`, `[able]` -> `[342, 5982, 128]`

**BPE (Byte-Pair Encoding)** is the standard algorithm. It finds the most common pairs of characters and merges them into a single token. Modern tokenizers (like OpenAI's `o200k_base` used in GPT-4o/5) have vocabularies of 200,000 tokens, which allows them to represent multiple languages and even raw code very efficiently.

---

## 🔑 Key Takeaways

- You will likely never **Pretrain** a model yourself. That is left to companies with massive compute clusters (OpenAI, Meta, Microsoft, Mistral).
- Your job as an AI Engineer is to take those Base Models and use **SFT and Alignment** to customize them for your specific enterprise use cases.
- Tokenization matters: a highly efficient tokenizer means the model can read more text for a cheaper price.

---

## ➡️ Next

Continue to **[Day 2: Supervised Fine-Tuning](../Day-02-Supervised-Fine-Tuning/README.md)**
