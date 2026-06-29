# Day 1: GraphRAG (Knowledge Graphs)

> **Type:** 📖 Theory + 💻 Code | **Time:** ~3 hours | **Project:** GraphRAGDemo
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand the limitations of standard Vector Search for global reasoning.
- Explain the Microsoft GraphRAG methodology (Entities, Relationships, Communities).
- Implement a basic Knowledge Graph extraction pipeline.
- Query a Knowledge Graph alongside Vector Search (Hybrid GraphRAG).

---

## 📖 The Problem with Standard RAG

Standard RAG answers **Local Questions** well:
*User:* "What is the return policy for laptops?"
*Vector RAG:* Finds the chunk titled "Laptop Returns" and answers perfectly.

Standard RAG fails at **Global Questions**:
*User:* "What are the top 3 themes across all customer complaints this year?"
*Vector RAG:* Can only retrieve the 5 most semantically similar complaints to the query. It literally cannot "see" the rest of the dataset.

### The Solution: GraphRAG

Developed by Microsoft, **GraphRAG** solves this by converting raw text into a Knowledge Graph.

1. **Extract Entities:** An LLM reads every document and extracts Entities (e.g., "John", "Contoso Corp", "Refund Delay").
2. **Extract Relationships:** The LLM notes that "John -> works for -> Contoso Corp".
3. **Build Communities:** The graph is clustered using algorithms (like Leiden) into hierarchal "Communities" (e.g., Theme: Shipping Delays).
4. **Generate Community Summaries:** An LLM generates a summary for each community.

When the user asks a Global Question, the system doesn't search raw text; it searches the *Community Summaries*, allowing it to reason across the entire dataset!

---

## 💻 Code Sample: Graph Extraction

Extracting a Knowledge Graph requires prompting an LLM to output structured nodes and edges.

```csharp
using Microsoft.Extensions.AI;
using System.Text.Json;

// =====================================================
// 1. Define Graph Data Structures
// =====================================================
record Entity(string Id, string Type, string Description);
record Relationship(string SourceId, string TargetId, string Type, string Description);
record KnowledgeGraph(List<Entity> Entities, List<Relationship> Relationships);

// =====================================================
// 2. Extract Graph using Structured Output
// =====================================================
public class GraphExtractor
{
    private readonly IChatClient _chatClient;

    public GraphExtractor(IChatClient chatClient) => _chatClient = chatClient;

    public async Task<KnowledgeGraph> ExtractGraphAsync(string text)
    {
        var prompt = $"""
            Analyze the following text and extract a Knowledge Graph.
            Identify key Entities (People, Organizations, Concepts) and the Relationships between them.
            
            Text to analyze:
            {text}
            """;

        // MEAI 10.7.0 automatically handles JSON schema enforcement
        var response = await _chatClient.GetResponseAsync<KnowledgeGraph>(
            prompt, new ChatOptions { Temperature = 0f });

        return response.Result ?? new KnowledgeGraph([], []);
    }
}

// Usage
var extractor = new GraphExtractor(chatClient);
var text = "Satya Nadella is the CEO of Microsoft. He led the partnership with OpenAI.";
var graph = await extractor.ExtractGraphAsync(text);

foreach (var r in graph.Relationships)
{
    Console.WriteLine($"{r.SourceId} --[{r.Type}]--> {r.TargetId}");
}
// Output: Satya Nadella --[CEO of]--> Microsoft
// Output: Satya Nadella --[led partnership with]--> OpenAI
```

---

## 🔑 Key Takeaways

- **Vector Search** is for finding specific needles in a haystack.
- **GraphRAG** is for understanding the shape of the haystack.
- GraphRAG requires significant upfront compute to build the graph, but provides massively superior answers for complex, global reasoning tasks.
- In 2026, most production enterprise RAG systems use a hybrid approach: searching both the Vector DB and the Knowledge Graph simultaneously.

---

## ➡️ Next

Continue to **[Day 2: RAPTOR and Self-RAG](../Day-02-RAPTOR-and-Self-RAG/README.md)**
