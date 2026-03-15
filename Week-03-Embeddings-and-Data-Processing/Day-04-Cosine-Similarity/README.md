# Day 4: Cosine Similarity

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Understand the math behind semantic search
- Implement cosine similarity in C#
- Compare documents by meaning, not keywords
- Build a simple in-memory semantic search

---

## 📖 What Is Cosine Similarity?

Cosine similarity measures the **angle** between two vectors:
- **1.0** = Identical direction (same meaning)
- **0.0** = Perpendicular (unrelated)
- **-1.0** = Opposite direction (opposite meaning)

### Formula

```
                    A · B           Σ(Ai × Bi)
cos(θ) = ───────────────── = ─────────────────────
              ||A|| × ||B||     √Σ(Ai²) × √Σ(Bi²)
```

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

IEmbeddingGenerator<string, Embedding<float>> embedder =
    new OpenAIClient(config["OpenAI:ApiKey"]!).AsEmbeddingGenerator("text-embedding-3-small");

// =====================================================
// Cosine Similarity Implementation
// =====================================================
static float CosineSimilarity(ReadOnlySpan<float> a, ReadOnlySpan<float> b)
{
    float dot = 0, magA = 0, magB = 0;
    for (int i = 0; i < a.Length; i++)
    {
        dot += a[i] * b[i];
        magA += a[i] * a[i];
        magB += b[i] * b[i];
    }
    return dot / (MathF.Sqrt(magA) * MathF.Sqrt(magB));
}

// =====================================================
// Compare word pairs
// =====================================================
var pairs = new[]
{
    ("cat", "kitten"),
    ("cat", "dog"),
    ("cat", "database"),
    ("C#", "dotnet"),
    ("REST API", "web service"),
    ("happy", "sad"),
    ("Entity Framework", "ORM"),
};

Console.WriteLine("=== Word/Phrase Similarity ===\n");

foreach (var (a, b) in pairs)
{
    var embA = await embedder.GenerateEmbeddingAsync(a);
    var embB = await embedder.GenerateEmbeddingAsync(b);
    var similarity = CosineSimilarity(embA.Vector.Span, embB.Vector.Span);
    
    var bar = new string('█', (int)(similarity * 20));
    Console.WriteLine($"  {a,-20} ↔ {b,-20} = {similarity:F4} {bar}");
}

// =====================================================
// Simple In-Memory Semantic Search
// =====================================================
Console.WriteLine("\n=== In-Memory Semantic Search ===\n");

var knowledgeBase = new[]
{
    "Dependency Injection is a design pattern for achieving IoC in C#",
    "Entity Framework Core is an ORM for .NET that maps objects to databases",
    "ASP.NET Core provides a framework for building web APIs and web apps",
    "xUnit is a popular testing framework for .NET applications",
    "Docker containers package applications with their dependencies",
    "Azure Functions enables serverless compute in the Microsoft cloud",
    "gRPC provides high-performance RPC communication between services",
    "SignalR enables real-time web functionality via WebSockets",
};

// Pre-compute embeddings for knowledge base
Console.WriteLine("Indexing knowledge base...");
var kbEmbeddings = new List<(string Text, Embedding<float> Embedding)>();
foreach (var text in knowledgeBase)
{
    var emb = await embedder.GenerateEmbeddingAsync(text);
    kbEmbeddings.Add((text, emb));
}

// Search!
var queries = new[] { "How do I connect to SQL Server?", "real-time notifications", "unit testing" };

foreach (var query in queries)
{
    var queryEmb = await embedder.GenerateEmbeddingAsync(query);
    
    var results = kbEmbeddings
        .Select(kb => (kb.Text, Score: CosineSimilarity(queryEmb.Vector.Span, kb.Embedding.Vector.Span)))
        .OrderByDescending(r => r.Score)
        .Take(3);
    
    Console.WriteLine($"\nQuery: \"{query}\"");
    foreach (var r in results)
        Console.WriteLine($"  {r.Score:F4} → {r.Text}");
}

Console.WriteLine("\n✅ Cosine similarity demo complete!");
```

---

## 🔑 Interpretation Guide

| Score Range | Meaning |
|------------|---------|
| 0.90 - 1.00 | Nearly identical meaning |
| 0.75 - 0.89 | Very similar / same topic |
| 0.50 - 0.74 | Somewhat related |
| 0.30 - 0.49 | Weakly related |
| 0.00 - 0.29 | Unrelated |

---

## ➡️ Next

Continue to **[Day 5: Batch Processing Pipeline](../Day-05-Batch-Processing-Pipeline/README.md)**
