# Day 4: Human-in-the-Loop

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Implement approval gates for critical AI actions
- Use Semantic Kernel filters for function invocation control
- Build a confirmation workflow for destructive operations
- Understand when and why to add human oversight

---

## 📖 Why Human-in-the-Loop?

```
WITHOUT Human-in-the-Loop:
User: "Issue a refund for order #12345"
Agent: → Calls ProcessRefund(#12345) → $1,299 refunded ← NO CONFIRMATION!

WITH Human-in-the-Loop:
User: "Issue a refund for order #12345"
Agent: → Wants to call ProcessRefund(#12345)
       → PAUSES: "I'd like to process a $1,299 refund for order #12345. Approve? (y/n)"
Human: → Reviews and confirms
Agent: → Calls ProcessRefund(#12345) → $1,299 refunded ✅
```

---

## 💻 Code Sample: SK Function Invocation Filter

```csharp
using Microsoft.SemanticKernel;

/// <summary>
/// Intercepts function calls and requires human approval for dangerous operations.
/// Implements the IFunctionInvocationFilter interface from Semantic Kernel.
/// </summary>
public class HumanApprovalFilter : IFunctionInvocationFilter
{
    // Functions that require human approval before execution
    private readonly HashSet<string> _criticalFunctions = new(StringComparer.OrdinalIgnoreCase)
    {
        "ProcessRefund",
        "DeleteCustomer",
        "UpdatePrice",
        "CancelOrder",
        "SendEmail",
        "ModifyInventory"
    };
    
    public async Task OnFunctionInvocationAsync(
        FunctionInvocationContext context,
        Func<FunctionInvocationContext, Task> next)
    {
        var functionName = context.Function.Name;
        
        if (_criticalFunctions.Contains(functionName))
        {
            // Display what the agent wants to do
            Console.ForegroundColor = ConsoleColor.Red;
            Console.WriteLine($"\n⚠️  APPROVAL REQUIRED");
            Console.WriteLine($"    Function: {functionName}");
            Console.WriteLine($"    Arguments:");
            
            foreach (var arg in context.Arguments)
            {
                Console.WriteLine($"      {arg.Key}: {arg.Value}");
            }
            
            Console.Write("\n    Approve this action? (y/n): ");
            Console.ResetColor();
            
            var approval = Console.ReadLine()?.Trim().ToLower();
            
            if (approval != "y" && approval != "yes")
            {
                // Block the function call
                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine("    ❌ Action blocked by human reviewer.\n");
                Console.ResetColor();
                
                // Set a result that tells the AI the action was denied
                context.Result = new FunctionResult(
                    context.Function,
                    "Action was denied by human reviewer. " +
                    "Inform the user that the action was not performed.");
                return;
            }
            
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("    ✅ Approved. Executing...\n");
            Console.ResetColor();
        }
        else
        {
            // Non-critical functions: log and proceed
            Console.ForegroundColor = ConsoleColor.DarkGray;
            Console.WriteLine($"  🔧 Auto-executing: {functionName}");
            Console.ResetColor();
        }
        
        // Execute the function
        await next(context);
    }
}
```

### Registration

```csharp
var builder = Kernel.CreateBuilder();
builder.AddOpenAIChatCompletion("gpt-4o-mini", apiKey);

// Register the approval filter
builder.Services.AddSingleton<IFunctionInvocationFilter, HumanApprovalFilter>();

var kernel = builder.Build();
kernel.Plugins.AddFromType<OrderPlugin>();

// Now when the agent tries to call ProcessRefund, it will pause!
```

### Order Plugin with Critical Functions

```csharp
public class OrderPlugin
{
    [KernelFunction, Description("Looks up order details by order number")]
    public string GetOrderDetails([Description("Order number")] string orderNumber)
    {
        return $"Order {orderNumber}: Laptop Pro 15, $1,299.00, Status: Delivered";
    }
    
    [KernelFunction, Description("Processes a refund for an order. THIS IS IRREVERSIBLE.")]
    public string ProcessRefund(
        [Description("Order number")] string orderNumber,
        [Description("Reason for refund")] string reason)
    {
        // In reality, this would call your payment processor
        return $"Refund processed for order {orderNumber}. Reason: {reason}. " +
               $"$1,299.00 will be returned in 3-5 business days.";
    }
    
    [KernelFunction, Description("Cancels an active order")]
    public string CancelOrder(
        [Description("Order number")] string orderNumber)
    {
        return $"Order {orderNumber} has been cancelled.";
    }
}
```

---

## 🔑 Approval Strategy Matrix

| Function Type | Auto-Execute? | Human Approval? | Example |
|--------------|--------------|----------------|---------|
| Read-only | ✅ Yes | ❌ No | GetOrderStatus, SearchProducts |
| Calculation | ✅ Yes | ❌ No | CalculateTax, GetDiscount |
| Notification | ⚠️ Depends | ⚠️ Sometimes | SendEmail, SendSMS |
| Data Mutation | ❌ No | ✅ Always | ProcessRefund, DeleteCustomer |
| Financial | ❌ No | ✅ Always | ChargeCard, IssueCredit |

---

## 🛡️ Agent Security & Red Teaming (2026 Standards)

Relying solely on "Human-in-the-Loop" for safety is not enough. You must actively defend against advanced attacks.

### 1. OWASP Top 10 for Agentic Applications
The OWASP foundation released a specific Top 10 for Agents in 2026. The most critical vulnerabilities are:
- **Agentic Prompt Injection:** Attackers hijack the agent's goal via hidden text in a retrieved document.
- **Over-Privileged Tools:** Giving an agent `DeleteCustomer(id)` instead of `RequestCustomerDeletion(id)`.
- **Unbounded Loops:** An agent gets stuck in an infinite retry loop, burning API credits.

### 2. Crescendo-Style Attacks
In a **Crescendo Attack**, the attacker doesn't immediately ask the agent to do something malicious. Instead, they slowly build trust over 20-30 turns, gradually steering the agent's logic until it willingly bypasses safety filters. 
**Defense:** Monitor the *sentiment and intent trajectory* of the conversation, not just individual prompts, and enforce hard context-window resets for sensitive operations.

### 3. Red-Teaming Tools
Before deploying an agent, you must test its defenses using automated red-teaming tools:
- **PyRIT (Python Risk Identification Tool):** Microsoft's open-source tool for finding risks in generative AI.
- **Garak:** An LLM vulnerability scanner that tests for prompt injection and hallucination.
- **DeepTeam:** A 2026 multi-agent red-teaming framework where *attacker agents* actively try to trick your *defender agent*.

---

## ➡️ Next

Continue to **[Day 5: Capstone Project](../Day-05-Capstone-Project/README.md)**
