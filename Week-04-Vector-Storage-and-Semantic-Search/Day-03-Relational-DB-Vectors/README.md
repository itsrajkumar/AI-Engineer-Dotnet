# Day 3: Relational DB & Vectors (PostgreSQL + pgvector)

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Set up PostgreSQL with the pgvector extension
- Store embeddings in a relational database
- Perform vector similarity search with SQL
- Use familiar C# data access patterns (Npgsql)

---

## 🐳 Docker Setup

```powershell
# Start PostgreSQL with pgvector
docker run -d --name pgvector-db `
  -e POSTGRES_PASSWORD=postgres `
  -e POSTGRES_DB=ai_engineer `
  -p 5432:5432 `
  pgvector/pgvector:pg16
```

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

// =====================================================
// Database setup
// =====================================================
var connString = config["PostgreSQL:ConnectionString"]
    ?? "Host=localhost;Database=ai_engineer;Username=postgres;Password=postgres";

var dataSourceBuilder = new NpgsqlDataSourceBuilder(connString);
dataSourceBuilder.UseVector(); // Enable pgvector support
await using var dataSource = dataSourceBuilder.Build();

await using var conn = await dataSource.OpenConnectionAsync();

// Create extension and table
await using (var cmd = conn.CreateCommand())
{
    cmd.CommandText = """
        CREATE EXTENSION IF NOT EXISTS vector;
        
        DROP TABLE IF EXISTS document_chunks;
        CREATE TABLE document_chunks (
            id SERIAL PRIMARY KEY,
            text TEXT NOT NULL,
            source VARCHAR(255),
            category VARCHAR(100),
            embedding vector(1536),  -- pgvector column type!
            created_at TIMESTAMP DEFAULT NOW()
        );
        
        -- Create HNSW index for fast similarity search 
        CREATE INDEX ON document_chunks 
            USING hnsw (embedding vector_cosine_ops);
        """;
    await cmd.ExecuteNonQueryAsync();
}

Console.WriteLine("✅ Database and table created with pgvector.\n");

// Embedding generator
IEmbeddingGenerator<string, Embedding<float>> embedder =
    new OpenAIClient(config["OpenAI:ApiKey"]!).AsEmbeddingGenerator("text-embedding-3-small");

// =====================================================
// Insert documents with embeddings
// =====================================================
var documents = new[]
{
    ("Dependency Injection decouples object creation from usage.", "patterns"),
    ("Entity Framework Core provides LINQ-based database queries.", "orm"),
    ("ASP.NET Core supports both MVC and Minimal API patterns.", "web"),
    ("xUnit uses Facts and Theories for test methods.", "testing"),
    ("Docker containers ensure consistent deployment environments.", "devops"),
};

foreach (var (text, category) in documents)
{
    var emb = await embedder.GenerateEmbeddingAsync(text);
    var vector = new Vector(emb.Vector.ToArray());
    
    await using var cmd = conn.CreateCommand();
    cmd.CommandText = """
        INSERT INTO document_chunks (text, category, embedding)
        VALUES ($1, $2, $3)
        """;
    cmd.Parameters.AddWithValue(text);
    cmd.Parameters.AddWithValue(category);
    cmd.Parameters.AddWithValue(vector);
    await cmd.ExecuteNonQueryAsync();
    
    Console.WriteLine($"  ✅ Inserted: {text[..40]}...");
}

// =====================================================
// Semantic search using pgvector
// =====================================================
Console.WriteLine("\n=== Semantic Search with pgvector ===\n");

var query = "How do I write unit tests in .NET?";
var queryEmb = await embedder.GenerateEmbeddingAsync(query);
var queryVector = new Vector(queryEmb.Vector.ToArray());

await using (var cmd = conn.CreateCommand())
{
    // Use <=> operator for cosine distance (lower = more similar)
    cmd.CommandText = """
        SELECT text, category, 
               1 - (embedding <=> $1) as similarity
        FROM document_chunks
        ORDER BY embedding <=> $1
        LIMIT 3
        """;
    cmd.Parameters.AddWithValue(queryVector);
    
    await using var reader = await cmd.ExecuteReaderAsync();
    
    Console.WriteLine($"Query: \"{query}\"\n");
    while (await reader.ReadAsync())
    {
        Console.WriteLine($"  Similarity: {reader.GetFloat(2):F4}");
        Console.WriteLine($"  Text: {reader.GetString(0)}");
        Console.WriteLine($"  Category: {reader.GetString(1)}\n");
    }
}

Console.WriteLine("✅ pgvector search complete!");
```

---

## 🔑 pgvector Operators

| Operator | Metric | Usage |
|----------|--------|-------|
| `<=>` | Cosine distance | `ORDER BY embedding <=> $1` |
| `<->` | Euclidean (L2) distance | `ORDER BY embedding <-> $1` |
| `<#>` | Inner product (negative) | `ORDER BY embedding <#> $1` |

---

## ➡️ Next

Continue to **[Day 4: Hybrid Search](../Day-04-Hybrid-Search/README.md)**
