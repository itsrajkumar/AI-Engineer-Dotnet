# Day 1: Agentic Architecture

> **Type:** 📖 Theory | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Understand the ReAct (Reason + Act) pattern
- Know how agents differ from chatbots
- Learn agent architecture patterns
- Map agent concepts to .NET patterns

---

## 📖 The ReAct Pattern

ReAct stands for **Reason + Act**. The agent:
1. **Thinks** about what it needs to do
2. **Acts** by calling a tool/function
3. **Observes** the result
4. **Thinks** again about what to do next
5. Repeats until the goal is achieved

```
User Goal: "What's the total price of 3 laptops with tax for California?"

┌─────────────────────────────────────────────────────────────┐
│ Agent Internal Reasoning:                                    │
│                                                              │
│ THOUGHT: I need to find the laptop price first.             │
│ ACTION:  GetProductPrice("laptop")                          │
│ OBSERVE: $999.99                                            │
│                                                              │
│ THOUGHT: Now I need to multiply by 3.                       │
│ ACTION:  Calculate(999.99 * 3)                              │
│ OBSERVE: $2999.97                                           │
│                                                              │
│ THOUGHT: I need California tax rate.                        │
│ ACTION:  GetTaxRate("California")                           │
│ OBSERVE: 7.25%                                              │
│                                                              │
│ THOUGHT: Now calculate the total with tax.                  │
│ ACTION:  CalculateTax(2999.97, 7.25)                        │
│ OBSERVE: $217.50                                            │
│                                                              │
│ THOUGHT: I have all the information. Total = $3217.47       │
│ FINAL ANSWER: "3 laptops at $999.99 each = $2999.97.       │
│                California tax (7.25%) = $217.50.             │
│                Total: $3,217.47"                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Agent vs. Chatbot vs. RAG

| Feature | Chatbot | RAG System | Agent |
|---------|---------|-----------|-------|
| Steps | 1 (ask → answer) | 2 (retrieve → answer) | N (plan → act → observe → repeat) |
| Autonomy | None | Partial | Full |
| Tool use | No | Search only | Multiple tools |
| Planning | No | No | Yes |
| Memory | Chat history | Vector DB | Chat + Vector + State |
| .NET Analogy | Simple API endpoint | API + database | Background service with decision logic |

---

## 🏗️ Agent Architecture Patterns

### Pattern 1: Single Agent (Simple)
```
User → Agent → [Tools: Search, Calculate, Email] → Response
```

### Pattern 2: Agent + RAG (Most Common in Enterprise)
```
User → Agent → [RAG Tool, Business Logic Tools, API Tools] → Response
```

### Pattern 3: Multi-Agent (Advanced)
```
User → Orchestrator Agent
         ├── Research Agent (searches docs)
         ├── Analysis Agent (processes data)  
         └── Writing Agent (formats response)
```

---

## 🔑 Key Agent Components in .NET

| Component | .NET Implementation |
|-----------|-------------------|
| Reasoning | LLM via `IChatCompletionService` |
| Tools | `[KernelFunction]`-decorated C# methods |
| Planning | `FunctionChoiceBehavior.Auto()` or Planners |
| Memory | `ChatHistory` + Vector Store |
| Orchestration | Semantic Kernel `Kernel` |

---

## ➡️ Next

Continue to **[Day 2: Semantic Kernel Planners](../Day-02-SK-Planners/README.md)**
