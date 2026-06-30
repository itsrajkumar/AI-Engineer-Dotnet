# Day 6: Hybrid LLM Strategy — Cloud-to-Local Progressive Migration

> **Type:** 🏗️ Architecture + 💻 Code | **Time:** ~4 hours | **Project:** HybridLlmGateway
>
> 🆕 *New for v4.0 (2026) — Bonus Module*

---

## 🎯 Learning Objectives

- Design a Hybrid LLM architecture that uses cloud APIs (Claude, GPT-5) **and** local models (Llama 4, Phi-4) simultaneously.
- Build an Intelligent Routing Middleware that decides *which model* handles each request in real-time.
- Implement a Data Capture Flywheel that logs every interaction to a local database, building a training dataset automatically.
- Understand the Progressive Distillation pipeline: using cloud model outputs to continuously fine-tune your local model.
- Plan and execute a 4-phase migration from "100% cloud" to "90%+ local".

---

## 📖 Why Hybrid?

Most teams start their AI journey by calling a cloud API (OpenAI, Anthropic Claude, Azure OpenAI). This is fast to set up and gives the best quality, but it creates three long-term problems:

1. **Cost:** At scale, per-token API costs become a significant line item. A customer service bot handling 100,000 conversations/day can cost tens of thousands of dollars per month.
2. **Privacy:** Every user message leaves your infrastructure and travels to a third-party server. For regulated industries (healthcare, finance, legal), this is a compliance nightmare.
3. **Dependency:** Your entire product is at the mercy of an external provider's uptime, rate limits, and pricing changes.

The solution is not to abandon cloud models entirely — they are genuinely more capable for complex tasks. The solution is to **gradually build up your local model's capabilities** using the cloud model as a teacher, until the local model can handle the vast majority of requests on its own.

```
                        THE HYBRID FLYWHEEL
    ┌────────────────────────────────────────────────────┐
    │                                                    │
    │  ┌──────────┐    ┌──────────┐    ┌──────────┐     │
    │  │  User     │───▶│  Router  │───▶│  Cloud   │     │
    │  │  Request  │    │ Middleware│    │  API     │     │
    │  └──────────┘    │          │    │(Claude/  │     │
    │                  │ Decides: │    │ GPT-5)   │     │
    │                  │ Local or │    └────┬─────┘     │
    │                  │ Cloud?   │         │           │
    │                  │          │───▶┌────▼─────┐     │
    │                  └─────┬────┘    │  Local   │     │
    │                        │        │  Model   │     │
    │                        │        │(Llama 4) │     │
    │                        │        └────┬─────┘     │
    │                        ▼             │           │
    │                  ┌──────────┐        │           │
    │                  │  Response │◀───────┘           │
    │                  │  + Log   │                    │
    │                  └────┬─────┘                    │
    │                       │                          │
    │                       ▼                          │
    │                  ┌──────────┐                    │
    │                  │  Local   │                    │
    │                  │ Database │ (PostgreSQL)       │
    │                  │ (Logs +  │                    │
    │                  │ Feedback)│                    │
    │                  └────┬─────┘                    │
    │                       │                          │
    │                       ▼  (Weekly/Monthly)        │
    │                  ┌──────────┐                    │
    │                  │ Fine-Tune│                    │
    │                  │ Local    │ (LoRA + QLoRA)     │
    │                  │ Model    │                    │
    │                  └──────────┘                    │
    │                       │                          │
    │        Local model gets smarter ───────────────▶ │
    │        Route MORE traffic to local               │
    └────────────────────────────────────────────────────┘
```

---

## 🏗️ Component 1: The Intelligent Router Middleware

The core of the hybrid strategy is a custom `IChatClient` middleware that sits between your application and the two model providers. It inspects every incoming request and makes a routing decision.

### Routing Criteria

