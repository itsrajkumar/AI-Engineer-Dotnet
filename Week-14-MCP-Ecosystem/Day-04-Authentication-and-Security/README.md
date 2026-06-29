# Day 4: Authentication & Security

> **Type:** 💻 Code | **Time:** ~2 hours | **Project:** SecureMcpServer
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Secure MCP SSE endpoints using standard ASP.NET Core middleware.
- Implement API Key authentication for Agent-to-Server connections.
- Handle Agent Identity mapping (User vs. Agent permissions).

---

## 📖 Securing the MCP Protocol

If you expose an MCP server to the internet without security, anyone can run your tools and read your data.

Because the HTTP/SSE transport is just standard HTTP, we can use the entire robust ASP.NET Core security ecosystem (Authentication, Authorization, JWTs, API Keys, Rate Limiting) to protect our MCP endpoints.

### 1. API Key Authentication

The most common way to authenticate an autonomous agent is via a secure API Key.

```csharp
using Microsoft.AspNetCore.Builder;
using ModelContextProtocol.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

// Add standard ASP.NET Core Authorization
builder.Services.AddAuthorization();
builder.Services.AddAuthentication("ApiKey")
    .AddScheme<ApiKeyOptions, ApiKeyHandler>("ApiKey", options => { });

builder.Services.AddMcpServer(options => { ... });

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

// =====================================================
// SECURE THE MCP ENDPOINT
// =====================================================
// We simply require authorization on the MCP routes!
app.MapMcpEndpoints("/mcp").RequireAuthorization();

app.Run();
```

### 2. Passing the Key from the Client (MAF)

When your MAF agent connects to the secure server, it must pass the API key in the headers.

```csharp
using ModelContextProtocol.Client;

var mcpClient = new McpHttpClient("https://api.mycompany.com/mcp/sse");

// Add the Authorization header
mcpClient.DefaultRequestHeaders.Add("Authorization", "Bearer agent_abc123");

await mcpClient.ConnectAsync();
```

### 3. Agent Identity vs. User Identity

A critical security concept in 2026 is **The Deputy Problem**.

If Alice asks the Agent to "Read my emails", the Agent connects to the Email MCP Server.
If the Agent authenticates to the MCP Server using *its own* generic API Key, the MCP Server doesn't know it's Alice asking. It might accidentally let the Agent read Bob's emails!

**The Solution:** The Agent must pass the *User's Identity* to the MCP Server.

```csharp
// Inside the MAF Agent (Client)
// Pass Alice's JWT token along with the request
mcpClient.DefaultRequestHeaders.Add("X-On-Behalf-Of", aliceJwtToken);
```

On the server side, your MCP tool checks both:
1. Is this a trusted Agent? (Validate the API Key).
2. Who is the Agent acting for? (Validate Alice's JWT).
3. Does Alice have permission to read this email?

---

## 🔑 Key Takeaways

- Secure MCP HTTP/SSE endpoints exactly like you would secure a standard REST API.
- Use API Keys or OAuth Client Credentials for **Agent Authentication**.
- Be highly aware of the **Confused Deputy Problem**. The MCP server must always verify the permissions of the end-user, not just the agent.

---

## ➡️ Next

Continue to **[Day 5: Enterprise Deployment](../Day-05-Enterprise-Deployment/README.md)**
