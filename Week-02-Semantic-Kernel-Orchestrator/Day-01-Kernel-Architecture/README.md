# Day 1: Kernel Architecture

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Initialize the Semantic Kernel `Kernel` object
- Understand the Kernel as a DI container for AI services
- Register AI services (OpenAI, Azure OpenAI, Ollama)
- Make your first SK-powered API call

---

## 📖 The Kernel Object

The `Kernel` is the **central hub** of Semantic Kernel. Think of it as a specialized `IServiceProvider` that knows about AI services, plugins, and prompt templates.

```csharp
// .NET DI (familiar):
var services = new ServiceCollection();
services.AddSingleton<IMyService, MyService>();
var provider = services.BuildServiceProvider();

// Semantic Kernel (same pattern!):
var builder = Kernel.CreateBuilder();
builder.AddOpenAIChatCompletion("gpt-4o-mini", apiKey);
var kernel = builder.Build();
```

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.SemanticKernel;

// =====================================================
// Day 1: Semantic Kernel — Kernel Architecture
// =====================================================

var config = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var apiKey = config["OpenAI:ApiKey"]
    ?? throw new InvalidOperationException("Set OpenAI:ApiKey");

// =====================================================
// Initialize the Kernel
// =====================================================
var builder = Kernel.CreateBuilder();

// Register AI service (choose one):
// Option A: OpenAI
builder.AddOpenAIChatCompletion(
    modelId: "gpt-4o-mini",
    apiKey: apiKey);

// Option B: Azure OpenAI (commented out)
// builder.AddAzureOpenAIChatCompletion(
//     deploymentName: "gpt-4o",
//     endpoint: config["AzureOpenAI:Endpoint"]!,
//     apiKey: config["AzureOpenAI:ApiKey"]!);

// Add logging (optional but recommended)
builder.Services.AddLogging(b => b
    .AddConsole()
    .SetMinimumLevel(Microsoft.Extensions.Logging.LogLevel.Warning));

// Build the kernel
var kernel = builder.Build();

// =====================================================
// Use the Kernel for simple text generation
// =====================================================
Console.WriteLine("=== Kernel Simple Invocation ===\n");

var result = await kernel.InvokePromptAsync(
    "What are the top 3 design patterns used in .NET? Be brief.");

Console.WriteLine(result);

// =====================================================
// Using Kernel with prompt settings
// =====================================================
Console.WriteLine("\n=== With Execution Settings ===\n");

var settings = new PromptExecutionSettings
{
    ExtensionData = new Dictionary<string, object>
    {
        ["temperature"] = 0.0,
        ["max_tokens"] = 200
    }
};

var preciseResult = await kernel.InvokePromptAsync(
    "List exactly 5 NuGet packages every .NET developer should know.",
    new KernelArguments(settings));

Console.WriteLine(preciseResult);

// =====================================================
// Using Kernel Arguments (template variables)
// =====================================================
Console.WriteLine("\n=== With Template Variables ===\n");

var args = new KernelArguments
{
    ["language"] = "C#",
    ["concept"] = "async/await"
};

var templatedResult = await kernel.InvokePromptAsync(
    "Explain {{$concept}} in {{$language}} in 2 sentences.",
    args);

Console.WriteLine(templatedResult);

Console.WriteLine("\n✅ Kernel Architecture demo complete!");
```

---

## 🔑 Key Architecture Points

| Component | Purpose | .NET Equivalent |
|-----------|---------|----------------|
| `Kernel` | Central orchestrator | `IServiceProvider` |
| `Kernel.CreateBuilder()` | Configure services | `WebApplication.CreateBuilder()` |
| `AddOpenAIChatCompletion()` | Register AI provider | `AddDbContext<>()` |
| `InvokePromptAsync()` | Send prompt | Controller action method |
| `KernelArguments` | Template variables | Route/query parameters |

---

## 📚 References

- [Semantic Kernel Overview](https://learn.microsoft.com/semantic-kernel/overview/)
- [Understanding the Kernel](https://learn.microsoft.com/semantic-kernel/concepts/kernel)
- [SK GitHub Repository](https://github.com/microsoft/semantic-kernel)

---

## ➡️ Next

Continue to **[Day 2: Semantic Functions](../Day-02-Semantic-Functions/README.md)**
