# Day 1: MCP Architecture

> **Type:** 📖 Theory | **Time:** ~1 hour
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand the problem MCP solves.
- Learn the core primitives: Resources, Prompts, and Tools.
- Understand MCP Transports (Stdio vs. HTTP/SSE).
- Review the JSON-RPC message format.

---

## 📖 The "N x M" Integration Problem

Before MCP, if you wanted an AI agent to read from GitHub, Slack, and your internal SQL database, you had to write custom API integration code for *every* agent framework you used. If you switched from Semantic Kernel to LangChain, you had to rewrite all your tool wrappers.

MCP solves this. It is a universal protocol (like HTTP) but specifically for AI context.

1. **You write the MCP Server once** (e.g., a GitHub MCP Server).
2. **Any MCP Client** (Claude Desktop, a MAF agent, a LangGraph agent) can connect to it instantly without any custom code.

### The Three Primitives

An MCP Server exposes exactly three things to an AI:

1. **Resources (Context):** Data the AI can read. Think of this as a file system for agents.
   *Example:* `github://repo/readme.md` or `database://customers/123`
2. **Prompts (Templates):** Reusable instructions that the server provides to the client.
   *Example:* "Review this pull request and check for security flaws."
3. **Tools (Actions):** Functions the AI can execute.
   *Example:* `create_github_issue(title, body)`

### Transports

How does the AI Agent talk to the MCP Server?

1. **Stdio (Standard Input/Output):** 
   - The Agent process directly spawns the Server process locally (e.g., `node server.js`).
   - They communicate by writing JSON to the console (`stdout`/`stdin`).
   - Extremely fast, highly secure (no network ports open), but requires the server code to be on the same machine as the agent.

2. **HTTP with SSE (Server-Sent Events):**
   - The Agent makes a POST request to an endpoint.
   - The Server streams events back via SSE.
   - Used in production enterprise deployments where the MCP Server is a microservice hosted in Kubernetes, accessed by multiple agents remotely.

### Under the Hood: JSON-RPC 2.0

All communication over MCP is structured as **JSON-RPC 2.0**.

*Example: The Agent asks what tools the server has:*
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}
```

*Example: The Server responds:*
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "query_database",
        "description": "Runs a SELECT statement",
        "inputSchema": { ... }
      }
    ]
  }
}
```

---

## 🔑 Key Takeaways

- **MCP** standardizes how AI agents connect to data and tools.
- It exposes **Resources** (read data), **Prompts** (instructions), and **Tools** (take actions).
- Production .NET systems will primarily use the **HTTP/SSE Transport** to host MCP servers as microservices.

---

## ➡️ Next

Continue to **[Day 2: Building MCP Servers](../Day-02-Building-MCP-Servers/README.md)**
