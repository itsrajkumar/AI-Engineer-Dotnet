# Day 4: Hybrid Search Integration

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Combine traditional keyword search with semantic vector search
- Implement pre-filtering and post-filtering strategies
- Understand Reciprocal Rank Fusion (RRF)
- Build a hybrid search service in C#

---

## 📖 Why Hybrid Search?

Neither keyword search nor semantic search is perfect alone:

| Scenario | Keyword Search | Semantic Search | Hybrid Search |
|----------|---------------|-----------------|---------------|
| Exact product SKU: "SKU-12345" | ✅ Perfect | ❌ May miss | ✅ Best of both |
| "How to handle errors" | ❌ Misses "exception handling" | ✅ Finds semantically | ✅ Best of both |
| Category filter + meaning | ❌ Only filters | ❌ No filtering | ✅ Filter + semantic |

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using Npgsql;
using OpenAI;
using Pgvector;
using Pgvector.Npgsql;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();
var connString = config["PostgreSQL:ConnectionString"]
    ?? "Host=localhost;Database=ai_engineer;Username=postgres;Password=postgres";

var dataSourceBuilder = new NpgsqlDataSourceBuilder(connString);
dataSourceBuilder.UseVector();
await using var dataSource = dataSourceBuilder.Build();
await using var conn = await dataSource.OpenConnectionAsync();

IEmbeddingGenerator<string, Embedding<float>> embedder =
    new OpenAIClient(config["OpenAI:ApiKey"]!).AsEmbeddingGenerator("text-embedding-3-small");

// =====================================================
// Hybrid Search: Keyword Filter + Semantic Search
// =====================================================
async Task<List<SearchResult>> HybridSearchAsync(
    NpgsqlConnection conn,
    string query,
    string? categoryFilter = null,
    int limit = 5)
{
    var queryEmb = await embedder.GenerateEmbeddingAsync(query);
    var queryVector = new Vector(queryEmb.Vector.ToArray());

    var sql = """
        SELECT text, category,
               1 - (embedding <=> $1) as semantic_score,
               ts_rank(to_tsvector('english', text), plainto_tsquery('english', $2)) as keyword_score
        FROM document_chunks
        WHERE ($3::varchar IS NULL OR category = $3)
        ORDER BY 
            -- Combine both scores with weighting
            (0.7 * (1 - (embedding <=> $1))) + 
            (0.3 * ts_rank(to_tsvector('english', text), plainto_tsquery('english', $2))) DESC
        LIMIT $4
        """;

    await using var cmd = conn.CreateCommand();
    cmd.CommandText = sql;
    cmd.Parameters.AddWithValue(queryVector);
    cmd.Parameters.AddWithValue(query);
    cmd.Parameters.AddWithValue(categoryFilter ?? (object)DBNull.Value);
    cmd.Parameters.AddWithValue(limit);

    var results = new List<SearchResult>();
    await using var reader = await cmd.ExecuteReaderAsync();
    while (await reader.ReadAsync())
    {
        results.Add(new SearchResult
        {
            Text = reader.GetString(0),
            Category = reader.GetString(1),
            SemanticScore = reader.GetFloat(2),
            KeywordScore = reader.GetFloat(3)
        });
    }
    return results;
}

// Example usage
Console.WriteLine("=== Hybrid Search: Semantic + Keyword + Category Filter ===\n");

var results = await HybridSearchAsync(conn, "database queries", categoryFilter: "orm");
foreach (var r in results)
{
    Console.WriteLine($"  Text: {r.Text}");
    Console.WriteLine($"  Semantic: {r.SemanticScore:F4} | Keyword: {r.KeywordScore:F4}");
    Console.WriteLine($"  Category: {r.Category}\n");
}

public record SearchResult
{
    public string Text { get; init; } = "";
    public string Category { get; init; } = "";
    public float SemanticScore { get; init; }
    public float KeywordScore { get; init; }
}
```

---

## 🔑 Hybrid Search Strategies

```
Strategy 1: Pre-filter then vector search
  WHERE category = 'orm' → then vector similarity
  Best for: Reducing search space, guaranteed metadata match

Strategy 2: Vector search then post-filter
  Vector similarity top 100 → then WHERE category = 'orm'
  Best for: When filter is loose and you want best semantic match

Strategy 3: Weighted combination (shown above)
  Score = 0.7 * semantic_score + 0.3 * keyword_score
  Best for: General purpose, tunable quality
```

---

## ➡️ Next

Continue to **[Day 5: Repository Pattern for Vectors](../Day-05-Repository-Pattern-Vectors/README.md)**
