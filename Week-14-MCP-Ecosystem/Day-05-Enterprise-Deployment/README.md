# Day 5: Enterprise Deployment & Capstone

> **Type:** 🏗️ Project | **Time:** ~4 hours
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Objectives

- Containerize your MCP server and your MAF Agent using Docker.
- Write a Docker Compose file to orchestrate the MCP ecosystem.
- Understand Kubernetes scaling for MCP Servers.

---

## 💻 Ecosystem Deployment

In a true enterprise scenario, the Agent and the MCP Server live on different machines.

### 1. Dockerfile for MCP Server

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /app
COPY . .
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:10.0
WORKDIR /app
COPY --from=build /app/out .
ENTRYPOINT ["dotnet", "ItSupport.McpServer.dll"]
```

### 2. Docker Compose (The Ecosystem)

```yaml
version: '3.8'

services:
  # The Backend Tool Provider
  mcp-server:
    build: 
      context: ./src/ItSupport.McpServer
    ports:
      - "5000:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production

  # The AI Agent
  maf-agent:
    build: 
      context: ./src/ItSupport.Agent
    ports:
      - "5001:80"
    environment:
      - OpenAI__ApiKey=${OPENAI_API_KEY}
      - MCP_SERVER_URL=http://mcp-server/mcp/sse
    depends_on:
      - mcp-server
```

### 3. Scaling MCP Servers

MCP Servers built with `ModelContextProtocol.AspNetCore` are entirely stateless HTTP APIs. This means they scale perfectly horizontally in Kubernetes!

If your agents are executing 10,000 database query tools per second, you simply scale the `mcp-server` pods up to handle the HTTP load.

---

## 🏆 Advanced Curriculum Complete!

Congratulations! You have completed the **Advanced AI Expansion Curriculum** (Weeks 11-14).

### What you accomplished:
- **Week 11:** You looked inside the black box, understanding the math and code behind Transformers and MoE.
- **Week 12:** You learned how to train and align models yourself using SFT, DPO, and LoRA.
- **Week 13:** You pushed beyond basic RAG into GraphRAG and Multi-Agent Swarms.
- **Week 14:** You decoupled your agents from your tools using the enterprise Model Context Protocol.

You are now working at the absolute cutting edge of AI Engineering in 2026. 

Thank you for completing the **AI Engineer (.NET) Roadmap**! 🎉
