# Week 13: Advanced RAG & Agents

Welcome to Week 13! In Week 5 and Week 7, we covered the basics of Vector Search and RAG. In Week 8, we introduced Autonomous Agents. Now, we are combining and escalating these concepts to the 2026 enterprise standard.

Standard "naive" RAG (chunking text and finding the nearest vector) is no longer sufficient for complex, multi-hop reasoning over entire codebases or corporate wikis. Standard single-agent loops are not robust enough for mission-critical enterprise tasks.

This week, we explore **Agentic RAG**, **GraphRAG**, and **Multi-Agent Swarms**.

## 🎯 Learning Objectives

By the end of this week, you will be able to:
- Implement Microsoft GraphRAG to answer complex, global questions about a dataset.
- Implement RAPTOR (Recursive Abstractive Processing for Tree-Organized Retrieval).
- Build a Self-Reflective RAG pipeline (Self-RAG / Corrective RAG).
- Orchestrate a Swarm of specialized AI agents.
- Monitor and debug complex multi-agent conversations.

## 📅 Daily Schedule

| Day | Topic | Description |
|-----|-------|-------------|
| **[Day 1](Day-01-GraphRAG/README.md)** | GraphRAG | Extracting entities and relationships into Knowledge Graphs for global reasoning. |
| **[Day 2](Day-02-RAPTOR-and-Self-RAG/README.md)** | RAPTOR & Self-RAG | Hierarchical summarization trees and agentic retrieval correction. |
| **[Day 3](Day-03-Agent-Swarms/README.md)** | Agent Swarms | Hand-offs between specialized agents (The Orchestrator-Specialist pattern). |
| **[Day 4](Day-04-Multi-Agent-Frameworks/README.md)** | Multi-Agent Frameworks | Comparing MAF, AutoGen (AG2), and LangGraph in the .NET ecosystem. |
| **[Day 5](Day-05-Agent-Observability/README.md)** | Agent Observability | Visualizing agent trajectories and debugging endless loops. |

## 🛠️ Prerequisites for Week 13

- Solid understanding of MAF (Microsoft Agent Framework) from Week 8.
- Familiarity with Vector Databases (Week 6).
- A graph database (e.g., Neo4j) or an in-memory graph library.

Let's build systems that can reason like a team of experts!

➡️ **[Start Day 1](Day-01-GraphRAG/README.md)**
