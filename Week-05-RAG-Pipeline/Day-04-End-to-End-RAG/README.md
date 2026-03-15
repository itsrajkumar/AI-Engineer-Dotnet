# Day 4: End-to-End RAG Implementation

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Wire together retrieval, augmentation, and generation
- Build a complete RAG service class
- Test with real domain-specific questions
- Measure answer quality

---

## 💻 Code Sample: Complete RAG Service

```csharp
using Microsoft.Extensions.AI;
using Microsoft.SemanticKernel;

/// <summary>
/// Complete RAG service that retrieves context and generates grounded answers.
/// </summary>
public class RagService
{
    private readonly RetrievalService _retrieval;
    private readonly Kernel _kernel;
    private readonly KernelFunction _ragFunction;
    
    private static readonly string RagPrompt = """
        <system>
        You are a knowledgeable assistant. Answer questions using ONLY the
        provided context. If the context doesn't contain the answer, say
        "I don't have enough information to answer this question."
        Always cite sources using [Source N] notation.
        </system>
        
        CONTEXT:
        ===
        {{$context}}
        ===
        
        QUESTION: {{$question}}
        
        ANSWER:
        """;
    
    public RagService(RetrievalService retrieval, Kernel kernel)
    {
        _retrieval = retrieval;
        _kernel = kernel;
        _ragFunction = kernel.CreateFunctionFromPrompt(RagPrompt);
    }
    
    public async Task<RagResponse> AskAsync(string question, string? category = null)
    {
        // Step 1: RETRIEVE
        var retrievalResult = await _retrieval.RetrieveAsync(
            question, topK: 3, minimumScore: 0.7f, categoryFilter: category);
        
        if (!retrievalResult.HasRelevantResults)
        {
            return new RagResponse
            {
                Answer = "I couldn't find any relevant information " +
                         "in our documentation to answer your question.",
                Sources = [],
                Grounded = false
            };
        }
        
        // Step 2: AUGMENT + GENERATE
        var result = await _kernel.InvokeAsync(_ragFunction, new KernelArguments
        {
            ["context"] = retrievalResult.Context,
            ["question"] = question
        });
        
        return new RagResponse
        {
            Answer = result.ToString(),
            Sources = retrievalResult.Chunks.Select(c => new SourceInfo
            {
                File = c.Chunk.Source,
                Score = c.Score,
                Preview = c.Chunk.Text[..Math.Min(100, c.Chunk.Text.Length)] + "..."
            }).ToList(),
            Grounded = true
        };
    }
}

public record RagResponse
{
    public string Answer { get; init; } = "";
    public List<SourceInfo> Sources { get; init; } = [];
    public bool Grounded { get; init; }
}

public record SourceInfo
{
    public string File { get; init; } = "";
    public float Score { get; init; }
    public string Preview { get; init; } = "";
}
```

### Usage

```csharp
// Setup (all the DI from previous days)
var ragService = new RagService(retrievalService, kernel);

// Ask a domain-specific question
var response = await ragService.AskAsync("What lifecycle options does DI offer in .NET?");

Console.WriteLine($"Answer: {response.Answer}\n");
Console.WriteLine($"Grounded: {response.Grounded}");
Console.WriteLine("Sources:");
foreach (var source in response.Sources)
    Console.WriteLine($"  📄 {source.File} (relevance: {source.Score:F2})");
```

---

## 🧪 Complete End-to-End Test

```csharp
// Test 1: Question with relevant docs
var r1 = await ragService.AskAsync("How do I register a service as Singleton?");
// Expected: Accurate answer citing DI documentation

// Test 2: Question with NO relevant docs
var r2 = await ragService.AskAsync("What is the weather today?");
// Expected: "I couldn't find any relevant information..."

// Test 3: Category-filtered question
var r3 = await ragService.AskAsync("database access", category: "orm");
// Expected: Only EF Core related content used
```

---

## ➡️ Next

Continue to **[Day 5: Handling Edge Cases](../Day-05-Edge-Cases/README.md)**
