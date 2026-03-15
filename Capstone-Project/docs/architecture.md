# Architecture Documentation

## High-Level Architecture

The Retail Assistant follows **Clean Architecture** principles, with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Presentation Layer                           │
│                     (RetailAssistant.API)                        │
│                                                                  │
│   ┌──────────────────┐  ┌──────────────────┐                   │
│   │ ChatEndpoints    │  │ IngestEndpoints  │                   │
│   │ POST /api/chat   │  │ POST /api/ingest │                   │
│   │ GET  /api/history│  │                  │                   │
│   └────────┬─────────┘  └────────┬─────────┘                   │
├────────────┼─────────────────────┼──────────────────────────────┤
│            ▼                     ▼                               │
│                      Core Layer                                  │
│                    (RetailAssistant.Core)                        │
│                                                                  │
│   ┌───────────────────────────────────────────────────────┐     │
│   │                    RetailAgent                         │     │
│   │   Uses Semantic Kernel to orchestrate the response     │     │
│   │                                                        │     │
│   │   ┌─────────────┐ ┌──────────────┐ ┌──────────────┐  │     │
│   │   │ RagService  │ │ OrderPlugin  │ │ RefundPlugin │  │     │
│   │   │ (Retrieval  │ │ (Read-only   │ │ (Human-in-   │  │     │
│   │   │  + Answer)  │ │  database)   │ │  the-Loop)   │  │     │
│   │   └─────────────┘ └──────────────┘ └──────────────┘  │     │
│   └───────────────────────────────────────────────────────┘     │
│                                                                  │
│   Dependencies (Interfaces):                                     │
│   IVectorRepository, IOrderRepository, IChatHistoryStore         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│                   Infrastructure Layer                            │
│               (RetailAssistant.Infrastructure)                   │
│                                                                  │
│   ┌──────────────────┐  ┌──────────────────┐                   │
│   │ PostgresVector   │  │ EFCore Order     │                   │
│   │ Repository       │  │ Repository       │                   │
│   │ (pgvector)       │  │ (Relational)     │                   │
│   └──────────────────┘  └──────────────────┘                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Data Flow

### Chat Request Flow

```
1. Client sends POST /api/chat { "message": "What is your return policy?" }
2. API creates/retrieves ChatHistory for the session
3. RetailAgent receives the message
4. SK Kernel determines what tools to use:
   a. If policy question → RagService.RetrieveAsync() → GenerateAnswer()
   b. If order query → OrderPlugin.GetOrder()
   c. If refund request → RefundPlugin.ProcessRefund() → HumanApprovalFilter
5. Response streamed back to client via SignalR or SSE
```

### Document Ingestion Flow

```
1. Admin sends POST /api/ingest { "filePath": "return-policy.md" }
2. Document is read and chunked (paragraph-based)
3. Each chunk is embedded via IEmbeddingGenerator
4. Chunks + embeddings stored in PostgresVectorRepository
5. Returns: { "chunksProcessed": 15, "status": "success" }
```

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Clean Architecture | Separation of concerns, testable core |
| Repository Pattern | Swap vector DB without changing core |
| SK Filters for HITL | Non-invasive approval gates |
| pgvector over Pinecone | Stay in PostgreSQL ecosystem, no vendor lock |
| Minimal APIs over Controllers | Simpler, modern .NET 8 pattern |
