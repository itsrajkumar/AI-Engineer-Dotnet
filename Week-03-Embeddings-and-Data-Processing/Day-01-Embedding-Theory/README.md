# Day 1: Embedding Theory

> **Type:** 📖 Theory | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Understand what embeddings are and why they exist
- Visualize high-dimensional vector spaces
- Know the difference between sparse and dense vectors
- Understand embedding models vs. LLMs

---

## 📖 What Are Embeddings?

An embedding is a **fixed-size array of floating-point numbers** (`float[]`) that captures the **semantic meaning** of a piece of text.

```
"The cat sat on the mat"  →  [0.0123, -0.0456, 0.0789, ..., 0.0321]
                               ↑ This is a 1536-dimension float array!

"A feline rested on a rug" → [0.0121, -0.0452, 0.0785, ..., 0.0319]
                               ↑ Very SIMILAR numbers because SIMILAR meaning!

"Stock market crashed"     → [-0.0891, 0.0234, -0.0567, ..., 0.0891]
                               ↑ DIFFERENT numbers because DIFFERENT meaning!
```

### .NET Analogy
```csharp
// Think of an embedding like a hash code, but for MEANING
// Unlike GetHashCode() which gives one number, 
// embedding gives you 1536+ numbers that capture semantic relationships

string text = "The cat sat on the mat";
int hashCode = text.GetHashCode();          // One number, no semantic meaning
float[] embedding = await Embed(text);       // 1536 numbers, captures full meaning
```

---

## 📐 Dimensionality

| Model | Dimensions | Array Size |
|-------|-----------|-----------|
| text-embedding-3-small | 1536 | `float[1536]` |
| text-embedding-3-large | 3072 | `float[3072]` |
| text-embedding-ada-002 | 1536 | `float[1536]` |
| nomic-embed-text (Ollama) | 768 | `float[768]` |

Why 1536 dimensions? Each dimension represents a different "aspect" of meaning. More dimensions = more nuance, but higher cost.

---

## 🎯 Key Properties of Embeddings

### 1. Similar Meaning = Similar Vectors
```
"happy"     → [0.2, 0.8, 0.3, ...]
"joyful"    → [0.21, 0.79, 0.31, ...]    ← Very close!
"sad"       → [-0.3, -0.7, 0.1, ...]     ← Very different!
```

### 2. Analogies Work Mathematically
```
King - Man + Woman ≈ Queen

vector("king") - vector("man") + vector("woman") ≈ vector("queen")
```

### 3. Embeddings Are NOT Reversible
```
text → embedding    ✅ Easy (API call)
embedding → text    ❌ Impossible (it's lossy compression)
```

---

## 📊 Embedding Models vs. LLM Models

| Feature | Embedding Model | LLM (Chat Model) |
|---------|----------------|-------------------|
| **Input** | Text | Text (messages) |
| **Output** | `float[]` (vector) | Generated text |
| **Purpose** | Represent meaning as numbers | Generate new text |
| **Cost** | Very cheap ($0.02/1M tokens) | More expensive ($0.15+/1M) |
| **Speed** | Very fast | Slower |
| **Use in RAG** | Convert text to searchable vectors | Generate final answer |

```
RAG Pipeline:
                    Embedding Model          LLM
User Question ──────────────────► float[] ──► Search DB ──► Context + Question ──► Answer
                    (cheap, fast)                           (reasoning, expensive)
```

---

## 🔑 Key Concepts for .NET Developers

### Vector Space
A "space" where each text occupies a position based on its meaning. Similar texts are neighbors.

### Distance Metrics
Used to measure "closeness" of two vectors:
- **Cosine Similarity** (most common): Measures the angle between vectors (0 = unrelated, 1 = identical meaning)
- **Euclidean Distance**: Straight-line distance between points
- **Dot Product**: Similar to cosine but affected by vector magnitude

---

## 📝 Self-Assessment

1. What data type does an embedding model return?
2. Why are embeddings useful for search?
3. Can you convert an embedding back to text?
4. What's the approximate cost of embedding 1 million tokens?
5. In what step of RAG are embeddings used?

<details>
<summary>Answers</summary>

1. `float[]` — an array of floating-point numbers
2. Because similar meanings produce similar vectors, enabling "search by meaning" not just keywords
3. No — embeddings are lossy; you can't reverse them
4. ~$0.02 with text-embedding-3-small
5. Two places: (a) embedding the user's question, and (b) embedding documents during ingestion

</details>

---

## ➡️ Next

Continue to **[Day 2: Generating Embeddings in .NET](../Day-02-Generating-Embeddings-DotNet/README.md)**
