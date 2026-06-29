# Day 5: Modern Architectures (2026 Standards)

> **Type:** 📖 Theory | **Time:** ~2 hours
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand the limitations of the standard Transformer.
- Learn about Mixture of Experts (MoE) routing.
- Explore State Space Models (Mamba).
- Understand Multimodal architectures (Vision Transformers).

---

## 📖 Beyond the Basic Transformer

The standard GPT architecture (from Day 4) scaled incredibly well from 2018 to 2023, but eventually hit a wall: **Compute Efficiency**. If you have a 1 Trillion parameter model, passing a single token through every single parameter is too slow and expensive.

### 1. Mixture of Experts (MoE)

In 2026, almost all frontier models (Llama 4, DeepSeek V4, Mistral Large, GPT-5) are **Mixture of Experts (MoE)** models.

- **The Concept:** Instead of one massive Feed Forward Network (MLP), you have 8 or 16 smaller MLPs ("Experts").
- **The Router:** A small routing network looks at each token and decides which 2 experts are best suited to process it (e.g., "This looks like code, send it to Expert 3 and 7").
- **The Benefit:** A 100B parameter model might only activate 12B parameters per token. You get the knowledge capacity of a massive model with the speed and cost of a small model.

### 2. State Space Models (Mamba / Mamba-2)

The biggest flaw in the Transformer is the Attention mechanism's memory scaling. Attention requires comparing every token to every previous token ($O(N^2)$ complexity). A 1 Million token context window requires immense VRAM.

**Mamba (SSM)** architectures solve this:
- They don't store the whole history. They compress history into a hidden "State Space".
- They scale linearly ($O(N)$), allowing effectively infinite context windows on consumer hardware.
- In 2026, many open-source models use a hybrid approach: Attention for local, precise recall, and Mamba for long-range context.

### 3. Vision Transformers (ViT)

How do models see?
- A Vision Transformer takes an image (e.g., 224x224 pixels) and chops it up into a grid of 16x16 "patches".
- Each patch is flattened into a vector and treated exactly like a "word token" in a text sequence.
- The model then uses standard Self-Attention to figure out how the top-left patch relates to the bottom-right patch.

Multimodal models (like GPT-5 and Llama 4) project both text tokens and image patches into the exact same embedding space, allowing the model to "reason" across text and images natively without an external OCR step.

---

## 🏆 Week 11 Complete!

You now understand the deep mathematical foundations of modern AI!

Continue to **[Week 12: Model Training & Alignment](../../Week-12-Model-Training-and-Alignment/README.md)** to learn how we teach these raw neural networks to actually follow instructions and behave safely.
