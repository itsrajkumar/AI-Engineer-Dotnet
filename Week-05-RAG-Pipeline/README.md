# Week 5: Building the RAG Pipeline (Retrieval-Augmented Generation)

> **Goal:** Connect the database, Semantic Kernel, and the LLM to eliminate hallucinations.

---

## 📅 Daily Breakdown

| Day | Topic | Type | Time |
|-----|-------|------|------|
| 1 | [RAG Architecture Review](./Day-01-RAG-Architecture/README.md) | 📖 Theory | ~3 hrs |
| 2 | [The Retrieval Step](./Day-02-Retrieval-Step/README.md) | 💻 Code | ~3 hrs |
| 3 | [The Augmentation Step](./Day-03-Augmentation-Step/README.md) | 💻 Code | ~3 hrs |
| 4 | [End-to-End RAG Implementation](./Day-04-End-to-End-RAG/README.md) | 💻 Code | ~3 hrs |
| 5 | [Handling Edge Cases](./Day-05-Edge-Cases/README.md) | 💻 Code | ~3 hrs |

---

## 🏗️ What is RAG?

RAG eliminates hallucinations by giving the LLM **real data** before it generates an answer:

```
WITHOUT RAG:                      WITH RAG:
┌───────────┐                     ┌───────────┐
│ User asks │                     │ User asks │
│ question  │                     │ question  │
└─────┬─────┘                     └─────┬─────┘
      │                                 │
      ▼                                 ▼
┌───────────┐                     ┌────────────────┐
│ LLM       │                     │ Embed question │
│ guesses   │ ◄── Hallucination!  └──────┬─────────┘
│ answer    │                            │
└───────────┘                            ▼
                                  ┌────────────────┐
                                  │ Search vector  │
                                  │ database       │
                                  └──────┬─────────┘
                                         │
                                         ▼
                                  ┌────────────────┐
                                  │ Inject results │
                                  │ into prompt    │
                                  └──────┬─────────┘
                                         │
                                         ▼
                                  ┌────────────────┐
                                  │ LLM answers    │
                                  │ from REAL data │ ◄── Accurate!
                                  └────────────────┘
```

---

## ➡️ Start Here

Begin with **[Day 1: RAG Architecture Review](./Day-01-RAG-Architecture/README.md)**
