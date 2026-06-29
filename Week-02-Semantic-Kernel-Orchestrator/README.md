# Week 2: Microsoft Semantic Kernel — The Orchestrator

> **Goal:** Replace Python frameworks like LangChain with Microsoft's enterprise-grade orchestrator.

---

> ⚠️ **IMPORTANT UPDATE (June 2026): Semantic Kernel is Deprecated**
>
> Microsoft Semantic Kernel has been succeeded by **Microsoft Agent Framework (MAF) 1.0 GA**. While SK was foundational from 2023-2025, MAF is now the primary orchestration framework for .NET.
>
> The concepts taught in this week (Plugins, Templates, Memory) are still highly relevant as they translate directly to MAF concepts. We preserve this week for legacy maintenance, but all new projects should use MAF (covered later in the course).

---

## 🎯 Learning Objectives

By the end of this week, you will:
- Initialize and configure the Semantic Kernel
- Create prompt templates (Semantic Functions)
- Write native C# plugins with `[KernelFunction]`
- Enable AI-driven tool calling (function calling)
- Manage conversational state across multiple turns

---

## 📅 Daily Breakdown

| Day | Topic | Type | Time |
|-----|-------|------|------|
| 1 | [Kernel Architecture](./Day-01-Kernel-Architecture/README.md) | 💻 Code | ~3 hrs |
| 2 | [Semantic Functions](./Day-02-Semantic-Functions/README.md) | 💻 Code | ~3 hrs |
| 3 | [Native C# Plugins](./Day-03-Native-CSharp-Plugins/README.md) | 💻 Code | ~3 hrs |
| 4 | [Tool Calling (Function Calling)](./Day-04-Tool-Calling/README.md) | 💻 Code | ~3 hrs |
| 5 | [State & History Management](./Day-05-State-and-History/README.md) | 💻 Code | ~3 hrs |

---

## 🏗️ What is Semantic Kernel?

Semantic Kernel (SK) is Microsoft's **open-source AI orchestration SDK**. It's the .NET equivalent of Python's LangChain, but designed with enterprise patterns in mind.

```
LangChain (Python)          ←→  Semantic Kernel (C#/.NET)
─────────────────────────────────────────────────────────
Chains                      ←→  Prompt Templates / Functions
Tools                       ←→  Plugins (KernelFunction)
Agents                      ←→  Planners / Agents
Memory                      ←→  Memory Stores/ Chat History
Vector Store                ←→  SK Connectors (MongoDB, etc.)
```

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                   SEMANTIC KERNEL                    │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │              KERNEL (DI Container)            │   │
│  │                                                │   │
│  │  ┌──────────┐  ┌──────────┐  ┌────────────┐  │   │
│  │  │ AI       │  │ Plugins  │  │ Memory     │  │   │
│  │  │ Services │  │ (C#      │  │ Stores     │  │   │
│  │  │ (GPT,    │  │  methods │  │ (Vectors,  │  │   │
│  │  │  Azure)  │  │  + tools)│  │  history)  │  │   │
│  │  └──────────┘  └──────────┘  └────────────┘  │   │
│  │                                                │   │
│  │  ┌──────────────────────────────────────────┐  │   │
│  │  │        Prompt Templates                   │  │   │
│  │  │  (Semantic Functions with variables)      │  │   │
│  │  └──────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │              PLANNERS / AGENTS                │   │
│  │  (Auto-orchestrate plugins to achieve goals)  │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

## 🔑 Why Semantic Kernel Over LangChain?

| Feature | Semantic Kernel | LangChain |
|---------|----------------|-----------|
| **Language** | C# (native .NET) | Python (with poor .NET support) |
| **Enterprise DI** | Built on `Microsoft.Extensions.DependencyInjection` | Custom container |
| **Type Safety** | Strongly typed C# plugins | Dynamic Python |
| **Maintainer** | Microsoft (same team as .NET) | 3rd party |
| **Azure Integration** | First-class | Plugin-based |
| **Production ready** | Enterprise-focused | Community-focused |
| **Learning curve** | Familiar to .NET devs | Requires Python knowledge |

---

## ➡️ Start Here

Begin with **[Day 1: Kernel Architecture](./Day-01-Kernel-Architecture/README.md)**
