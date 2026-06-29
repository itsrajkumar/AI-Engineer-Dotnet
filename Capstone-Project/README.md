# 🏆 Capstone Project: AI-Powered Retail Assistant

> A portfolio-worthy project that combines everything from the 14-week roadmap.

---

## 📋 Overview

The **AI-Powered Retail Assistant** is a .NET 10 Web API that:
1. **Answers store policy questions** using Advanced Agentic RAG and GraphRAG
2. **Looks up orders** using AI function calling via MEAI (v10.7.0)
3. **Processes returns and refunds** with human-in-the-loop approval and Guardrails
4. **Handles complex multi-step requests** using Microsoft Agent Framework (MAF)
5. **Connects to external tools** using Model Context Protocol (MCP) and A2A

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| API | ASP.NET Core 10 Minimal APIs |
| AI Orchestration | Microsoft Agent Framework (MAF) 1.0 GA |
| AI Provider | MEAI 10.7.0 (Azure OpenAI / Foundry Local) |
| Vector Database | PostgreSQL + pgvector / MongoDB 8.3 |
| Relational DB | PostgreSQL + EF Core 10 |
| Real-time | Server-Sent Events (SSE) / SignalR |
| Integration | MCP v1.4.0 / A2A Protocol v1.0 |
| Observability | OpenTelemetry / LangFuse |
| Containerization | Docker + Docker Compose |
| Testing | xUnit + DeepEval |

---

## 🚀 Quick Start

```powershell
# Start databases
docker-compose up -d

# Navigate to the API project
cd src/RetailAssistant.API

# Set secrets
dotnet user-secrets init
dotnet user-secrets set "OpenAI:ApiKey" "sk-your-key"

# Run the API
dotnet run

# Open: https://localhost:5001/swagger
```

---

## 📐 Architecture

See [docs/architecture.md](./docs/architecture.md) for detailed architecture documentation.

```
Client Request
      │
      ▼
┌──────────────────┐
│ ASP.NET Core API │
│ POST /api/chat   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Retail Agent    │
│  (MAF Agent)     │
│                  │
│  Decides:        │
│  • Is this a     │──► RAG Pipeline ──► Vector DB (pgvector)
│    policy Q?     │
│  • Is this an    │──► Order Plugin ──► Orders DB (EF Core)
│    order lookup? │
│  • Is this a     │──► Refund Plugin ──► Human Approval ──► Payment API
│    return?       │
└──────────────────┘
```

---

## 📁 Project Structure

```
src/
├── RetailAssistant.API/                 # Presentation layer
├── RetailAssistant.Core/                # Business/domain logic
├── RetailAssistant.Infrastructure/      # Data access
└── RetailAssistant.Tests/               # Unit & integration tests
```

---

## ✅ Implementation Checklist

### Phase 1: Foundation
- [ ] Create solution structure with Clean Architecture
- [ ] Set up PostgreSQL with Docker
- [ ] Configure MAF and MEAI 10.7.0 with DI
- [ ] Create chat endpoint

### Phase 2: RAG
- [ ] Ingest store policies into vector database
- [ ] Build retrieval service
- [ ] Create RAG prompt template
- [ ] Test with policy questions

### Phase 3: Plugins
- [ ] Implement Order lookup plugin
- [ ] Implement Inventory plugin
- [ ] Implement Refund plugin with approval gate
- [ ] Test multi-step scenarios

### Phase 4: Polish
- [ ] Add SignalR streaming
- [ ] Add error handling middleware
- [ ] Add request logging & telemetry
- [ ] Write unit tests
- [ ] Add Swagger documentation
- [ ] Create Docker Compose for full stack

---

## 📖 Further Documentation

- [Architecture Deep Dive](./docs/architecture.md)
- [Deployment Guide](./docs/deployment.md)
