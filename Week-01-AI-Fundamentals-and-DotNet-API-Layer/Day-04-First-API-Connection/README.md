# Day 4: Your First API Connection

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** FirstLLMCall

---

## 🎯 Learning Objectives

- Build a complete .NET 8 console app that communicates with an LLM
- Understand the request/response lifecycle of an LLM API call
- Handle errors, rate limiting, and retries properly
- Parse and use the API response in a meaningful way

---

## 📖 Understanding the API Call Lifecycle

```
Your C# Code
    │
    ├─ 1. Construct message(s)
    │     [System prompt, User message]
    │
    ├─ 2. Set parameters
    │     [Model, Temperature, Max tokens]
    │
    ├─ 3. Send HTTP POST request
    │     POST https://api.openai.com/v1/chat/completions
    │     Authorization: Bearer sk-...
    │     Body: { messages, model, temperature, ... }
    │
    ├─ 4. Wait for response
    │     (typically 1-10 seconds depending on complexity)
    │
    ├─ 5. Parse response
    │     { id, model, usage, choices: [{ message, finish_reason }] }
    │
    └─ 6. Use the generated text
          response.Choices[0].Message.Content
```

---

## 💻 Code Project

### Project Setup

```powershell
cd "d:\Study\AI-Engineer\AI-Engineer-With-.Net\Week-01-AI-Fundamentals-and-DotNet-API-Layer\Day-04-First-API-Connection\src"

dotnet new console -n FirstLLMCall
cd FirstLLMCall

dotnet add package Microsoft.Extensions.AI --prerelease
dotnet add package Microsoft.Extensions.AI.OpenAI --prerelease
dotnet add package Microsoft.Extensions.Configuration.UserSecrets
dotnet add package Microsoft.Extensions.DependencyInjection
dotnet add package Microsoft.Extensions.Logging.Console

dotnet user-secrets init
dotnet user-secrets set "OpenAI:ApiKey" "sk-your-key-here"
```

### Program.cs

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using OpenAI;
using System.Text.Json;

// =====================================================
// Day 4: Your First LLM API Connection
// =====================================================
// A complete console app that talks to an LLM.
// Demonstrates: DI setup, error handling, retry logic,
// response parsing, and structured output.
// =====================================================

var config = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

// Set up Dependency Injection (like a real .NET app)
var services = new ServiceCollection();

services.AddLogging(builder => builder
    .AddConsole()
    .SetMinimumLevel(LogLevel.Information));

// Register IChatClient with DI
var apiKey = config["OpenAI:ApiKey"]
    ?? throw new InvalidOperationException("Set OpenAI:ApiKey in user secrets");

services.AddChatClient(builder => builder
    .Use(new OpenAIClient(apiKey).AsChatClient("gpt-4o-mini")));

var serviceProvider = services.BuildServiceProvider();
var chatClient = serviceProvider.GetRequiredService<IChatClient>();
var logger = serviceProvider.GetRequiredService<ILogger<Program>>();

// =====================================================
// Example 1: Basic Text Generation
// =====================================================
Console.WriteLine("╔══════════════════════════════════════════════╗");
Console.WriteLine("║     Day 4: Your First LLM API Connection     ║");
Console.WriteLine("╚══════════════════════════════════════════════╝\n");

logger.LogInformation("Sending first request to LLM...");

try
{
    var response = await chatClient.CompleteAsync(
        "Explain what a .NET Web API is, in exactly 2 sentences.");
    
    Console.WriteLine($"📝 Response:\n{response.Message.Text}\n");
    Console.WriteLine($"📊 Usage: {response.Usage?.InputTokenCount} input + {response.Usage?.OutputTokenCount} output tokens");
    Console.WriteLine($"🤖 Model: {response.ModelId}");
}
catch (Exception ex)
{
    logger.LogError(ex, "API call failed");
}

// =====================================================
// Example 2: Structured JSON Output
// =====================================================
Console.WriteLine("\n--- Example 2: Structured JSON Output ---\n");