| Criterion | Route to Local | Route to Cloud |
|-----------|---------------|----------------|
| **Task Complexity** | Simple Q&A, classification, extraction, formatting | Complex reasoning, multi-step analysis, creative writing |
| **Data Sensitivity** | Contains PII, medical records, financial data | Generic, non-sensitive queries |
| **Confidence** | Local model has been fine-tuned on this domain | New/unfamiliar domain, first deployment |
| **Cost Budget** | Monthly cloud budget is exhausted | Budget available |
| **Latency** | Real-time requirement (<200ms) | Async/batch processing (latency tolerant) |

### 💻 Code: HybridRoutingMiddleware

```csharp
using Microsoft.Extensions.AI;
using System.Text.RegularExpressions;

/// <summary>
/// An IChatClient middleware that intelligently routes requests
/// between a local model and a cloud API.
/// </summary>
public class HybridRoutingMiddleware : DelegatingChatClient
{
    private readonly IChatClient _localClient;
    private readonly IChatClient _cloudClient;
    private readonly HybridRoutingOptions _options;
    private readonly ILogger<HybridRoutingMiddleware> _logger;

    public HybridRoutingMiddleware(
        IChatClient localClient,
        IChatClient cloudClient,
        HybridRoutingOptions options,
        ILogger<HybridRoutingMiddleware> logger)
        : base(localClient)
    {
        _localClient = localClient;
        _cloudClient = cloudClient;
        _options = options;
        _logger = logger;
    }

    public override async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken ct = default)
    {
        var lastMessage = messages.LastOrDefault(m => m.Role == ChatRole.User);
        var userText = lastMessage?.Text ?? "";

        // =====================================================
        // ROUTING DECISION ENGINE
        // =====================================================
        var route = DecideRoute(userText, messages, options);

        _logger.LogInformation(
            "Routing decision: {Route} | Reason: {Reason} | Prompt length: {Len}",
            route.Target, route.Reason, userText.Length);

        var selectedClient = route.Target switch
        {
            RouteTarget.Local => _localClient,
            RouteTarget.Cloud => _cloudClient,
            _ => _localClient
        };

        try
        {
            var response = await selectedClient.GetResponseAsync(messages, options, ct);

            // Tag the response with routing metadata (for logging)
            response.AdditionalProperties ??= new AdditionalPropertiesDictionary();
            response.AdditionalProperties["route.target"] = route.Target.ToString();
            response.AdditionalProperties["route.reason"] = route.Reason;

            return response;
        }
        catch (Exception ex) when (route.Target == RouteTarget.Local)
        {
            // FALLBACK: If local model fails, try cloud
            _logger.LogWarning(ex, "Local model failed, falling back to cloud");
            return await _cloudClient.GetResponseAsync(messages, options, ct);
        }
    }

    private RoutingDecision DecideRoute(
        string userText, IEnumerable<ChatMessage> messages, ChatOptions? options)
    {
        // Rule 1: PII Detection → Force Local (data never leaves)
        if (ContainsPii(userText))
            return new(RouteTarget.Local, "PII detected — keeping data local");

        // Rule 2: Cloud budget exhausted → Force Local
        if (_options.MonthlyCloudBudgetExhausted)
            return new(RouteTarget.Local, "Monthly cloud budget exhausted");

        // Rule 3: Complex reasoning (long prompt, multi-turn) → Cloud
        var messageCount = messages.Count();
        var totalTokenEstimate = userText.Length / 4; // rough estimate
        if (messageCount > 6 || totalTokenEstimate > 500)
            return new(RouteTarget.Cloud, "Complex multi-turn conversation");

        // Rule 4: Tool/Function calls requested → Cloud (more reliable)
        if (options?.Tools?.Count > 0 && !_options.LocalModelSupportsTools)
            return new(RouteTarget.Cloud, "Tool calling required");

        // Rule 5: Local model confidence (has it been fine-tuned for this domain?)
        if (_options.LocalModelDomains.Any(d =>
            userText.Contains(d, StringComparison.OrdinalIgnoreCase)))
            return new(RouteTarget.Local, "Query matches fine-tuned domain");

        // Default: Use local for simple queries, cloud for everything else
        return totalTokenEstimate < 150
            ? new(RouteTarget.Local, "Simple query — local is sufficient")
            : new(RouteTarget.Cloud, "Moderate complexity — using cloud");
    }

    private static bool ContainsPii(string text)
    {
        // Simplified PII detection (in production, use a proper NER model or regex library)
        var emailPattern = @"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b";
        var phonePattern = @"\b\d{3}[-.]?\d{3}[-.]?\d{4}\b";
        var ssnPattern = @"\b\d{3}-\d{2}-\d{4}\b";

        return Regex.IsMatch(text, emailPattern)
            || Regex.IsMatch(text, phonePattern)
            || Regex.IsMatch(text, ssnPattern);
    }
}

// Supporting types
public enum RouteTarget { Local, Cloud }
public record RoutingDecision(RouteTarget Target, string Reason);

public class HybridRoutingOptions
{
    public bool MonthlyCloudBudgetExhausted { get; set; }
    public bool LocalModelSupportsTools { get; set; }
    public List<string> LocalModelDomains { get; set; } = [];
}
```

