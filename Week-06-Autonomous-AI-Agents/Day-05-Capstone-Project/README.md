# Day 5: Capstone Project Planning

> **Type:** 🏗️ Project | **Time:** ~4 hours

---

## 🎯 Learning Objectives

- Design a portfolio-worthy AI application
- Architecture an end-to-end system using everything from Weeks 1-6
- Create project scaffolding with Clean Architecture
- Plan the capstone project for the [Capstone folder](../../Capstone-Project/README.md)

---

## 🏗️ Capstone: AI-Powered Retail Assistant

### Project Description

Build a **.NET 8 Web API** that powers an AI retail assistant capable of:
1. **Answering policy questions** using RAG (e.g., "What's your return policy?")
2. **Looking up orders** using function calling
3. **Processing returns/refunds** with human-in-the-loop approval
4. **Multi-step reasoning** for complex customer requests

---

## 📐 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND (Optional)                       │
│                 (HTML/JS or Angular/Blazor)                  │
└─────────────────────┬───────────────────────────────────────┘
                      │ REST/SignalR
                      ▼
┌─────────────────────────────────────────────────────────────┐
│               RetailAssistant.API                           │
│                                                              │
│  Controllers/Endpoints:                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ POST /chat   │  │ GET /history │  │ POST /ingest     │  │
│  └──────┬───────┘  └──────────────┘  └──────────────────┘  │
│         │                                                    │
│  ┌──────▼───────────────────────────────────────────────┐   │
│  │              RetailAssistant.Core                     │   │
│  │                                                       │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐  │   │
│  │  │ Chat Agent   │  │ RAG Service  │  │ Plugins    │  │   │
│  │  │ (SK Kernel)  │──│ (Retrieval + │  │ (Orders,   │  │   │
│  │  │              │  │  Generation) │  │  Returns,  │  │   │
│  │  │              │  │              │  │  Inventory)│  │   │
│  │  └──────────────┘  └──────────────┘  └────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│         │                                                    │
│  ┌──────▼───────────────────────────────────────────────┐   │
│  │         RetailAssistant.Infrastructure                │   │
│  │                                                       │   │
│  │  ┌──────────────┐  ┌──────────────┐                  │   │
│  │  │ Vector Store │  │ Order DB     │                  │   │
│  │  │ (pgvector)   │  │ (EF Core)    │                  │   │
│  │  └──────────────┘  └──────────────┘                  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 📁 Solution Structure

```
Capstone-Project/
├── src/
│   ├── RetailAssistant.API/            # ASP.NET Core Web API
│   │   ├── Controllers/
│   │   │   └── ChatController.cs
│   │   ├── Program.cs
│   │   └── appsettings.json
│   │
│   ├── RetailAssistant.Core/           # Business logic
│   │   ├── Agents/
│   │   │   └── RetailAgent.cs
│   │   ├── Services/
│   │   │   ├── RagService.cs
│   │   │   └── RetrievalService.cs
│   │   ├── Plugins/
│   │   │   ├── OrderPlugin.cs
│   │   │   ├── InventoryPlugin.cs
│   │   │   └── PolicyPlugin.cs
│   │   ├── Models/
│   │   │   └── ChatModels.cs
│   │   └── Interfaces/
│   │       └── IVectorRepository.cs
│   │
│   ├── RetailAssistant.Infrastructure/ # Data access
│   │   ├── Repositories/
│   │   │   └── PostgresVectorRepository.cs
│   │   ├── Data/
│   │   │   └── AppDbContext.cs
│   │   └── Documents/                 # Store policies to ingest
│   │       ├── return-policy.md
│   │       ├── shipping-policy.md
│   │       └── warranty-policy.md
│   │
│   └── RetailAssistant.Tests/         # Tests
│       ├── Services/
│       │   └── RagServiceTests.cs
│       └── Plugins/
│           └── OrderPluginTests.cs
│
└── docs/
    ├── architecture.md
    └── deployment.md
```

---

## ✅ Features to Implement

### Must Have
- [ ] Chat endpoint with streaming response
- [ ] RAG for store policy questions
- [ ] Order lookup plugin
- [ ] Human-in-the-loop for refund processing
- [ ] Chat history management

### Nice to Have
- [ ] Document ingestion endpoint
- [ ] Blazor or HTML frontend
- [ ] SignalR for real-time streaming
- [ ] Telemetry with Application Insights
- [ ] Docker Compose for full stack

---

## 🎉 Course Complete!

Congratulations! You've completed the entire 6-week roadmap:

| Week | Topic | Status |
|------|-------|--------|
| 1 | AI Fundamentals & .NET API Layer | ✅ |
| 2 | Microsoft Semantic Kernel | ✅ |
| 3 | Embeddings & Data Processing | ✅ |
| 4 | Vector Storage & Semantic Search | ✅ |
| 5 | RAG Pipeline | ✅ |
| 6 | Autonomous AI Agents | ✅ |

You now have the skills to build production-grade AI applications with .NET!

**Final Step:** Build out the capstone project in [Capstone-Project/](../../Capstone-Project/README.md)
