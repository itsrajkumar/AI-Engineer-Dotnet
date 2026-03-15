# Day 5: State & History Management

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Implement chat history management in Semantic Kernel
- Build a sliding window strategy for long conversations
- Implement conversation summarization to compress history
- Store and retrieve chat sessions

---

## 📖 The History Problem

```
Turn 1:   Input ~200 tokens  │  Context window: 128K
Turn 5:   Input ~2,000 tokens │  
Turn 20:  Input ~10,000 tokens │  Getting expensive...
Turn 50:  Input ~30,000 tokens │  Very expensive!
Turn 100: Input ~75,000 tokens │  Near the limit!
```

---

## 💻 Code Sample: History Management Strategies

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

var kernel = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion("gpt-4o-mini", config["OpenAI:ApiKey"]!)
    .Build();

var chatService = kernel.GetRequiredService<IChatCompletionService>();

// =====================================================
// Strategy 1: Sliding Window (keep last N messages)
// =====================================================
var history = new ChatHistory("You are a helpful .NET assistant.");
const int MaxMessages = 10; // Keep last 10 messages (5 turns)

Console.WriteLine("=== Chat with Sliding Window History ===\n");

for (int turn = 0; turn < 15; turn++) // Simulate 15 turns
{
    var userMessage = $"Question about topic {turn + 1}: What is .NET?";
    history.AddUserMessage(userMessage);
    
    // Trim history if too long (keep system + last N)
    while (history.Count > MaxMessages + 1) // +1 for system message
    {
        history.RemoveAt(1); // Remove oldest non-system message
    }
    
    var response = await chatService.GetChatMessageContentAsync(history);
    history.AddAssistantMessage(response.Content ?? "");
    
    Console.WriteLine($"Turn {turn + 1}: History has {history.Count} messages");
}

// =====================================================
// Strategy 2: Summarization (compress old messages)
// =====================================================
Console.WriteLine("\n=== With Summarization ===\n");

async Task<string> SummarizeHistory(ChatHistory hist, Kernel k)
{
    var summaryPrompt = $"""
        Summarize the following conversation in 2-3 sentences, 
        capturing the key topics discussed:
        
        {string.Join("\n", hist.Skip(1).Select(m => $"[{m.Role}]: {m.Content}"))}
        """;
    
    var summary = await k.InvokePromptAsync(summaryPrompt);
    return summary.ToString();
}

var history2 = new ChatHistory("You are a .NET mentor.");

// After every 10 messages, summarize and compress
if (history2.Count > 12)
{
    var summary = await SummarizeHistory(history2, kernel);
    history2.Clear();
    history2.AddSystemMessage("You are a .NET mentor.");
    history2.AddSystemMessage($"Summary of our conversation so far: {summary}");
    Console.WriteLine($"History compressed. Summary: {summary}");
}

Console.WriteLine("\n✅ State management strategies demonstrated!");
```

---

## 🔑 Strategy Comparison

| Strategy | Memory | Cost | Context Quality |
|----------|--------|------|----------------|
| **Full History** | Unlimited growth | Expensive | Perfect |
| **Sliding Window** | Fixed (N messages) | Predictable | Loses old context |
| **Summarization** | Compressed | Moderate | Good approximation |
| **Hybrid** | Best of both | Balanced | Best overall |

---

## 🎉 Week 2 Complete!

You now understand:
- ✅ Semantic Kernel architecture and initialization
- ✅ Prompt templates (Semantic Functions)
- ✅ Native C# plugins with `[KernelFunction]`
- ✅ AI-driven tool calling
- ✅ Conversation state management

**Next:** [Week 3: Embeddings & Data Processing](../../Week-03-Embeddings-and-Data-Processing/README.md)