### 💻 Code: Registering the Hybrid Pipeline in DI

```csharp
// Program.cs — ASP.NET Core
var builder = WebApplication.CreateBuilder(args);

// 1. Register the LOCAL model client (Ollama)
var localClient = new OllamaChatClient(
    new Uri("http://localhost:11434"), "llama4-scout");

// 2. Register the CLOUD model client (Anthropic Claude)
var cloudClient = new AnthropicChatClient(
    builder.Configuration["Anthropic:ApiKey"]!, "claude-5-fable");

// 3. Register the Hybrid Router as the application's IChatClient
builder.Services.AddSingleton<IChatClient>(sp =>
{
    var logger = sp.GetRequiredService<ILogger<HybridRoutingMiddleware>>();
    var options = new HybridRoutingOptions
    {
        LocalModelDomains = ["return policy", "order status", "shipping"],
        LocalModelSupportsTools = false
    };

    return new ChatClientBuilder(localClient)
        .UseOpenTelemetry()         // Trace EVERY request (local or cloud)
        .Use(inner => new HybridRoutingMiddleware(
            localClient, cloudClient, options, logger))
        .Build();
});
```

> **Key Insight:** Your application code (controllers, agents, services) only ever sees `IChatClient`. It has **zero knowledge** of whether the request was handled locally or in the cloud. The routing is entirely transparent.

---

## 🏗️ Component 2: The Data Capture Flywheel

Every single interaction — whether routed to local or cloud — must be logged to a local database. This log becomes the raw material for fine-tuning your local model.

### 💻 Code: Interaction Logger