var jsonPrompt = """
    Analyze the following .NET package and return a JSON object with these fields:
    - name (string): Package name
    - purpose (string): One sentence description
    - category (string): One of "ORM", "Testing", "AI", "Web", "Utility"
    - popularityScore (int): Estimated 1-10

    Package: Entity Framework Core

    Return ONLY valid JSON, no explanation.
    """;

var jsonResponse = await chatClient.CompleteAsync(jsonPrompt, new ChatOptions
{
    Temperature = 0.0f  // Deterministic for structured output
});

Console.WriteLine($"Raw JSON:\n{jsonResponse.Message.Text}\n");

// Parse the JSON response into a C# record
try
{
    var packageInfo = JsonSerializer.Deserialize<PackageInfo>(
        jsonResponse.Message.Text!,
        new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
    
    Console.WriteLine($"Parsed → Name: {packageInfo?.Name}, Category: {packageInfo?.Category}");
}
catch (JsonException ex)
{
    Console.WriteLine($"JSON parsing failed: {ex.Message}");
}

// =====================================================
// Example 3: Interactive Chat Loop
// =====================================================
Console.WriteLine("\n--- Example 3: Interactive Chat ---");
Console.WriteLine("Type your questions (type 'exit' to quit)\n");

while (true)
{
    Console.Write("You: ");
    var userInput = Console.ReadLine();
    
    if (string.IsNullOrWhiteSpace(userInput) || userInput.Equals("exit", StringComparison.OrdinalIgnoreCase))
        break;
    
    try
    {
        Console.Write("AI: ");
        await foreach (var chunk in chatClient.CompleteStreamingAsync(userInput))
        {
            Console.Write(chunk.Text);
        }
        Console.WriteLine("\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"\n❌ Error: {ex.Message}\n");
    }
}

Console.WriteLine("\n✅ Day 4 Complete!");

// =====================================================
// Models
// =====================================================
public record PackageInfo(string Name, string Purpose, string Category, int PopularityScore);
```

---

## 🔒 Error Handling Best Practices

```csharp
// Common errors and how to handle them:

try
{
    var response = await chatClient.CompleteAsync(prompt);
}
catch (HttpRequestException ex) when (ex.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
{
    // Rate limited — wait and retry
    logger.LogWarning("Rate limited. Waiting 60 seconds...");
    await Task.Delay(TimeSpan.FromSeconds(60));
    // Retry...
}
catch (HttpRequestException ex) when (ex.StatusCode == System.Net.HttpStatusCode.Unauthorized)
{
    // Invalid API key
    logger.LogError("Invalid API key. Check your user secrets.");
    throw;
}
catch (TaskCanceledException)
{
    // Request timeout
    logger.LogWarning("Request timed out. The model may be overloaded.");
}
catch (JsonException ex)
{
    // Response wasn't valid JSON when expected
    logger.LogWarning(ex, "Failed to parse LLM response as JSON. Raw response: {Response}", response);
}
```

---

## 💡 Key Learnings

| Concept | Details |
|---------|---------|
| **DI Registration** | Use `services.AddChatClient()` for proper DI integration |
| **Temperature = 0** | Always use for structured/JSON output to get deterministic results |
| **Streaming** | Use `CompleteStreamingAsync` for better UX in interactive apps |
| **Error Handling** | Always handle rate limiting (429), auth errors (401), and timeouts |
| **Token Counting** | Check `response.Usage` to monitor costs |

---

## 📚 References

- [Azure.AI.OpenAI NuGet](https://www.nuget.org/packages/Azure.AI.OpenAI)
- [OpenAI API Error Codes](https://platform.openai.com/docs/guides/error-codes)
- [Microsoft.Extensions.AI with DI](https://learn.microsoft.com/dotnet/ai/ai-extensions)

---

## ➡️ Next

Continue to **[Day 5: System Prompts & Roles](../Day-05-System-Prompts-and-Roles/README.md)**
