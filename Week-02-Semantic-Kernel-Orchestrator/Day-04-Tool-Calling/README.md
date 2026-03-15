# Day 4: Tool Calling (Function Calling)

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Enable automatic function calling (tool use)
- Understand how the AI decides which function to call
- See the AI pause, request a function execution, and use the result
- Handle multiple sequential tool calls

---

## 📖 How Tool Calling Works

```
User: "What's the final price of a $250 laptop with the electronics promotion?"

┌──────────────────────────────────────────────────────────┐
│ AI Processing:                                            │
│                                                           │
│ Step 1: "I need to know the promotion for Electronics"   │
│         → Calls: GetCurrentPromotion("Electronics")       │
│         ← Returns: "15% off all electronics this week!"   │
│                                                           │
│ Step 2: "The promotion is 15%. I need to calculate."     │
│         → Calls: CalculateDiscount(250, 15)               │
│         ← Returns: 212.50                                 │
│                                                           │
│ Step 3: "Now I can answer the user."                     │
│         → "The laptop is $212.50 after the 15% discount!" │
└──────────────────────────────────────────────────────────┘
```

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.OpenAI;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

var kernel = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion("gpt-4o-mini", config["OpenAI:ApiKey"]!)
    .Build();

// Register our plugin from Day 3
kernel.Plugins.AddFromType<PricingPlugin>();

// =====================================================
// Enable Auto Function Calling
// =====================================================
var settings = new OpenAIPromptExecutionSettings
{
    // This is the KEY setting — it tells the AI it CAN call your C# methods
    FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
};

Console.WriteLine("Ask pricing questions (the AI will use your C# methods!)");
Console.WriteLine("Example: 'What's the price of a $300 TV after the electronics discount?'\n");

while (true)
{
    Console.Write("You: ");
    var input = Console.ReadLine();
    if (string.IsNullOrWhiteSpace(input) || input == "exit") break;

    var result = await kernel.InvokePromptAsync(input, new KernelArguments(settings));
    Console.WriteLine($"\nAI: {result}\n");
}
```

### With Tool Call Logging

```csharp
// Subscribe to kernel events to see WHEN functions are called

kernel.FunctionInvoking += (sender, args) =>
{
    Console.ForegroundColor = ConsoleColor.Yellow;
    Console.WriteLine($"  🔧 Calling: {args.Function.Name}({string.Join(", ", 
        args.Arguments.Select(a => $"{a.Key}={a.Value}"))})");
    Console.ResetColor();
};

kernel.FunctionInvoked += (sender, args) =>
{
    Console.ForegroundColor = ConsoleColor.Green;
    Console.WriteLine($"  ✅ Result: {args.Result}");
    Console.ResetColor();
};
```

---

## 🔑 Function Choice Behaviors

```csharp
// Auto: AI decides when to call functions (most common)
FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()

// Required: AI MUST call a function (forces tool use)
FunctionChoiceBehavior = FunctionChoiceBehavior.Required()

// None: AI cannot call functions (read-only mode)
FunctionChoiceBehavior = FunctionChoiceBehavior.None()
```

---

## ⚠️ Safety Considerations

The AI is now executing YOUR C# code. Consider:
- What if the AI calls `DeleteUser()` unexpectedly?
- What if it passes malicious parameters?
- What if it calls a function in an infinite loop?

**Solution: Filters** (See Week 6, Day 4: Human-in-the-Loop)

---

## ➡️ Next

Continue to **[Day 5: State & History Management](../Day-05-State-and-History/README.md)**
