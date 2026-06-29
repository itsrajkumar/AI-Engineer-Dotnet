# Day 3: The Transformer Architecture

> **Type:** 📖 Theory | **Time:** ~3 hours
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Deconstruct the famous "Attention Is All You Need" architecture.
- Understand the difference between Encoders and Decoders.
- Deep dive into Self-Attention (Queries, Keys, Values).
- Understand Positional Encodings (RoPE, ALiBi).
- Learn why LayerNorm and Residual Connections are critical.

---

## 📖 The Architecture that Changed Everything

Before 2017, sequence models (like RNNs and LSTMs) processed text one word at a time, making them slow to train and bad at remembering long-range context. 

The **Transformer** changed this by processing the entire sequence at once using the **Attention Mechanism**.

### 1. Encoder vs Decoder

- **Encoder-Only (e.g., BERT):** Looks at the whole sequence simultaneously to build a deep understanding. Great for classification or semantic search (Embeddings!).
- **Decoder-Only (e.g., GPT, Llama, Claude):** Predicts the next word using only the *past* words (causal masking). Great for generation.
- **Encoder-Decoder (e.g., T5):** Takes an input sequence, encodes it, and generates a new sequence. Great for translation.

### 2. The Core Components of a GPT (Decoder) Block

A modern LLM is just a stack of identical "Transformer Blocks" repeated 12, 24, or 96 times. Each block contains:

1. **Layer Normalization (LayerNorm):** Stabilizes training by ensuring values don't explode.
2. **Self-Attention:** The heart of the model. Figures out which words are related to each other.
3. **Residual Connection:** Adds the input of the layer directly to the output (`x = x + attention(x)`). Prevents the gradient from vanishing in deep networks.
4. **Feed Forward Network (MLP):** A standard neural network (from Day 2) applied to each token individually to process the information gathered by the attention mechanism.

### 3. Self-Attention Deep Dive (Q, K, V)

Attention is a database retrieval concept applied to matrices.

- **Query (Q):** What I am looking for.
- **Key (K):** What I contain.
- **Value (V):** What I actually mean.

**The Math:** $Attention(Q, K, V) = softmax(\frac{QK^T}{\sqrt{d_k}})V$

1. Multiply Queries and Keys ($QK^T$). This creates an "attention matrix" showing how much every word relates to every other word.
2. Scale it by dividing by $\sqrt{d_k}$.
3. Apply Softmax so the scores add up to 1.
4. Multiply by the Values ($V$) to get the final representation.

### 4. Positional Encoding

Because the Transformer processes everything at once, it has no idea about the order of the words. "Dog bites man" and "Man bites dog" look identical.

We must inject positional information into the embeddings.
- **2017 standard:** Sinusoidal encodings (using sine and cosine waves).
- **2026 standard:** **RoPE (Rotary Position Embeddings)** used in Llama 4, or **ALiBi** which allows models to extrapolate to context windows larger than they were trained on (e.g. 1 Million+ tokens).

---

## 🔑 Key Takeaways

- The Transformer processes text in parallel, allowing massive scale.
- **Self-Attention** determines how words relate to each other in context.
- Modern GenAI models (GPT-5, Llama 4) are **Decoder-only** transformers.
- **RoPE** is the modern standard for Positional Encoding.

---

## ➡️ Next

Continue to **[Day 4: Build a Mini-Transformer](../Day-04-Mini-Transformer/README.md)**
