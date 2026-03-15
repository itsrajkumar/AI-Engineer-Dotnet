# Week 4: Vector Storage & Semantic Search

> **Goal:** Store your embeddings using familiar database technologies and perform semantic search.

---

## 📅 Daily Breakdown

| Day | Topic | Type | Time |
|-----|-------|------|------|
| 1 | [Vector Database Fundamentals](./Day-01-Vector-DB-Fundamentals/README.md) | 📖 Theory | ~3 hrs |
| 2 | [Document DB & Vector Search](./Day-02-Document-DB-Vector-Search/README.md) | 💻 Code | ~3 hrs |
| 3 | [Relational DB & Vectors](./Day-03-Relational-DB-Vectors/README.md) | 💻 Code | ~3 hrs |
| 4 | [Hybrid Search Integration](./Day-04-Hybrid-Search/README.md) | 💻 Code | ~3 hrs |
| 5 | [Repository Pattern for Vectors](./Day-05-Repository-Pattern-Vectors/README.md) | 💻 Code | ~3 hrs |

---

## 🏗️ Architecture Context

```
Week 4 Focus: Where to PUT the embeddings from Week 3

┌──────────────────────────────────────────────────────┐
│                  STORAGE OPTIONS                      │
│                                                       │
│  ┌─────────────────┐  ┌──────────────────────────┐   │
│  │ Dedicated        │  │ Existing DB + Extension  │   │
│  │ Vector DBs       │  │                          │   │
│  │                  │  │ ┌─────────────────────┐  │   │
│  │ • Pinecone       │  │ │ MongoDB Atlas       │  │   │  ◄── Day 2
│  │ • Milvus         │  │ │ ($vectorSearch)     │  │   │
│  │ • Qdrant         │  │ └─────────────────────┘  │   │
│  │ • Weaviate       │  │ ┌─────────────────────┐  │   │
│  │                  │  │ │ PostgreSQL          │  │   │  ◄── Day 3
│  │ (New tech)       │  │ │ (pgvector)          │  │   │
│  │                  │  │ └─────────────────────┘  │   │
│  └─────────────────┘  │ ┌─────────────────────┐  │   │
│                        │ │ SQL Server          │  │   │
│                        │ │ (emerging support)  │  │   │
│                        │ └─────────────────────┘  │   │
│                        └──────────────────────────┘   │
└──────────────────────────────────────────────────────┘
```

---

## ➡️ Start Here

Begin with **[Day 1: Vector Database Fundamentals](./Day-01-Vector-DB-Fundamentals/README.md)**
