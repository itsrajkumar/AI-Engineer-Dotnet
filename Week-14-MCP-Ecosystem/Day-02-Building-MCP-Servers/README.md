# Day 2: Building MCP Servers in .NET

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** McpServerDemo
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Set up a new ASP.NET Core project as an MCP Server.
- Register Resources, Prompts, and Tools using the MCP .NET SDK.
- Configure the HTTP/SSE transport.
- Test your MCP server locally using the MCP Inspector.

---

## 💻 Code Sample: Building an IT Support MCP Server

We will build a simple MCP server that allows agents to view IT tickets and resolve them.

### 1. Setup

```powershell
dotnet new web -n ItSupport.McpServer
cd ItSupport.McpServer
dotnet add package ModelContextProtocol.AspNetCore
```

### 2. Program.cs

```csharp
using Microsoft.AspNetCore.Builder;
using ModelContextProtocol.AspNetCore;
using System.Text.Json;

var builder = WebApplication.CreateBuilder(args);

// Register the MCP Server components
builder.Services.AddMcpServer(options =>
{
    options.ServerName = "IT-Support-System";
    options.ServerVersion = "1.0.0";
});

var app = builder.Build();

// Dummy database
var tickets = new Dictionary<string, string>
{
    ["TKT-1"] = "User's laptop won't turn on.",
    ["TKT-2"] = "Need access to the marketing folder."
};

// =====================================================
// 1. Define RESOURCES (Data the agent can read)
// =====================================================
app.MapMcpResource("ticket://{id}", "An IT Support Ticket", (string id) =>
{
    if (tickets.TryGetValue(id, out var desc))
        return new McpResourceResult(desc);
    return McpResourceResult.NotFound();
});

// =====================================================
// 2. Define PROMPTS (Instructions for the agent)
// =====================================================
app.MapMcpPrompt("triage_ticket", "Instructions for triaging a ticket", () =>
{
    return new McpPromptResult(
        "You are an IT Support Agent. Read the ticket details using the ticket:// resource. " +
        "If it is a hardware issue, escalate it. If it is an access request, use the grant_access tool."
    );
});

// =====================================================
// 3. Define TOOLS (Actions the agent can take)
// =====================================================
app.MapMcpTool("grant_access", "Grants a user access to a folder", (AccessRequest req) =>
{
    Console.WriteLine($"[TOOL EXECUTED] Granted {req.UserId} access to {req.Folder}");
    return new McpToolResult(true, $"Access granted to {req.Folder}");
});

// =====================================================
// Configure the Transport (HTTP/SSE)
// =====================================================
app.MapMcpEndpoints("/mcp");

app.Run();

// DTO for the Tool
record AccessRequest(string UserId, string Folder);
```

### 3. Testing with MCP Inspector

You can test your server without writing an agent by using the official MCP Inspector.

1. Start your .NET application:
   ```powershell
   dotnet run --urls "http://localhost:5000"
   ```
2. Run the MCP inspector using `npx`:
   ```powershell
   npx @modelcontextprotocol/inspector http://localhost:5000/mcp/sse
   ```
3. Open your browser. You can manually browse the Resources, trigger Prompts, and execute the Tools!

---

## 🔑 Key Takeaways

- The `ModelContextProtocol.AspNetCore` package makes building an MCP server feel exactly like building standard minimal APIs.
- You define routes for Resources using `ticket://{id}` URI schemas.
- You can test any MCP server globally using the web-based MCP Inspector.

---

## ➡️ Next

Continue to **[Day 3: MCP Clients in MAF](../Day-03-MCP-Clients-in-MAF/README.md)**
