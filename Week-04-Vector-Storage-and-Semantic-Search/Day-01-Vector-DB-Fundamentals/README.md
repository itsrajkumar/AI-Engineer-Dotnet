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

## 📊 Vector DB Options for .NET

| Database | Type | .NET SDK | Free Tier | Best For |
|----------|------|----------|-----------|----------|
| **MongoDB Atlas** | Document + Vector | ✅ MongoDB.Driver | ✅ 512MB | .NET devs who know Mongo |
| **PostgreSQL (pgvector)** | Relational + Vector | ✅ Npgsql + Pgvector | ✅ Local Docker | SQL devs, hybrid queries |
| **Qdrant** | Dedicated Vector | ✅ Qdrant.Client | ✅ 1GB cloud | Best pure vector performance |
| **Azure AI Search** | Enterprise Search | ✅ Azure.Search.Documents | Trial | Enterprise, Azure ecosystem |
| **Pinecone** | Dedicated Vector | ✅ REST Client | ✅ Limited | Serverless vector search |
| **ChromaDB** | Embedded Vector | ⚠️ HTTP client | ✅ Local | Quick prototyping |
| **SQL Server** | Relational | ✅ SqlClient | ✅ Express | When staying in SQL Server |

### Recommendation for .NET Developers

```
Already use MongoDB?     → MongoDB Atlas Vector Search (Day 2)
Already use PostgreSQL?  → pgvector extension (Day 3)
Need best performance?   → Qdrant
Need Azure enterprise?   → Azure AI Search
Just learning?           → Start with pgvector in Docker
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
