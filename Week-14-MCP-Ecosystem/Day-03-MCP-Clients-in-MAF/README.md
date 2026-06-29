# Day 3: MCP Clients in MAF

> **Type:** 💻 Code | **Time:** ~2 hours | **Project:** McpAgentDemo
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Connect a Microsoft Agent Framework (MAF) agent to a remote MCP Server.
- Understand dynamic tool discovery.
- Allow an agent to read remote MCP Resources.

---

## 📖 The Power of Dynamic Discovery

In Week 8 and Week 13, when we built agents, we hardcoded their tools:
`Tools = { AIFunctionFactory.Create(LookupOrder, "LookupOrder") }`

With MCP, you don't hardcode tools. You connect the Agent to an MCP Client. The Client queries the MCP Server ("What tools do you have?"), and automatically maps those remote JSON-RPC tools into local `AITool` objects that the LLM understands.

If the IT Support team adds a new "Reset Password" tool to their MCP Server tomorrow, your agent instantly learns how to reset passwords without you writing a single line of new code or restarting your agent!

---

## 💻 Code Sample: Connecting an Agent to MCP

Let's connect a MAF Agent to the server we built on Day 2.

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Agents.Framework;
using ModelContextProtocol.Client;

// =====================================================
// 1. Initialize the MCP Client
// =====================================================
// Connect via HTTP SSE to the server we built yesterday
var mcpClient = new McpHttpClient("http://localhost:5000/mcp/sse");
await mcpClient.ConnectAsync();

// =====================================================
// 2. Discover Tools from the Server
// =====================================================
var mcpTools = await mcpClient.GetToolsAsAIToolsAsync();
Console.WriteLine($"Discovered {mcpTools.Count} tools from MCP Server.");

// =====================================================
// 3. Configure the Agent
// =====================================================
var chatClient = new OpenAIClient(apiKey).AsChatClient("gpt-5.4-mini");

var agent = new ChatAgent(chatClient, "IT-Agent")
{
    Instructions = "You are a helpful IT support agent.",
    // Assign the dynamically discovered tools to the agent!
    Tools = mcpTools.ToList()
};

// =====================================================
// 4. Run the Agent
// =====================================================
var response = await agent.InvokeAsync("Please grant bob123 access to the marketing folder.");
Console.WriteLine(response.Content);
// The agent will automatically invoke the remote MCP 'grant_access' tool over HTTP!
```

### Reading Resources

Agents can also read resources exposed by the MCP server.

```csharp
// The agent needs context on a ticket before answering the user
var ticketContext = await mcpClient.ReadResourceAsync("ticket://TKT-1");

var response = await agent.InvokeAsync($"""
    The user is asking about their ticket. 
    Here is the ticket context: {ticketContext}
    
    User message: Why is my computer broken?
    """);
```

---

## 🔑 Key Takeaways

- MCP Clients handle the JSON-RPC complexity for you, converting remote MCP configurations into native .NET `AITool` definitions.
- **Dynamic Discovery** means your agents become automatically smarter as your backend teams add more capabilities to their MCP servers, enabling true microservice decoupling.

---

## ➡️ Next

Continue to **[Day 4: Authentication and Security](../Day-04-Authentication-and-Security/README.md)**
