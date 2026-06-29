# Day 2: Semantic Kernel Planners

> **Type:** 💻 Code | **Time:** ~3 hours

---

> ⚠️ **Update (June 2026):** Semantic Kernel's planners and the `FunctionChoiceBehavior` shown here are now considered legacy for building agents. **Microsoft Agent Framework (MAF)** is the recommended path. This lesson has been updated to show how these concepts migrate to MAF.

## 🎯 Learning Objectives

- Use `FunctionChoiceBehavior.Auto()` for step-by-step planning
- Provide the Kernel with complex goals requiring multiple steps
- See the AI autonomously select and chain plugins
- Understand the evolution from Planners to Function Calling

---

## 📖 Evolution of Planning (.NET Ecosystem)

```text
SK v1.0: StepwisePlanner (deprecated)
  → Explicit planning step, generated plan as text

SK v1.x: FunctionChoiceBehavior.Auto() (legacy agent pattern)
  → LLM natively decides which functions to call via tool calling

MAF 1.0 GA (2026): Agent-Native Architecture (Current)
  → Replaces SK for agent orchestration. Uses standard IChatClient under the hood.
  → Built-in state management, checkpointing, and human-in-the-loop.
```

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using System.ComponentModel;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

var kernel = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion("gpt-4o-mini", config["OpenAI:ApiKey"]!)
    .Build();

// Register multiple plugins
kernel.Plugins.AddFromType<InventoryPlugin>();
kernel.Plugins.AddFromType<PricingPlugin>();
kernel.Plugins.AddFromType<DateTimePlugin>();

// Enable AUTO function calling — the AI will plan and execute!
var settings = new OpenAIPromptExecutionSettings
{
    FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
};

// Observe what the agent does
kernel.FunctionInvoking += (_, args) =>
{
    Console.ForegroundColor = ConsoleColor.Yellow;
    Console.WriteLine($"\n  🔧 Agent calling: {args.Function.Name}");
    Console.ResetColor();
};

// Give the agent a COMPLEX goal that requires multiple steps
var chatHistory = new ChatHistory();
chatHistory.AddSystemMessage("""
    You are a retail assistant agent. You have access to inventory,
    pricing, and date/time tools. Complete the user's request by
    calling the appropriate tools in sequence.
    """);

chatHistory.AddUserMessage("""
    I want to buy 3 of whatever electronics item has the highest stock.
    What's my total including 8% tax? Also tell me today's date.
    """);

var chatService = kernel.GetRequiredService<IChatCompletionService>();

Console.WriteLine("🤖 Agent processing complex request...\n");

var response = await chatService.GetChatMessageContentAsync(
    chatHistory, settings, kernel);

Console.WriteLine($"\n📋 Agent's Final Answer:\n{response.Content}");

// =====================================================
// Plugins
// =====================================================
public class InventoryPlugin
{
    [KernelFunction, Description("Gets current stock count for a product category")]
    public int GetStockCount([Description("Category: Electronics, Clothing, Books")] string category)
    {
        return category.ToLower() switch
        {
            "electronics" => 150,
            "clothing" => 230,
            "books" => 89,
            _ => 0
        };
    }
    
    [KernelFunction, Description("Gets the highest-stock item in a category")]
    public string GetHighestStockItem([Description("Category name")] string category)
    {
        return category.ToLower() switch
        {
            "electronics" => "Wireless Bluetooth Headphones",
            "clothing" => "Classic Fit T-Shirt",
            "books" => "C# in Depth, 4th Edition",
            _ => "Unknown"
        };
    }
    
    [KernelFunction, Description("Gets the price of a specific product")]
    public double GetProductPrice([Description("Full product name")] string productName)
    {
        return productName.ToLower() switch
        {
            var n when n.Contains("headphones") => 79.99,
            var n when n.Contains("t-shirt") => 24.99,
            var n when n.Contains("c# in depth") => 49.99,
            _ => 0
        };
    }
}

public class DateTimePlugin
{
    [KernelFunction, Description("Gets the current date and time")]
    public string GetCurrentDateTime() => DateTime.Now.ToString("dddd, MMMM dd, yyyy HH:mm:ss");
    
    [KernelFunction, Description("Gets the current day of the week")]
    public string GetDayOfWeek() => DateTime.Now.DayOfWeek.ToString();
}
```

---

## 🔑 What Happens Internally

```
User Goal: "Buy 3 of highest stock electronics item, with 8% tax, and today's date"

Agent Internal Steps:
1. I need to know which electronics item has highest stock
   → Calls GetHighestStockItem("Electronics") → "Wireless Bluetooth Headphones"

2. I need the price of that item
   → Calls GetProductPrice("Wireless Bluetooth Headphones") → $79.99

3. I need to calculate total for 3 items
   → Math: 3 × $79.99 = $239.97

4. I need to add 8% tax
   → Math: $239.97 × 1.08 = $259.17

5. I need today's date
   → Calls GetCurrentDateTime() → "Monday, March 16, 2026"

6. Now I can compose my answer!
```

---

## 🚀 Migration to MAF (2026)

In MAF, you don't configure raw execution settings on a `ChatHistory` or `Kernel`. Instead, you define an `Agent` and equip it with tools via `AIFunctionFactory`:

```csharp
// MAF 1.0 GA Agent creation
var inventoryTool = AIFunctionFactory.Create(new InventoryPlugin().GetHighestStockItem, "GetHighestStockItem");
var priceTool = AIFunctionFactory.Create(new InventoryPlugin().GetProductPrice, "GetProductPrice");

var agent = new ChatAgent(chatClient, "RetailAgent")
{
    Instructions = "You are a retail assistant...",
    Tools = { inventoryTool, priceTool }
};

// MAF handles the loop, state, and tool execution automatically!
var response = await agent.InvokeAsync("I want to buy 3 of the highest stock electronics.");
```

---

## ➡️ Next

Continue to **[Day 3: Multi-Plugin Environments](../Day-03-Multi-Plugin-Environments/README.md)**
