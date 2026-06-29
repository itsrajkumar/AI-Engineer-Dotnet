# Day 5: Agent Observability & Debugging

> **Type:** 💻 Code | **Time:** ~2 hours | **Project:** SwarmMonitor
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Visualize multi-agent trajectories using Langfuse V4.
- Debug unbounded agent loops.
- Set up Circuit Breakers for agent workflows.

---

## 📖 The "Black Box" Problem

When a user talks to a single LLM, it's easy to debug. You read the prompt, you read the response.

When a user talks to a Swarm of 5 agents, the following might happen in under 3 seconds:
1. Orchestrator routes to WebAgent.
2. WebAgent searches the internet.
3. WebAgent routes to SummaryAgent.
4. SummaryAgent hallucinates.
5. CriticAgent rejects the summary and sends it back to SummaryAgent.
6. SummaryAgent tries again.

If this fails, looking at a standard console log is a nightmare. You need **Agent Trajectory Visualization**.

---

## 💻 Code Sample: Trajectory Logging

In MAF 1.0, you can hook into the agent pipeline to emit rich trace events to an observability platform like Langfuse or Arize Phoenix.

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Agents.Framework;
using OpenTelemetry.Trace;

// =====================================================
// Custom Swarm Event Listener
// =====================================================
public class SwarmObservabilityMiddleware : IAgentMiddleware
{
    public async Task<AgentResponse> InvokeAsync(
        AgentContext context, 
        AgentDelegate next)
    {
        using var activity = Telemetry.Source.StartActivity($"Agent Run: {context.Agent.Name}");
        
        activity?.SetTag("agent.name", context.Agent.Name);
        activity?.SetTag("agent.input_length", context.UserMessage.Length);

        // Record the exact time this specific agent started thinking
        var startTime = DateTime.UtcNow;

        try
        {
            var response = await next(context);
            
            activity?.SetTag("agent.tool_calls_count", response.ToolCalls.Count);
            activity?.SetTag("agent.status", "Success");
            
            return response;
        }
        catch (Exception ex)
        {
            activity?.SetTag("agent.status", "Failed");
            activity?.RecordException(ex);
            throw;
        }
    }
}
```

### 🛑 Circuit Breakers (Preventing Infinite Loops)

The most common critical bug in multi-agent systems is the **Infinite Loop**.
Agent A asks Agent B for data. Agent B returns an error. Agent A asks Agent B again. Repeat until your OpenAI bill is $5,000.

You must implement Circuit Breakers:

```csharp
var swarm = new AgentSwarm(orchestrator)
{
    // CRITICAL: Never let an agent loop indefinitely
    MaxTransitions = 5,
    
    // Fallback if the swarm gets stuck
    FallbackMessage = "I'm sorry, I'm having trouble coordinating with my tools right now. Please try again later."
};
```

---

## 🔑 Key Takeaways

- Never deploy a multi-agent swarm without a visualization tool (Langfuse, Arize Phoenix, or custom OpenTelemetry dashboards).
- Log the **Trajectory** (the exact sequence of agent handoffs and tool calls).
- Always configure strict `MaxTransitions` or **Circuit Breakers** to prevent catastrophic API billing loops.

---

## 🏆 Week 13 Complete!

You have mastered the most advanced concepts in Agentic AI.

Continue to the final module, **[Week 14: Model Context Protocol Deep Dive](../../Week-14-MCP-Ecosystem/README.md)**!