```csharp
using Microsoft.EntityFrameworkCore;
using System.Text.Json;

// =====================================================
// 1. Entity Framework Core Model
// =====================================================
public class InteractionLog
{
    public long Id { get; set; }
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;

    // Request
    public string SystemPrompt { get; set; } = "";
    public string UserMessage { get; set; } = "";
    public string ConversationHistory { get; set; } = "[]"; // JSON

    // Response
    public string AssistantResponse { get; set; } = "";
    public string ModelUsed { get; set; } = "";       // e.g., "llama4-scout" or "claude-5-fable"
    public string RouteTarget { get; set; } = "";     // "Local" or "Cloud"
    public string RouteReason { get; set; } = "";     // Why was this route chosen?

    // Metrics
    public int InputTokens { get; set; }
    public int OutputTokens { get; set; }
    public int LatencyMs { get; set; }
    public decimal EstimatedCostUsd { get; set; }

    // Feedback (filled in later by user or automated judge)
    public int? UserRating { get; set; }              // 1-5 stars, null = no feedback
    public string? UserCorrection { get; set; }       // If user rewrote the answer
    public double? JudgeScore { get; set; }           // LLM-as-Judge quality score (0.0 - 1.0)
    public bool? MarkedForTraining { get; set; }      // Has this been flagged as training data?
}

public class HybridDbContext : DbContext
{
    public DbSet<InteractionLog> InteractionLogs => Set<InteractionLog>();

    public HybridDbContext(DbContextOptions<HybridDbContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<InteractionLog>(e =>
        {
            e.HasIndex(l => l.Timestamp);
            e.HasIndex(l => l.RouteTarget);
            e.HasIndex(l => l.MarkedForTraining);
            e.HasIndex(l => l.JudgeScore);
        });
    }
}

// =====================================================
// 2. Logging Middleware (wraps the Hybrid Router)
// =====================================================
public class InteractionLoggingMiddleware : DelegatingChatClient
{
    private readonly HybridDbContext _db;

    public InteractionLoggingMiddleware(IChatClient inner, HybridDbContext db)
        : base(inner) => _db = db;

    public override async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken ct = default)
    {
        var sw = System.Diagnostics.Stopwatch.StartNew();
        var response = await base.GetResponseAsync(messages, options, ct);
        sw.Stop();

        // Extract routing metadata from the response
        var routeTarget = response.AdditionalProperties?
            .GetValueOrDefault("route.target")?.ToString() ?? "Unknown";
        var routeReason = response.AdditionalProperties?
            .GetValueOrDefault("route.reason")?.ToString() ?? "";

        var userMsg = messages.LastOrDefault(m => m.Role == ChatRole.User);
        var systemMsg = messages.FirstOrDefault(m => m.Role == ChatRole.System);

        var log = new InteractionLog
        {
            SystemPrompt = systemMsg?.Text ?? "",
            UserMessage = userMsg?.Text ?? "",
            AssistantResponse = response.Message.Text ?? "",
            ModelUsed = response.ModelId ?? "unknown",
            RouteTarget = routeTarget,
            RouteReason = routeReason,
            LatencyMs = (int)sw.ElapsedMilliseconds,
            InputTokens = (int)(response.Usage?.InputTokenCount ?? 0),
            OutputTokens = (int)(response.Usage?.OutputTokenCount ?? 0),
        };

        _db.InteractionLogs.Add(log);
        await _db.SaveChangesAsync(ct);

        return response;
    }
}
```

### 💻 Code: User Feedback Endpoint

Your application UI should include thumbs-up/thumbs-down buttons. When the user clicks them, this endpoint updates the log:

```csharp
app.MapPost("/api/feedback", async (
    FeedbackRequest request,
    HybridDbContext db) =>
{
    var log = await db.InteractionLogs.FindAsync(request.InteractionId);
    if (log is null) return Results.NotFound();

    log.UserRating = request.Rating;
    log.UserCorrection = request.Correction;

    // If the user provided a correction, this is GOLD for fine-tuning
    if (!string.IsNullOrEmpty(request.Correction))
    {
        log.MarkedForTraining = true;
    }

    await db.SaveChangesAsync();
    return Results.Ok();
});

record FeedbackRequest(long InteractionId, int Rating, string? Correction = null);
```

---

## 🏗️ Component 3: The Automated Quality Judge

You can't wait for users to rate every response. A background worker periodically runs an **LLM-as-a-Judge** evaluation on recent logs.

