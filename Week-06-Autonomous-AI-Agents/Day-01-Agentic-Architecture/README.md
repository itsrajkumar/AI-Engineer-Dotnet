# Day 1: Agentic Architecture

> **Type:** 📖 Theory | **Time:** ~3 hours
>
> 🆕 *Updated with content from [Lesson 4: AI Agents with Microsoft Agent Framework](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/04-AgentsWithMAF/readme.md) from Generative AI for Beginners .NET v2*

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

| Component | .NET Implementation (SK) | .NET Implementation (MAF 🆕) |
|-----------|-------------------|--------------------------|
| Reasoning | LLM via `IChatCompletionService` | LLM via `IChatClient` |
| Tools | `[KernelFunction]`-decorated C# methods | `AIFunctionFactory.Create()` |
| Planning | `FunctionChoiceBehavior.Auto()` or Planners | Built-in agent orchestration |
| Memory | `ChatHistory` + Vector Store | Thread state management |
| Orchestration | Semantic Kernel `Kernel` | `AgentPipeline`, `GroupChat` |
| External Tools | N/A | Model Context Protocol (MCP) |

### 🆕 Microsoft Agent Framework (MAF 1.0 GA)

The agentic ecosystem evolved rapidly: **AutoGen** (2023) → **AG2** (2025) → **Microsoft Agent Framework (MAF)** (2026).
MAF 1.0 GA (April 2026) is the official, enterprise-ready successor to Semantic Kernel for building agents in .NET. It integrates perfectly with `Microsoft.Extensions.AI`.

![MAF Architecture](../../assets/maf-architecture.png)

In the v4 course, **Microsoft Agent Framework** replaces Semantic Kernel as the primary way to build agents:

```
                    ┌─────────────────────────────────────┐
                    │           MAF AGENT                 │
                    │                                     │
 User Input ────────►│  ┌─────────────────────────────┐   │
                    │  │   LLM + IChatClient           │   │
                    │  │   Plans and decides which      │   │
                    │  │   tools to use                 │   │
                    │  └─────────────────────────────┘   │
                    │           │         │              │
                    │     ┌─────┴─────┬───┴────┐         │
                    │     ▼           ▼        ▼         │
                    │  ┌─────┐   ┌─────┐   ┌─────┐       │
                    │  │Tools│   │State│   │ MCP │       │
                    │  └─────┘   └─────┘   └─────┘       │
                    │                                     │
                    └─────────────────────────────────────┘
                                     │
                                     ▼
                              Response + Actions
```

| Feature | Semantic Kernel | Microsoft Agent Framework |
|---------|-----------------|---------------------------|
| Primary Use | Orchestration + Plugins | Dedicated agent building |
| Multi-Agent | Limited | Built-in (sequential, group chat) |
| Tool Calling | `[KernelFunction]` | `AIFunctionFactory` |
| External Tools | Manual integration | MCP Protocol (standardized) |
| State | `ChatHistory` | Thread-based state management |

> 📚 **Deep dive:** See [Week 7, Day 5: .NET 10 Migration Guide](../../Week-07-Responsible-AI-and-Production/Day-05-DotNet10-Migration-Guide/README.md) for code examples of MAF agents.

---

## 🌐 The 2026 Agent Ecosystem

While we focus on MAF in .NET, you should be aware of the broader ecosystem:
- **OpenAI Agents SDK:** OpenAI's official framework for building native agents, tightly coupled with the Swarm API.
- **Google ADK (Agent Development Kit):** Google's Vertex-integrated agent framework.
- **A2A Protocol v1.0:** The "Agent-to-Agent" protocol, governed by the Linux Foundation and adopted by 150+ organizations, allowing a MAF agent to securely communicate with a Google ADK agent.

---

## ➡️ Next

Continue to **[Day 2: Semantic Kernel Planners](../Day-02-SK-Planners/README.md)**
