# Week 14: The MCP Ecosystem

Welcome to the final week of the Advanced AI Expansion Curriculum! 

In Week 8, we briefly touched on the **Model Context Protocol (MCP)**, the open standard for connecting AI models to data sources and tools. In 2026, MCP has become as ubiquitous for AI as HTTP is for the web.

This week, we will do a deep dive into building, securing, and deploying enterprise-grade MCP servers and clients using ASP.NET Core and MAF.

## 🎯 Learning Objectives

By the end of this week, you will be able to:
- Understand the core architecture of the Model Context Protocol (v1.4.0+).
- Build an MCP Server in C# that exposes Resources, Prompts, and Tools.
- Build an MCP Client within a MAF Agent.
- Secure MCP communication using OAuth and API keys.
- Deploy an MCP ecosystem using Docker and Kubernetes.

## 📅 Daily Schedule

| Day | Topic | Description |
|-----|-------|-------------|
| **[Day 1](Day-01-MCP-Architecture/README.md)** | MCP Architecture | Transports (Stdio, SSE), JSON-RPC, and the core primitives (Resources, Prompts, Tools). |
| **[Day 2](Day-02-Building-MCP-Servers/README.md)** | Building MCP Servers | Creating an ASP.NET Core API that implements the MCP specification. |
| **[Day 3](Day-03-MCP-Clients-in-MAF/README.md)** | MCP Clients in MAF | Hooking an external MCP server into a Microsoft Agent Framework workflow. |
| **[Day 4](Day-04-Authentication-and-Security/README.md)** | Authentication & Security | Securing your MCP endpoints and managing agent identity. |
| **[Day 5](Day-05-Enterprise-Deployment/README.md)** | Enterprise Deployment | Scaling MCP servers and handling distributed tool execution. |

## 🛠️ Prerequisites for Week 14

- Advanced C# and ASP.NET Core knowledge.
- Familiarity with the Microsoft Agent Framework (Week 8).
- Install the MCP SDK for .NET:
  ```powershell
  dotnet add package ModelContextProtocol.AspNetCore
  ```

Let's learn how to give our AI agents secure, standardized access to the entire world!

➡️ **[Start Day 1](Day-01-MCP-Architecture/README.md)**
