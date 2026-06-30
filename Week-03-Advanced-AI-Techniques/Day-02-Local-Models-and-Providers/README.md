# Day 2: Local Models & Providers

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** LocalModelsDemo
>
> 🆕 *Based on [Lesson 5: Local Models](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/05-LocalModels/readme.md) from Generative AI for Beginners .NET v2*

---

## 🎯 Learning Objectives

- Install and run Ollama and Foundry Local v1.2 for local AI model inference
- Understand GGUF quantization for efficient local inference
- Use vLLM for high-throughput containerized model serving
- Connect local models to .NET via `IChatClient`
- Build a hybrid fallback strategy (local + cloud)
- Understand when local models are the right choice

---

## 📖 Why Local Models?

```
Cloud Models:                     Local Models:
──────────────                    ──────────────────
✅ Most capable (GPT-5)           ✅ FREE — no API costs
✅ Always latest version           ✅ 100% private — data never leaves
❌ API costs ($$$)                 ✅ No internet required
❌ Data leaves your machine        ✅ Low latency (no network)
❌ Rate limits                     ❌ Less capable than cloud models
❌ Internet required               ❌ Requires local GPU/RAM
```

### When to Use Local Models

| Scenario | Recommendation |
|----------|---------------|
| Development / prototyping | ✅ Local (save money!) |
| Sensitive data (PII, medical) | ✅ Local (data stays local) |
| Offline environments | ✅ Local (no internet needed) |
| Maximum quality needed | ☁️ Cloud (GPT-5, Claude) |
| Production at scale | ☁️ Cloud (managed infra) |
| Cost-sensitive production | 🔄 Hybrid (local + cloud fallback) |
| Progressive cloud-to-local migration | 🔄 Hybrid + Fine-Tuning (see below) |

---

## 💻 Setting Up Local Providers (2026)

### 1. Ollama (Best for Development)
```powershell
# Install Ollama (Windows)
winget install Ollama.Ollama

# Pull latest 2026 models
ollama pull phi4-mini          # Microsoft's efficient 3.8B model
ollama pull llama4-scout       # Meta's 17B MoE (runs fast locally)
ollama pull deepseek-v4-flash  # Highly optimized reasoning model
ollama pull bge-m3             # 2026 standard for embeddings

# Test it
ollama run llama4-scout "Hello from .NET 10!"
```

### 2. Foundry Local v1.2 (Enterprise Simulation)
Microsoft's Azure AI Foundry Local allows you to simulate the Azure OpenAI environment on your local machine. It provides the exact same endpoints and token metrics as the cloud version, but runs models locally using ONNX Runtime.

### 3. vLLM (Production Local Hosting)
For serving local models in production (e.g., in your own Kubernetes cluster), **vLLM** has become the industry standard due to PagedAttention and continuous batching.

### 4. GGUF Quantization
To run large models on consumer hardware, models are downloaded in **GGUF** format. This format supports extreme quantization (down to 1-bit or 2-bit), allowing massive models like Llama 4 Scout to fit comfortably in 16GB of RAM.

---

## 💻 Code Sample: Local + Cloud Hybrid

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;

// =====================================================
// Day 2: Local Models & Hybrid Fallback
// =====================================================

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

// =====================================================
// Provider 1: Ollama (local, free)
// =====================================================
IChatClient localClient = new OllamaChatClient(
    new Uri("http://localhost:11434"), "phi4-mini");

// =====================================================
// Provider 2: OpenAI (cloud, paid)
// =====================================================
IChatClient cloudClient = new OpenAIClient(config["OpenAI:ApiKey"]!)
    .AsChatClient("gpt-5.4-mini");

// =====================================================
// Hybrid Strategy: Try local first, fall back to cloud
// =====================================================
Console.WriteLine("╔══════════════════════════════════════════════╗");
Console.WriteLine("║  Local Models & Hybrid Fallback Demo         ║");
Console.WriteLine("╚══════════════════════════════════════════════╝\n");

