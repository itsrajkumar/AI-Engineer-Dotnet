# Day 2: Generating Embeddings in .NET

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Generate embeddings from text using C#
- Use `IEmbeddingGenerator<string, Embedding<float>>` from Microsoft.Extensions.AI
- Visualize and compare embedding outputs
- Understand batch embedding for efficiency

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;

// =====================================================
// Day 2: Generating Embeddings in .NET
// =====================================================

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();
var apiKey = config["OpenAI:ApiKey"]!;

// Create an embedding generator using Microsoft.Extensions.AI
IEmbeddingGenerator<string, Embedding<float>> embeddingGenerator =
    new OpenAIClient(apiKey).AsEmbeddingGenerator("text-embedding-3-small");

// =====================================================
// Single Embedding
// =====================================================
Console.WriteLine("=== Single Embedding ===\n");

var singleEmbedding = await embeddingGenerator.GenerateEmbeddingAsync(
    "Dependency Injection is a design pattern in .NET");

Console.WriteLine($"Dimensions: {singleEmbedding.Vector.Length}");
Console.WriteLine($"First 10 values: [{string.Join(", ", 
    singleEmbedding.Vector.ToArray().Take(10).Select(v => v.ToString("F4")))}]");

// =====================================================
// Batch Embeddings (more efficient for multiple texts)
// =====================================================
Console.WriteLine("\n=== Batch Embeddings ===\n");

var texts = new[]
{
    "C# is a modern programming language",
    "Python is popular for machine learning",
    "JavaScript runs in the browser",
    "SQL is used for database queries",
    "Docker containers virtualize applications"
};

var batchEmbeddings = await embeddingGenerator.GenerateAsync(texts);

foreach (var (text, embedding) in texts.Zip(batchEmbeddings))
{
    Console.WriteLine($"\"{text}\"");
    Console.WriteLine($"  → {embedding.Vector.Length} dimensions");
    Console.WriteLine($"  → First 5: [{string.Join(", ", 
        embedding.Vector.ToArray().Take(5).Select(v => v.ToString("F4")))}]\n");
}

// =====================================================
// Comparing Similarity (preview for Day 4)
// =====================================================
Console.WriteLine("=== Quick Similarity Preview ===\n");

var embA = await embeddingGenerator.GenerateEmbeddingAsync("cat");
var embB = await embeddingGenerator.GenerateEmbeddingAsync("kitten");
var embC = await embeddingGenerator.GenerateEmbeddingAsync("database");

float CosineSimilarity(ReadOnlyMemory<float> a, ReadOnlyMemory<float> b)
{
    var spanA = a.Span;
    var spanB = b.Span;
    float dot = 0, magA = 0, magB = 0;
    for (int i = 0; i < spanA.Length; i++)
    {
        dot += spanA[i] * spanB[i];
        magA += spanA[i] * spanA[i];
        magB += spanB[i] * spanB[i];
    }
    return dot / (MathF.Sqrt(magA) * MathF.Sqrt(magB));
}

Console.WriteLine($"cat ↔ kitten:   {CosineSimilarity(embA.Vector, embB.Vector):F4}");
Console.WriteLine($"cat ↔ database: {CosineSimilarity(embA.Vector, embC.Vector):F4}");
Console.WriteLine("(Higher = more similar)");

Console.WriteLine("\n✅ Embedding generation complete!");
```

---

## 🔑 Key Takeaways

- `IEmbeddingGenerator` is the abstraction — works with OpenAI, Azure, Ollama
- Batch generation is **cheaper and faster** than individual calls
- The resulting `float[]` is what you store in vector databases (Week 4)
- Cosine similarity measures how "close" two embeddings are

---

## ➡️ Next

Continue to **[Day 3: Document Chunking](../Day-03-Document-Chunking/README.md)**
