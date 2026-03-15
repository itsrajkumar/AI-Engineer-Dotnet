# Week 1: AI Fundamentals & The .NET API Layer

> **Goal:** Understand LLM mechanics and establish a programmatic connection using modern C#.

---

## 🎯 Learning Objectives

By the end of this week, you will:
- Understand the theory behind Generative AI, LLMs, and SLMs
- Master prompt engineering patterns (Zero-shot, Few-shot, Chain-of-Thought)
- Use `Microsoft.Extensions.AI` as a unified abstraction layer
- Connect to an LLM via C# and send/receive messages programmatically
- Implement chat history with System, User, and Assistant roles

---

## 📅 Daily Breakdown

| Day | Topic | Type | Time |
|-----|-------|------|------|
| 1 | [AI Theory & Terminology](./Day-01-AI-Theory-and-Terminology/README.md) | 📖 Theory | ~3 hrs |
| 2 | [Prompt Engineering Basics](./Day-02-Prompt-Engineering-Basics/README.md) | 📖 Theory + Practice | ~3 hrs |
| 3 | [Microsoft.Extensions.AI](./Day-03-Microsoft-Extensions-AI/README.md) | 💻 Code | ~3 hrs |
| 4 | [Your First API Connection](./Day-04-First-API-Connection/README.md) | 💻 Code | ~3 hrs |
| 5 | [System Prompts & Roles](./Day-05-System-Prompts-and-Roles/README.md) | 💻 Code | ~3 hrs |

---

## 🏗️ Architecture Context

```
Week 1 Focus Area:
┌──────────────────────────────────────────┐
│           YOUR .NET APPLICATION          │
│                                          │
│  ┌─────────────────────────────────────┐ │
│  │   Microsoft.Extensions.AI           │ │  ◄── Day 3
│  │   (Unified Abstraction Layer)       │ │
│  └────────────┬────────────────────────┘ │
│               │                          │
│  ┌────────────▼────────────────────────┐ │
│  │   Azure.AI.OpenAI SDK              │ │  ◄── Day 4
│  │   (Provider-specific SDK)          │ │
│  └────────────┬────────────────────────┘ │
│               │                          │
│  ┌────────────▼────────────────────────┐ │
│  │   Chat History & Roles             │ │  ◄── Day 5
│  │   (System / User / Assistant)      │ │
│  └─────────────────────────────────────┘ │
└─────────────────┬────────────────────────┘
                  │ HTTP/REST
                  ▼
         ┌────────────────┐
         │   LLM Service  │
         │  (GPT-4o-mini) │
         └────────────────┘
```

---

## 📚 Key Concepts This Week

| Concept | Analogy for .NET Devs |
|---------|----------------------|
| LLM | A pre-trained "service" that generates text — like a very smart AutoMapper |
| Token | Not a JWT — it's a sub-word unit (~4 chars in English) |
| Context Window | Like a request body size limit on your API |
| Temperature | Like a randomness seed — 0 = deterministic, 1 = creative |
| Prompt | The "request" you send to the LLM "API" |
| System Prompt | Like middleware that sets behavior before every request |
| Embedding | A `float[]` that represents the "meaning" of text mathematically |

---

## ➡️ Start Here

Begin with **[Day 1: AI Theory & Terminology](./Day-01-AI-Theory-and-Terminology/README.md)**