```csharp
public class QualityJudgeWorker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly IChatClient _judgeClient; // Use cloud model as the judge

    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        while (!ct.IsCancellationRequested)
        {
            using var scope = _scopeFactory.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<HybridDbContext>();

            // Find unjudged interactions from the last 24 hours
            var unjudged = await db.InteractionLogs
                .Where(l => l.JudgeScore == null
                    && l.Timestamp > DateTime.UtcNow.AddHours(-24))
                .Take(50)
                .ToListAsync(ct);

            foreach (var log in unjudged)
            {
                var judgePrompt = $"""
                    You are a strict quality evaluator. Score the following AI response
                    on a scale of 0.0 to 1.0 based on accuracy, helpfulness, and safety.
                    
                    USER QUESTION: {log.UserMessage}
                    AI RESPONSE: {log.AssistantResponse}
                    
                    Output ONLY a JSON object: {{ "score": 0.85, "reason": "..." }}
                    """;

                var result = await _judgeClient.GetResponseAsync(judgePrompt);
                // Parse score and update
                // (simplified — use structured output in production)

                log.JudgeScore = 0.85; // parsed from result
                log.MarkedForTraining = log.JudgeScore >= 0.8; // High-quality = training data
            }

            await db.SaveChangesAsync(ct);
            await Task.Delay(TimeSpan.FromHours(1), ct); // Run hourly
        }
    }
}
```

---

## 🏗️ Component 4: The Training Data Export Pipeline

Once you have accumulated enough high-quality, judged interactions in your local database, you export them as a fine-tuning dataset.

```csharp
// Export high-quality cloud responses as training data for local model
public class TrainingDataExporter
{
    private readonly HybridDbContext _db;

    public async Task ExportToJsonlAsync(string outputPath)
    {
        // Select ONLY high-quality, cloud-generated responses
        // These are the "teacher's answers" we want the local model to learn
        var trainingData = await _db.InteractionLogs
            .Where(l => l.MarkedForTraining == true
                && l.RouteTarget == "Cloud"         // Cloud model gave this answer
                && l.JudgeScore >= 0.85             // High quality
                && l.UserRating == null || l.UserRating >= 4) // User didn't hate it
            .OrderByDescending(l => l.JudgeScore)
            .Take(5000) // Start with 5,000 examples
            .ToListAsync();

        using var writer = File.CreateText(outputPath);

        foreach (var log in trainingData)
        {
            // Convert to standard fine-tuning JSONL format
            var row = new
            {
                messages = new[]
                {
                    new { role = "system", content = log.SystemPrompt },
                    new { role = "user", content = log.UserMessage },
                    new { role = "assistant", content = log.AssistantResponse }
                }
            };

            await writer.WriteLineAsync(
                JsonSerializer.Serialize(row, new JsonSerializerOptions
                {
                    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
                }));
        }

        Console.WriteLine($"Exported {trainingData.Count} training examples to {outputPath}");
    }
}
```

After exporting, you fine-tune your local model using LoRA (see [Week 12, Day 4](../../Week-12-Model-Training-and-Alignment/Day-04-Parameter-Efficient-Fine-Tuning/README.md)):

```bash
# Fine-tune with LoRA using your exported dataset
# (Using a Python tool like unsloth or axolotl, called from your .NET CI/CD pipeline)
python fine_tune.py \
    --base-model meta-llama/Llama-4-Scout-17B \
    --dataset ./training_data.jsonl \
    --method qlora \
    --output ./adapters/v2-retail-specialist/

# Convert to GGUF for Ollama
python convert.py --adapter ./adapters/v2-retail-specialist/ --output retail-v2.gguf

# Import into Ollama
ollama create retail-assistant:v2 -f Modelfile
```

---

## 📋 The 4-Phase Migration Roadmap

| Phase | Cloud % | Local % | Duration | What Changes |
|-------|---------|---------|----------|-------------|
| **Phase 1: Cloud-Primary** | 95% | 5% | Weeks 1-4 | Deploy with cloud API. Local handles only dev/test. Begin logging ALL interactions to local DB. |
| **Phase 2: Hybrid Routing** | 60% | 40% | Weeks 5-12 | Deploy intelligent router. Route simple queries (classification, extraction, FAQ) to local. Cloud handles complex reasoning. First LoRA fine-tune after 2,000+ logged examples. |
| **Phase 3: Local-Primary** | 20% | 80% | Months 2-4 | After 2-3 fine-tuning cycles, local model handles most domain-specific tasks accurately. Cloud is reserved for edge cases, new domains, and as the quality judge. |
| **Phase 4: Cloud-as-Fallback** | 5% | 95% | Month 4+ | Local model is fully domain-specialized. Cloud is used only for catastrophic fallback and periodic quality audits. Monthly cost drops 80-90%. |

