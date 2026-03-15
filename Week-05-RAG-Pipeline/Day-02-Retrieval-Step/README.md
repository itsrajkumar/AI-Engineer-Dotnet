# Day 2: The Retrieval Step

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Convert a user's question into an embedding
- Search the vector database for relevant chunks
- Filter and rank results by relevance
- Build a retrieval service in C#

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.AI;

/// <summary>
/// Retrieves relevant document chunks from the vector store
/// based on semantic similarity to the user's question.
/// </summary>
public class RetrievalService
{
    private readonly IVectorRepository _vectorRepo;
    private readonly IEmbeddingGenerator<string, Embedding<float>> _embedder;
    
    public RetrievalService(
        IVectorRepository vectorRepo,
        IEmbeddingGenerator<string, Embedding<float>> embedder)
    {
        _vectorRepo = vectorRepo;
        _embedder = embedder;
    }
    
    /// <summary>
    /// Retrieves the most relevant chunks for a given question.
    /// </summary>
    public async Task<RetrievalResult> RetrieveAsync(
        string question,
        int topK = 3,
        float minimumScore = 0.7f,
        string? categoryFilter = null)
    {
        // Step 1: Convert question to embedding
        var questionEmbedding = await _embedder.GenerateEmbeddingAsync(question);
        
        // Step 2: Search vector database
        var searchResults = await _vectorRepo.SearchAsync(
            questionEmbedding.Vector.ToArray(),
            limit: topK * 2,  // Fetch extra, then filter
            categoryFilter: categoryFilter);
        
        // Step 3: Filter by minimum relevance score
        var relevantResults = searchResults
            .Where(r => r.Score >= minimumScore)
            .Take(topK)
            .ToList();
        
        // Step 4: Combine into context string
        var context = string.Join("\n\n---\n\n",
            relevantResults.Select((r, i) =>
                $"[Source {i + 1}: {r.Chunk.Source}]\n{r.Chunk.Text}"));
        
        return new RetrievalResult
        {
            Context = context,
            Chunks = relevantResults,
            HasRelevantResults = relevantResults.Any()
        };
    }
}

public record RetrievalResult
{
    public string Context { get; init; } = "";
    public IReadOnlyList<SearchResult> Chunks { get; init; } = [];
    public bool HasRelevantResults { get; init; }
}
```

### Usage

```csharp
var retrieval = new RetrievalService(vectorRepo, embedder);

var result = await retrieval.RetrieveAsync(
    "How does Dependency Injection work in .NET?",
    topK: 3,
    minimumScore: 0.75f);

if (result.HasRelevantResults)
{
    Console.WriteLine("Retrieved context:");
    Console.WriteLine(result.Context);
    
    Console.WriteLine("\nSources:");
    foreach (var chunk in result.Chunks)
        Console.WriteLine($"  - {chunk.Chunk.Source} (Score: {chunk.Score:F4})");
}
else
{
    Console.WriteLine("No relevant documents found.");
}
```

---

## ➡️ Next

Continue to **[Day 3: The Augmentation Step](../Day-03-Augmentation-Step/README.md)**