async Task<string> HybridQueryAsync(string prompt)
{
    try
    {
        Console.Write("  🏠 Trying local model... ");
        var sw = System.Diagnostics.Stopwatch.StartNew();
        var response = await localClient.GetResponseAsync(prompt,
            new ChatOptions { MaxOutputTokens = 200 });
        sw.Stop();
        Console.WriteLine($"✅ ({sw.ElapsedMilliseconds}ms)");
        return response.Message.Text ?? "";
    }
    catch (Exception ex)
    {
        Console.WriteLine($"❌ ({ex.Message})");
        Console.Write("  ☁️ Falling back to cloud... ");
        var sw = System.Diagnostics.Stopwatch.StartNew();
        var response = await cloudClient.GetResponseAsync(prompt,
            new ChatOptions { MaxOutputTokens = 200 });
        sw.Stop();
        Console.WriteLine($"✅ ({sw.ElapsedMilliseconds}ms)");
        return response.Message.Text ?? "";
    }
}

// Test the hybrid strategy
var result = await HybridQueryAsync(
    "What is dependency injection in .NET? Answer in 2 sentences.");
Console.WriteLine($"\n  📝 {result}\n");

// =====================================================
// Comparison: Local vs Cloud
// =====================================================
Console.WriteLine("--- Speed Comparison ---\n");

var prompt = "What is the capital of France? One word answer.";

var localSw = System.Diagnostics.Stopwatch.StartNew();
try
{
    var localResult = await localClient.GetResponseAsync(prompt,
        new ChatOptions { Temperature = 0f });
    localSw.Stop();
    Console.WriteLine($"  🏠 Local:  {localResult.Message.Text?.Trim()} ({localSw.ElapsedMilliseconds}ms)");
}
catch
{
    localSw.Stop();
    Console.WriteLine($"  🏠 Local:  unavailable");
}

var cloudSw = System.Diagnostics.Stopwatch.StartNew();
var cloudResult = await cloudClient.GetResponseAsync(prompt,
    new ChatOptions { Temperature = 0f });
cloudSw.Stop();
Console.WriteLine($"  ☁️ Cloud:  {cloudResult.Message.Text?.Trim()} ({cloudSw.ElapsedMilliseconds}ms)");

Console.WriteLine("\n✅ Local models demo complete!");
```

---

## 📊 Popular Local Models for .NET

| Model | Params | Size | Best For | Speed |
|-------|--------|------|----------|-------|
| **phi4-mini** | 3.8B | ~2GB | General chat, coding | ⚡ Fast |
| **llama4-scout** | 17B (MoE)| ~6GB | High quality text | ⚡ Fast |
| **deepseek-v4-flash**| 14B | ~5GB | Coding, logic | ⚡ Fast |
| **mistral-small-4** | 22B | ~8GB | Complex reasoning | 🔷 Medium |
| **phi4-multimodal** | 4B | ~2.5GB | Image/audio analysis | ⚡ Fast |
| **bge-m3** | 567M | ~1.1GB | Embeddings (multilingual)| ⚡⚡ Very fast |

---

## 🔄 The Hybrid LLM Strategy (Production Preview)

The simple local/cloud fallback shown above is just the beginning. In a real production system, the strategy goes much deeper:

1. **Intelligent Routing:** A middleware automatically decides *per-request* whether to use local or cloud based on complexity, data sensitivity, cost budget, and latency requirements.
2. **Data Flywheel:** Every cloud API response is logged to a local database alongside user feedback (thumbs up/down).
3. **Progressive Distillation:** Those high-quality cloud responses become training data. You fine-tune your local model using LoRA (see [Week 12](../../Week-12-Model-Training-and-Alignment/README.md)) so it gradually learns to produce cloud-quality answers.
4. **Gradual Migration:** Over weeks/months, your local model improves. You route more and more traffic locally. Cloud API costs drop 80-90%.

> 📖 **Deep Dive:** For the full architecture, C# code, EF Core logging schema, and the 4-phase migration roadmap, see **[Week 9, Day 6: Hybrid LLM Strategy](../../Week-09-Production-AI-Engineering/Day-06-Hybrid-LLM-Strategy/README.md)**.

---

## 🔑 Key Takeaways

| Concept | Details |
|---------|---------|
| **Ollama** | Run models locally, free, same `IChatClient` |
| **Hybrid fallback** | Try local first, cloud if unavailable |
| **Same interface** | `OllamaChatClient` implements `IChatClient` |
| **No code changes** | Swap providers via config, not code |
| **Cost savings** | Dev/test locally, deploy to cloud for production |

---

## ➡️ Next

Continue to **[Day 3: Fine-Tuning Concepts](../Day-03-Fine-Tuning-Concepts/README.md)**