### 📊 Measuring Progress

Track these metrics at each phase to know when to advance:

```csharp
// Query your interaction logs to measure local model quality
var stats = await db.InteractionLogs
    .Where(l => l.Timestamp > DateTime.UtcNow.AddDays(-7))
    .GroupBy(l => l.RouteTarget)
    .Select(g => new
    {
        Target = g.Key,
        Count = g.Count(),
        AvgJudgeScore = g.Average(l => l.JudgeScore ?? 0),
        AvgLatencyMs = g.Average(l => l.LatencyMs),
        TotalCostUsd = g.Sum(l => l.EstimatedCostUsd),
        UserSatisfaction = g.Average(l => (double?)l.UserRating ?? 0)
    })
    .ToListAsync();

// When AvgJudgeScore for Local >= 0.85 AND UserSatisfaction >= 4.0,
// you are ready to increase the local routing percentage.
```

---

## ⚠️ Critical Pitfalls

### 1. Model Collapse
If you only train on synthetic/cloud-generated data, your local model may start producing increasingly generic, repetitive outputs. **Always mix in real human-written examples** (at least 20% of the training set).

### 2. Catastrophic Forgetting
When you fine-tune Llama 4 on your retail domain, it might forget how to do general tasks (math, coding). **Use LoRA adapters** (not full fine-tuning) so the base model's general knowledge is preserved.

### 3. Evaluation Drift
Your LLM-as-a-Judge might score differently than a human. **Regularly calibrate** by having a human review a random sample of 50 judged interactions per month.

### 4. Distribution Shift
If your routing logic sends *only* easy queries to local, your training data will be biased towards easy examples. Periodically route a small random sample of complex queries to local (shadow mode) to collect harder training data.

---

## 🔑 Key Takeaways

| Concept | Details |
|---------|---------|
| **Hybrid Router** | An `IChatClient` middleware that transparently routes requests to local or cloud based on rules |
| **Data Flywheel** | Log every interaction → Judge quality → Export high-quality examples → Fine-tune local model |
| **Progressive Migration** | Start 95% cloud → End 95% local over 3-4 months |
| **LoRA Adapters** | Fine-tune without catastrophic forgetting; swap adapters per domain |
| **IChatClient Abstraction** | Your application code never changes — only the routing rules evolve |

---

## 📖 Cross-References

- **[Week 2, Day 5: Middleware Pipelines](../../Week-02-MEAI-Deep-Dive/Day-05-Middleware-Pipelines/README.md)** — The `DelegatingChatClient` pattern used by the router.
- **[Week 3, Day 2: Local Models](../../Week-03-Advanced-AI-Techniques/Day-02-Local-Models-and-Providers/README.md)** — Setting up Ollama and Foundry Local.
- **[Week 9, Day 2: Observability](../Day-02-Observability-and-Monitoring/README.md)** — OpenTelemetry tracing for routing decisions.
- **[Week 12, Day 4: LoRA/QLoRA](../../Week-12-Model-Training-and-Alignment/Day-04-Parameter-Efficient-Fine-Tuning/README.md)** — The fine-tuning technique used in the distillation pipeline.
- **[Week 12, Day 5: Synthetic Data](../../Week-12-Model-Training-and-Alignment/Day-05-Synthetic-Data-Generation/README.md)** — Generating training data from cloud model outputs.

---

## ➡️ Next

Return to **[Week 9 Overview](../README.md)** or continue to **[Week 10: Responsible AI](../../Week-10-Responsible-AI-and-Capstone/README.md)**
