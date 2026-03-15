# Week 3: Embeddings & Data Processing

> **Goal:** Learn how to represent text as mathematics so the AI can search by "meaning."

---

## 🎯 Learning Objectives

By the end of this week, you will:
- Understand how text is converted to high-dimensional vectors
- Generate embeddings using .NET
- Implement document chunking strategies
- Calculate cosine similarity between vectors
- Build a batch processing pipeline

---

## 📅 Daily Breakdown

| Day | Topic | Type | Time |
|-----|-------|------|------|
| 1 | [Embedding Theory](./Day-01-Embedding-Theory/README.md) | 📖 Theory | ~3 hrs |
| 2 | [Generating Embeddings in .NET](./Day-02-Generating-Embeddings-DotNet/README.md) | 💻 Code | ~3 hrs |
| 3 | [Document Chunking Strategies](./Day-03-Document-Chunking/README.md) | 💻 Code | ~3 hrs |
| 4 | [Cosine Similarity](./Day-04-Cosine-Similarity/README.md) | 💻 Code | ~3 hrs |
| 5 | [Batch Processing Pipeline](./Day-05-Batch-Processing-Pipeline/README.md) | 💻 Code | ~3 hrs |

---

## 🏗️ Why Embeddings Matter

```
Traditional Search:          Semantic Search:
"car" matches "car"          "car" matches "automobile", "vehicle", "sedan"
"dog" ≠ "puppy"             "dog" ≈ "puppy" ≈ "canine"

Keyword matching             MEANING matching
(exact string)              (mathematical similarity)
```

Embeddings are the **foundation** of RAG (Week 5). Without embeddings, there is no semantic search.

---

## ➡️ Start Here

Begin with **[Day 1: Embedding Theory](./Day-01-Embedding-Theory/README.md)**
