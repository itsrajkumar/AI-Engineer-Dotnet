# Day 1: RAG Architecture Review

> **Type:** 📖 Theory | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Understand the complete RAG pipeline end-to-end
- Map each step to specific .NET components
- Learn the ingestion pipeline vs. query pipeline
- Understand quality metrics for RAG systems

---

## 📖 The Evolution of RAG (2023 vs 2026)

| Concept | 2023 (Standard RAG) | 2026 (Agentic/Advanced RAG) |
|---------|---------------------|----------------------------|
| **Orchestration** | Linear Pipeline (Retrieve → Generate) | Agentic RAG (Agents plan, search, verify) |
| **Retrieval** | Single Vector Search | Hybrid (Vector + Keyword + GraphRAG) |
| **Chunking** | Fixed Size | Contextual Chunking, Late Chunking |
| **Verification** | Prompt engineering | Self-RAG, Corrective RAG (CRAG) |
| **Documents** | Flat list | Hierarchical (RAPTOR) |

---

## 📖 The Two RAG Pipelines (Standard Model)

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

## 🚀 Advanced RAG Paradigms (2026 Standards)

### 1. Agentic RAG
The dominant paradigm in 2026. Instead of a hardcoded pipeline, an **Agent** acts as the orchestrator. The agent is given search tools and decides *when* to search, *how many times* to search, and *how* to synthesize the results. If the retrieved context is poor, the agent loops back and searches again.

### 2. GraphRAG
Created by Microsoft, GraphRAG extracts entities and their relationships from documents to build a Knowledge Graph. It allows LLMs to answer global questions like *"What are the main themes across all these documents?"* which traditional vector RAG fails at.

### 3. RAPTOR (Recursive Abstractive Processing for Tree-Organized Retrieval)
Documents are clustered and summarized hierarchically. When a user asks a high-level question, the system retrieves the high-level summaries. When asking detailed questions, it retrieves the leaf nodes.

### 4. CRAG (Corrective RAG) & Self-RAG
**CRAG:** An evaluator grades the retrieved documents. If they are irrelevant, the system performs a web search or rewrites the query.
**Self-RAG:** The LLM generates the answer and simultaneously outputs "reflection tokens" grading its own generation for hallucination and relevance, self-correcting in real-time.

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
