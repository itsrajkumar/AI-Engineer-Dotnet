# Day 2: Observability & Monitoring

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** AIObservability

---

## 🎯 Learning Objectives

- Implement OpenTelemetry for AI-powered applications
- Track AI costs: token usage, request latency, error rates
- Build a cost dashboard for AI spending
- Set up distributed tracing across AI middleware pipeline
- Create alerts for anomalous AI behavior

---

## 📖 Why Observability for AI?

```
WITHOUT Observability:              WITH Observability:
───────────────────────            ───────────────────────
"Our AI costs went up 3x"          "User #42 sent 500 prompt
                                    injection attempts at 3am,
Why? 🤷                             costing $47 in tokens"

"The chatbot is slow"              "P95 latency is 4.2s due
                                    to GPT-5 cold starts in
Why? 🤷                             us-east region"

"Sometimes it gives wrong          "Hallucination rate increased
answers"                            12% after switching to
                                    cheaper model yesterday"
Why? 🤷
```

---

## 💻 Code Sample: OpenTelemetry Setup

```csharp
using Microsoft.Extensions.AI;
using OpenTelemetry;
using OpenTelemetry.Metrics;
using OpenTelemetry.Trace;
using OpenTelemetry.Logs;

var builder = WebApplication.CreateBuilder(args);

// =====================================================
// OpenTelemetry Configuration
// =====================================================
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddSource("Microsoft.Extensions.AI")  // AI traces!
        .AddOtlpExporter())                    // → Jaeger/Zipkin
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddMeter("Microsoft.Extensions.AI")   // AI metrics!
        .AddMeter("AICostTracker")             // Custom metrics
        .AddOtlpExporter());                   // → Prometheus

// =====================================================
// AI Client with Telemetry Middleware
// =====================================================
builder.Services.AddChatClient(pipeline => pipeline
    .UseOpenTelemetry()              // Built-in MEAI telemetry
    .Use<CostTrackingMiddleware>()   // Custom cost tracking
    .Use(innerClient));

var app = builder.Build();
```

### Custom Cost Tracking Middleware

```csharp
using Microsoft.Extensions.AI;
using System.Diagnostics;
using System.Diagnostics.Metrics;

/// <summary>
/// Tracks AI costs and publishes them as OpenTelemetry metrics.
/// </summary>
public class CostTrackingMiddleware : DelegatingChatClient
{
    private static readonly Meter s_meter = new("AICostTracker", "1.0.0");
    private static readonly Counter<long> s_inputTokens =
        s_meter.CreateCounter<long>("ai.tokens.input", "tokens", "Input tokens consumed");
    private static readonly Counter<long> s_outputTokens =
        s_meter.CreateCounter<long>("ai.tokens.output", "tokens", "Output tokens generated");
    private static readonly Counter<double> s_estimatedCost =
        s_meter.CreateCounter<double>("ai.cost.estimated_usd", "USD", "Estimated cost in USD");
    private static readonly Histogram<double> s_latency =
        s_meter.CreateHistogram<double>("ai.request.duration_ms", "ms", "Request duration");

    public CostTrackingMiddleware(IChatClient inner) : base(inner) { }

    public override async Task<ChatResponse> GetResponseAsync(
        IList<ChatMessage> chatMessages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        var sw = Stopwatch.StartNew();

        var response = await base.GetResponseAsync(
            chatMessages, options, cancellationToken);

        sw.Stop();

        // Record metrics
        var inputTokens = response.Usage?.InputTokenCount ?? 0;
        var outputTokens = response.Usage?.OutputTokenCount ?? 0;
        var model = response.ModelId ?? "unknown";

        var tags = new TagList { { "model", model } };

        s_inputTokens.Add(inputTokens, tags);
        s_outputTokens.Add(outputTokens, tags);
        s_latency.Record(sw.Elapsed.TotalMilliseconds, tags);

        // Estimate cost (GPT-4o-mini pricing)
        var cost = (inputTokens * 0.00015 / 1000) + (outputTokens * 0.0006 / 1000);
        s_estimatedCost.Add(cost, tags);

        // Add to distributed trace
        Activity.Current?.SetTag("ai.input_tokens", inputTokens);
        Activity.Current?.SetTag("ai.output_tokens", outputTokens);
        Activity.Current?.SetTag("ai.estimated_cost_usd", $"{cost:F6}");
        Activity.Current?.SetTag("ai.model", model);

        return response;
    }
}
```

---

## 📊 Key Metrics to Track

| Metric | Type | Description | Alert Threshold |
|--------|------|-------------|-----------------|
| `ai.tokens.input` | Counter | Total input tokens | >100K/hour |
| `ai.tokens.output` | Counter | Total output tokens | >50K/hour |
| `ai.cost.estimated_usd` | Counter | Running cost | >$10/hour |
| `ai.request.duration_ms` | Histogram | Latency distribution | P95 > 5000ms |
| `ai.errors.count` | Counter | Failed AI requests | >5% error rate |
| `ai.cache.hit_rate` | Gauge | Cache effectiveness | <50% hit rate |

---

## 📈 AI FinOps & Optimization (2026 Standards)

### 1. AI FinOps (Cost-per-Resolved-Outcome)
In 2026, tracking just "tokens" is insufficient. FinOps for AI tracks **cost per resolved outcome**. If an agent spends $0.50 in tokens but resolves a customer service ticket that normally costs $8.00 in human labor, the AI has a positive ROI. Metrics must tie API costs directly to business outcomes (e.g., `ai.cost.per_ticket_resolved`).

### 2. Semantic & Prompt Caching
To reduce costs and latency, production systems heavily utilize:
- **Semantic Caching:** Caching the *intent* of a query using vector similarity. If User A asks "How to install MEAI?" and User B asks "Steps for installing Microsoft Extensions AI", the cache recognizes they mean the same thing and returns the cached answer without calling the LLM.
- **Prompt Caching:** Frontier models (GPT-5, Claude 5) support native prompt caching. If you send the same massive RAG context repeatedly, the provider caches it, slashing input costs by up to 80%.

---

## 🧰 Advanced Observability Platforms

While OpenTelemetry is the foundation, specialized AI observability platforms provide deeper insights:
- **Langfuse V4:** The industry standard for tracing complex agentic applications. V4 adds full MAF integration and real-time cost-per-outcome analytics.
- **Arize Phoenix:** Excellent for tracing and evaluating RAG pipelines, including UMAP visualizations of your vector embeddings to spot semantic drift.

---

## 🔑 Key Takeaways

- **OpenTelemetry** is the standard for AI observability in .NET
- `.UseOpenTelemetry()` middleware adds traces automatically
- Track **token usage** to control costs
- Build **custom metrics** for business-specific monitoring
- Set **alerts** on cost, latency, and error rate thresholds

---

## ➡️ Next

Continue to **[Day 3: Content Safety & Guardrails](../Day-03-Content-Safety-and-Guardrails/README.md)**
