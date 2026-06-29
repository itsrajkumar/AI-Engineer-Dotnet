# Day 3: Agent Swarms (Orchestrator-Specialist Pattern)

> **Type:** 📖 Theory + 💻 Code | **Time:** ~3 hours | **Project:** SwarmDemo
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand why single agents fail at complex tasks.
- Implement the Orchestrator-Specialist multi-agent pattern.
- Handle agent hand-offs safely.
- Share state between multiple agents in MAF.

---

## 📖 The Orchestrator-Specialist Pattern

If you give a single agent 50 different tools (Database Query, Web Search, Email Send, Code Execute, Math Solver), the LLM will get confused. It will hallucinate tool arguments or use the wrong tool.

**The Solution:** Divide and Conquer.

Instead of one omnipotent agent, you create a "Swarm" of specialized agents.
1. **The Orchestrator:** The only agent the user talks to. It has no tools of its own, except the ability to route the conversation to a Specialist.
2. **The Specialists:** Tiny, highly-focused agents with 1 or 2 tools each. (e.g., A "Database Agent", an "Email Agent").

### How it Works (The Hand-off)

1. User: "Look up John's order and email him the receipt."
2. Orchestrator analyzes the intent and transfers control to the `DatabaseAgent`.
3. `DatabaseAgent` runs the SQL tool, gets the receipt data, and returns it to the Orchestrator.
4. Orchestrator transfers control to the `EmailAgent`, passing the receipt data.
5. `EmailAgent` sends the email and returns success.
6. Orchestrator replies to the User: "I found the order and emailed John the receipt."

---

## 💻 Code Sample: Multi-Agent Swarm in MAF

In the Microsoft Agent Framework (MAF 1.0), agent hand-offs are a native concept.

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Agents.Framework;

// =====================================================
// 1. Define the Specialists
// =====================================================
var dbAgent = new ChatAgent(chatClient, "DatabaseAgent")
{
    Instructions = "You query the database. Return the raw data.",
    Tools = { AIFunctionFactory.Create(QueryDatabase, "QueryDb") }
};

var emailAgent = new ChatAgent(chatClient, "EmailAgent")
{
    Instructions = "You format and send emails.",
    Tools = { AIFunctionFactory.Create(SendEmail, "SendEmail") }
};

// =====================================================
// 2. Define the Orchestrator
// =====================================================
var orchestrator = new ChatAgent(chatClient, "Orchestrator")
{
    Instructions = "You are a retail assistant. Route tasks to the appropriate specialist."
};

// =====================================================
// 3. Configure the Swarm (Routing)
// =====================================================
// Give the orchestrator the ability to call the specialists
var swarm = new AgentSwarm(orchestrator);
swarm.RegisterSpecialist(dbAgent, "Use this to lookup orders or customer data");
swarm.RegisterSpecialist(emailAgent, "Use this to send communications to customers");

// =====================================================
// 4. Run the Swarm
// =====================================================
var response = await swarm.InvokeAsync("Find order #123 and email the receipt to john@test.com");
Console.WriteLine(response.Content);

// Mock Tools
string QueryDatabase(string orderId) => $"Order {orderId}: $50.00";
string SendEmail(string address, string content) => $"Email sent to {address}";
```

---

## 🔑 Key Takeaways

- Never give one agent more than 5-7 tools.
- Use the **Orchestrator-Specialist** pattern to break complex workflows into manageable, isolated steps.
- Specialists should be kept dumb and focused; the Orchestrator maintains the "big picture" context.

---

## ➡️ Next

Continue to **[Day 4: Multi-Agent Frameworks](../Day-04-Multi-Agent-Frameworks/README.md)**
