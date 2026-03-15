# Day 1: RAG Architecture Review

> **Type:** 📖 Theory | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Understand the complete RAG pipeline end-to-end
- Map each step to specific .NET components
- Learn the ingestion pipeline vs. query pipeline
- Understand quality metrics for RAG systems

---

## 📖 The Two RAG Pipelines

### Pipeline 1: Document Ingestion (Offline)

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  1. Load     │    │  2. Chunk    │    │  3. Embed    │    │  4. Store    │
│  Documents   │───►│  Documents   │───►│  Chunks      │───►│  in Vector   │
│  (PDF, MD,   │    │  (Split into │    │  (Generate   │    │  Database    │
│   TXT, DOCX) │    │   300-500    │    │   float[]    │    │  (pgvector,  │
│              │    │   token      │    │   for each   │    │   MongoDB)   │
│              │    │   chunks)    │    │   chunk)     │    │              │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
    Week 3               Week 3             Week 3              Week 4
```

### Pipeline 2: Query (Real-time)

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  1. User     │    │  2. Embed    │    │  3. Search   │    │  4. Augment  │
│  Question    │───►│  Question    │───►│  Vector DB   │───►│  Prompt +    │
│              │    │  (Same model │    │  (Find top K │    │  Context     │
│              │    │   as ingest) │    │   similar)   │    │              │
└──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                                   │
                    ┌──────────────┐    ┌──────────────┐          │
                    │  6. Return   │    │  5. LLM      │◄─────────┘
                    │  Answer to   │◄───│  Generates   │
                    │  User        │    │  Answer from  │
                    │              │    │  Context ONLY │
                    └──────────────┘    └──────────────┘
```

---

## 🗺️ Mapping to .NET Components

| RAG Step | .NET Component | Week |
|----------|---------------|------|
| Load documents | `File.ReadAllTextAsync()`, PDF libraries | Week 3 |
| Chunk documents | `TextChunker` (custom class) | Week 3 |
| Embed text | `IEmbeddingGenerator<string, Embedding<float>>` | Week 3 |
| Store vectors | `IVectorRepository` (MongoDB/pgvector) | Week 4 |
| Search vectors | `IVectorRepository.SearchAsync()` | Week 4 |
| Build prompt | Semantic Kernel prompt templates | Week 2 |
| Generate answer | `IChatClient` / `IChatCompletionService` | Week 1 |

---

## 📊 RAG Quality Metrics

| Metric | What it Measures | Good Score |
|--------|-----------------|-----------|
| **Faithfulness** | Does the answer use only the provided context? | > 0.9 |
| **Relevancy** | Is the retrieved context relevant to the question? | > 0.8 |
| **Answer Correctness** | Is the final answer factually correct? | > 0.85 |
| **Context Recall** | Does the retrieval find ALL relevant chunks? | > 0.7 |

---

## 🔑 Common RAG Failures

| Problem | Cause | Solution |
|---------|-------|----------|
| Hallucination | LLM ignores context | Stronger system prompt with guardrails |
| Wrong answers | Retrieved chunks are irrelevant | Better chunking strategy, hybrid search |
| Missing answers | Relevant data not in DB | More thorough document ingestion |
| Costly | Too many tokens | Smaller chunks, filter irrelevant results |

---

## ➡️ Next

Continue to **[Day 2: The Retrieval Step](../Day-02-Retrieval-Step/README.md)**
