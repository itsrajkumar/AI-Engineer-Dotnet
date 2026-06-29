# Day 1: Vector Database Fundamentals

> **Type:** 📖 Theory | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Understand how vector databases differ from traditional databases
- Learn ANN (Approximate Nearest Neighbor) algorithms
- Know when to use dedicated vs. extension-based vector storage
- Compare vector database options available for .NET

---

## 📖 What Is a Vector Database?

A vector database is optimized for **storing and searching high-dimensional vectors** (like embeddings). Instead of SQL-style `WHERE` queries, you ask: *"Find me the 5 vectors most similar to this one."*

### Traditional DB vs. Vector DB

```
Traditional Database:                    Vector Database:
SELECT * FROM Products                   Find 5 nearest neighbors
WHERE Category = 'Electronics'           to vector [0.12, -0.34, ...]
AND Price < 500                          in collection "products"

Exact match                              Approximate match
O(log n) with index                      O(log n) with ANN index
Returns rows                             Returns vectors + scores
```

---

## 🔍 ANN (Approximate Nearest Neighbor) Algorithms

Finding the exact nearest vector in a million-dimensional space is computationally expensive. ANN algorithms trade a tiny bit of accuracy for massive speed gains.

### HNSW (Hierarchical Navigable Small World)
- **Most popular** algorithm for vector search
- Builds a multi-layered graph of vectors
- Navigates from top (coarse) to bottom (fine) layer
- **Used by:** PostgreSQL pgvector, MongoDB Atlas, Qdrant

### IVF (Inverted File Index)
- Clusters vectors into partitions (like sharding)
- Searches only the most relevant clusters
- **Used by:** Milvus, FAISS

### Product Quantization (PQ)
- Compresses vectors to use less memory
- Trades accuracy for reduced storage
- Often combined with IVF

---

## 📊 Vector DB Options for .NET (2026 Landscape)

| Database | Type | .NET SDK | Notable 2026 Features |
|----------|------|----------|-----------------------|
| **MongoDB Atlas v8.3** | Document + Vector | ✅ MongoDB.Driver | Automated embeddings (Voyage-4), flat indexes |
| **PostgreSQL (pgvector 0.9.x)** | Relational + Vector | ✅ Npgsql + Pgvector | Iterative scans, halfvec, Matryoshka support |
| **Qdrant** | Dedicated Vector | ✅ Qdrant.Client | TurboQuant (extreme compression) |
| **Pinecone** | Dedicated Vector | ✅ Pinecone.NET | Nexus architecture, KnowQL query language |
| **Weaviate** | Dedicated Vector | ✅ Weaviate.Client | v1.38 Engram (agentic memory optimized) |
| **Milvus 3.0** | Dedicated Vector | ✅ Milvus.Client | Beta release, GPU acceleration, hybrid search |
| **LanceDB** | Serverless / Embedded | ✅ LanceDB.NET | Disk-based vector search (massively scalable) |
| **Turbopuffer** | Serverless API | ⚠️ HTTP client | True serverless, built on object storage |
| **SQL Server** | Relational | ✅ SqlClient | Native vector support in Azure SQL / SQL Server 2025 |

### Recommendation for .NET Developers

```
Already use MongoDB?     → MongoDB Atlas Vector Search v8.3
Already use PostgreSQL?  → pgvector 0.9.x
Need best performance?   → Qdrant or Pinecone Nexus
Need embedded/local scale?→ LanceDB
Serverless API preference?→ Turbopuffer
```

---

## 🔑 Key Concepts

### Indexing Strategies
```
No Index (Brute Force):
  Every query compares against ALL vectors → Accurate but slow
  OK for < 10,000 vectors

HNSW Index:
  Builds graph for fast traversal → Fast, slightly approximate
  Best for 10K–10M vectors

IVF Index:
  Partitions then searches → Fastest for very large datasets
  Best for > 1M vectors
```

### Metadata Filtering
Vector databases store both the vector AND metadata alongside it:
```json
{
  "id": "doc-123-chunk-5",
  "vector": [0.12, -0.34, 0.56, ...],
  "metadata": {
    "source": "employee-handbook.pdf",
    "page": 42,
    "department": "Engineering",
    "last_updated": "2024-01-15"
  }
}
```

This allows **filtered vector search:** "Find similar vectors, but only from the Engineering department."

---

## ➡️ Next

Continue to **[Day 2: Document DB & Vector Search](../Day-02-Document-DB-Vector-Search/README.md)**
