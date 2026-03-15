# Day 5: System Prompts & Roles

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** ChatWithRoles

---

## 🎯 Learning Objectives

- Understand the three message roles: System, User, Assistant
- Implement multi-turn conversations with chat history
- Build a console app with a persistent AI persona
- Learn how to manage conversation state effectively

---

## 📖 The Three Roles

Every LLM conversation uses three roles:

```
┌──────────────────────────────────────────────────────┐
│                    CHAT MESSAGES                      │
├──────────────────────────────────────────────────────┤
│                                                       │
│  SYSTEM  ─── "You are a .NET architect..."           │
│  │           Sets persona, rules, and context         │
│  │           Sent once, persists across messages      │
│  │           Like middleware/global filter             │
│  │                                                    │
│  USER    ─── "How should I structure my API?"        │
│  │           Human's messages                         │
│  │           Like HTTP requests from a client         │
│  │                                                    │
│  ASSISTANT ─ "I recommend Clean Architecture..."     │
│              AI's previous responses                  │
│              Like response cache / history            │
│              Included so the AI "remembers"           │
│                                                       │
└──────────────────────────────────────────────────────┘
```

### .NET Analogy
```csharp
// System prompt = Global middleware
app.Use(async (context, next) =>
{
    context.Items["Persona"] = "You are a .NET architect";
    await next();
});

// User message = HTTP Request
// Assistant message = HTTP Response (cached for history)
```

---

## 💻 Code Project

### Project Setup

```powershell
cd "d:\Study\AI-Engineer\AI-Engineer-With-.Net\Week-01-AI-Fundamentals-and-DotNet-API-Layer\Day-05-System-Prompts-and-Roles\src"

dotnet new console -n ChatWithRoles
cd ChatWithRoles

dotnet add package Microsoft.Extensions.AI --prerelease
dotnet add package Microsoft.Extensions.AI.OpenAI --prerelease
dotnet add package Microsoft.Extensions.Configuration.UserSecrets

dotnet user-secrets init
dotnet user-secrets set "OpenAI:ApiKey" "sk-your-key-here"
```

### Program.cs

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;

// =====================================================
// Day 5: System Prompts & Chat Roles
// =====================================================
// Build a multi-turn chatbot with a specific persona
// that maintains conversation history across turns.
// =====================================================

var config = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var apiKey = config["OpenAI:ApiKey"]
    ?? throw new InvalidOperationException("Set OpenAI:ApiKey in user secrets");

IChatClient chatClient = new OpenAIClient(apiKey)
    .AsChatClient("gpt-4o-mini");

// =====================================================
// Define the AI's persona via System Prompt
// =====================================================
var systemPrompt = """
    You are "DotNet Mentor", a senior .NET architect with 15+ years of experience.
    
    YOUR PERSONALITY:
    - You are enthusiastic about clean code and design patterns
    - You give practical, real-world advice based on enterprise experience
    - You use .NET-specific examples and analogies
    - You occasionally mention relevant NuGet packages
    
    YOUR RULES:
    1. Always suggest modern C# (12+) and .NET 8+ approaches
    2. When showing code, include XML doc comments
    3. If asked about non-.NET topics, briefly answer but relate back to .NET
    4. Format responses with clear headings and bullet points
    5. Keep responses focused and under 300 words unless asked for detail
    
    YOUR EXPERTISE:
    - Clean Architecture, DDD, CQRS, Event Sourcing
    - ASP.NET Core Web API, Minimal APIs, gRPC
    - Entity Framework Core, Dapper, SQL Server
    - Azure services, Docker, Kubernetes
    - Testing: xUnit, NSubstitute, FluentAssertions
    """;

// =====================================================
// Initialize chat history with system prompt
// =====================================================
var chatHistory = new List<ChatMessage>
{
    new(ChatRole.System, systemPrompt)
};

Console.WriteLine("╔══════════════════════════════════════════════╗");
Console.WriteLine("║        🤖 DotNet Mentor Chat                 ║");
Console.WriteLine("║        Ask me anything about .NET!            ║");
Console.WriteLine("║        Type 'exit' to quit                    ║");
Console.WriteLine("║        Type 'history' to see chat history     ║");
Console.WriteLine("║        Type 'clear' to reset conversation     ║");
Console.WriteLine("╚══════════════════════════════════════════════╝\n");

