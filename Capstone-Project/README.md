# 🏆 Capstone Project: AI-Powered Retail Assistant

> A portfolio-worthy project that combines everything from the 6-week roadmap.

---

## 📋 Overview

The **AI-Powered Retail Assistant** is a .NET 8 Web API that:
1. **Answers store policy questions** using RAG (Retrieval-Augmented Generation)
2. **Looks up orders** using AI function calling/plugins
3. **Processes returns and refunds** with human-in-the-loop approval
4. **Handles complex multi-step requests** using Semantic Kernel agents

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| API | ASP.NET Core 8 Minimal APIs |
| AI Orchestration | Microsoft Semantic Kernel |
| AI Provider | OpenAI / Azure OpenAI |
| Vector Database | PostgreSQL + pgvector |
| Relational DB | PostgreSQL + EF Core |
| Real-time | SignalR (for streaming) |
| Containerization | Docker + Docker Compose |
| Testing | xUnit + NSubstitute |

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
│  (SK Kernel)     │
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
- [ ] Configure Semantic Kernel with DI
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