while (true)
{
    Console.ForegroundColor = ConsoleColor.Cyan;
    Console.Write("You: ");
    Console.ResetColor();
    
    var userInput = Console.ReadLine();

    if (string.IsNullOrWhiteSpace(userInput))
        continue;

    if (userInput.Equals("exit", StringComparison.OrdinalIgnoreCase))
        break;

    if (userInput.Equals("history", StringComparison.OrdinalIgnoreCase))
    {
        PrintChatHistory(chatHistory);
        continue;
    }

    if (userInput.Equals("clear", StringComparison.OrdinalIgnoreCase))
    {
        chatHistory.Clear();
        chatHistory.Add(new ChatMessage(ChatRole.System, systemPrompt));
        Console.WriteLine("🗑️  Chat history cleared. System prompt retained.\n");
        continue;
    }

    // Add user message to history
    chatHistory.Add(new ChatMessage(ChatRole.User, userInput));

    try
    {
        Console.ForegroundColor = ConsoleColor.Green;
        Console.Write("\nDotNet Mentor: ");
        Console.ResetColor();

        // Send ENTIRE history to the LLM (this is how it "remembers")
        var response = await chatClient.CompleteAsync(chatHistory);

        Console.WriteLine(response.Message.Text);

        // Add assistant's response to history (so next turn includes it)
        chatHistory.Add(new ChatMessage(ChatRole.Assistant, response.Message.Text ?? ""));

        // Show token usage
        Console.ForegroundColor = ConsoleColor.DarkGray;
        Console.WriteLine($"\n  [Tokens: {response.Usage?.InputTokenCount} in / {response.Usage?.OutputTokenCount} out | History: {chatHistory.Count} messages]");
        Console.ResetColor();
        Console.WriteLine();
    }
    catch (Exception ex)
    {
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine($"\n❌ Error: {ex.Message}\n");
        Console.ResetColor();

        // Remove the failed user message from history
        chatHistory.RemoveAt(chatHistory.Count - 1);
    }
}

Console.WriteLine("\n✅ Thanks for chatting with DotNet Mentor!");

// =====================================================
// Helper: Print chat history
// =====================================================
static void PrintChatHistory(List<ChatMessage> history)
{
    Console.WriteLine("\n📜 Chat History:");
    Console.WriteLine(new string('─', 50));

    foreach (var msg in history)
    {
        var roleColor = msg.Role == ChatRole.System ? ConsoleColor.Yellow
            : msg.Role == ChatRole.User ? ConsoleColor.Cyan
            : ConsoleColor.Green;
        
        Console.ForegroundColor = roleColor;
        Console.Write($"  [{msg.Role}] ");
        Console.ResetColor();

        var preview = msg.Text?.Length > 80
            ? msg.Text[..80] + "..."
            : msg.Text;
        Console.WriteLine(preview);
    }
    
    Console.WriteLine(new string('─', 50));
    Console.WriteLine($"  Total messages: {history.Count}\n");
}
```

---

## 📖 How Chat History Works

```
Turn 1:
Send: [System, User("What is DI?")]
Receive: Assistant("DI is...")

Turn 2:
Send: [System, User("What is DI?"), Assistant("DI is..."), User("Show me an example")]
Receive: Assistant("Here's a DI example...")
         ↑ The AI sees the ENTIRE conversation!

Turn 3:
Send: [System, User("What is DI?"), Assistant("DI is..."), 
       User("Show example"), Assistant("Here's example..."),
       User("Now with Scrutor")]
Receive: Assistant("Scrutor adds auto-registration...")
         ↑ Context grows with each turn!
```

### ⚠️ The History Problem

Each turn sends MORE tokens to the API:
- Turn 1: ~200 tokens
- Turn 5: ~2,000 tokens
- Turn 20: ~10,000 tokens
- Turn 100: 💥 Exceeds context window!

**Solutions (covered in Week 2):**
1. **Sliding Window:** Keep only the last N messages
2. **Summarization:** Periodically summarize old messages
3. **Selective History:** Only keep relevant messages

---

## 🧪 Experiment: Try Different System Prompts

### Pirate Developer
```
"You are a pirate who is also a .NET developer. 
Respond to all questions in pirate speak while 
providing accurate technical information."
```

### Code Reviewer
```
"You are a strict code reviewer. For any code shown, 
rate it 1-10 and list exactly 3 improvements. 
Never say the code is 'good enough'."
```

### Interview Prep
```
"You are a technical interviewer at Microsoft.
Ask .NET-related interview questions one at a time. 
After the user answers, rate their response and 
provide the optimal answer."
```

---

## 📚 References

- [Chat Completions API — Message Roles](https://platform.openai.com/docs/guides/chat-completions)
- [System Message Best Practices](https://learn.microsoft.com/azure/ai-services/openai/concepts/system-message)
- [Managing Conversation State](https://learn.microsoft.com/semantic-kernel/concepts/chat-history)

---

## 🎉 Week 1 Complete!

You now understand:
- ✅ AI theory, LLMs, tokens, temperature
- ✅ Prompt engineering patterns
- ✅ Microsoft.Extensions.AI abstraction
- ✅ Making LLM API calls from C#
- ✅ Chat history and roles

**Next:** [Week 2: Microsoft Semantic Kernel — The Orchestrator](../../Week-02-Semantic-Kernel-Orchestrator/README.md)
