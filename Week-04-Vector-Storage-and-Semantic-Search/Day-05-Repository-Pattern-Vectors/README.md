# Day 5: C# Repository Pattern for Vectors

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Abstract vector storage behind a clean C# interface
- Implement the Repository Pattern for vector operations
- Enable easy provider swapping (MongoDB ↔ PostgreSQL ↔ Qdrant)
- Follow SOLID principles in your AI data layer

---

## 💻 Code Sample

### Interface Definition

```csharp
/// <summary>
/// Abstracts vector storage operations.
/// Implementations can use MongoDB, PostgreSQL, Qdrant, etc.
/// </summary>
public interface IVectorRepository
{
    /// <summary>Stores a text chunk with its embedding</summary>
    Task<string> UpsertAsync(DocumentChunk chunk);
    
    /// <summary>Finds the most similar chunks to a query vector</summary>
    Task<IReadOnlyList<SearchResult>> SearchAsync(
        float[] queryVector,
        int limit = 5,
        string? categoryFilter = null);
    
    /// <summary>Removes a chunk by ID</summary>
    Task DeleteAsync(string id);
    
    /// <summary>Gets a chunk by ID</summary>
    Task<DocumentChunk?> GetByIdAsync(string id);
}

public record DocumentChunk
{
    public string Id { get; init; } = Guid.NewGuid().ToString();
    public string Text { get; init; } = "";
    public string Source { get; init; } = "";
    public string Category { get; init; } = "";
    public float[] Embedding { get; init; } = [];
    public Dictionary<string, string> Metadata { get; init; } = new();
    public DateTime CreatedAt { get; init; } = DateTime.UtcNow;
}

public record SearchResult
{
    public DocumentChunk Chunk { get; init; } = new();
    public float Score { get; init; }
}
```

### PostgreSQL Implementation

```csharp
using Npgsql;
using Pgvector;
using Pgvector.Npgsql;

public class PostgresVectorRepository : IVectorRepository
{
    private readonly NpgsqlDataSource _dataSource;
    
    public PostgresVectorRepository(string connectionString)
    {
        var builder = new NpgsqlDataSourceBuilder(connectionString);
        builder.UseVector();
        _dataSource = builder.Build();
    }
    
    public async Task<string> UpsertAsync(DocumentChunk chunk)
    {
        await using var conn = await _dataSource.OpenConnectionAsync();
        await using var cmd = conn.CreateCommand();
        cmd.CommandText = """
            INSERT INTO document_chunks (id, text, source, category, embedding, created_at)
            VALUES ($1, $2, $3, $4, $5, $6)
            ON CONFLICT (id) DO UPDATE SET
                text = EXCLUDED.text,
                embedding = EXCLUDED.embedding
            RETURNING id
            """;
        cmd.Parameters.AddWithValue(chunk.Id);
        cmd.Parameters.AddWithValue(chunk.Text);
        cmd.Parameters.AddWithValue(chunk.Source);
        cmd.Parameters.AddWithValue(chunk.Category);
        cmd.Parameters.AddWithValue(new Vector(chunk.Embedding));
        cmd.Parameters.AddWithValue(chunk.CreatedAt);
        
        return (await cmd.ExecuteScalarAsync())?.ToString() ?? chunk.Id;
    }
    
    public async Task<IReadOnlyList<SearchResult>> SearchAsync(
        float[] queryVector, int limit = 5, string? categoryFilter = null)
    {
        await using var conn = await _dataSource.OpenConnectionAsync();
        await using var cmd = conn.CreateCommand();
        cmd.CommandText = """
            SELECT id, text, source, category, 
                   1 - (embedding <=> $1) as score
            FROM document_chunks
            WHERE ($2::varchar IS NULL OR category = $2)
            ORDER BY embedding <=> $1
            LIMIT $3
            """;
        cmd.Parameters.AddWithValue(new Vector(queryVector));
        cmd.Parameters.AddWithValue(categoryFilter ?? (object)DBNull.Value);
        cmd.Parameters.AddWithValue(limit);
        
        var results = new List<SearchResult>();
        await using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            results.Add(new SearchResult
            {
                Chunk = new DocumentChunk
                {
                    Id = reader.GetString(0),
                    Text = reader.GetString(1),
                    Source = reader.GetString(2),
                    Category = reader.GetString(3)
                },
                Score = reader.GetFloat(4)
            });
        }
        return results;
    }
    
    public async Task DeleteAsync(string id)
    {
        await using var conn = await _dataSource.OpenConnectionAsync();
        await using var cmd = conn.CreateCommand();
        cmd.CommandText = "DELETE FROM document_chunks WHERE id = $1";
        cmd.Parameters.AddWithValue(id);
        await cmd.ExecuteNonQueryAsync();
    }
    
    public async Task<DocumentChunk?> GetByIdAsync(string id)
    {
        await using var conn = await _dataSource.OpenConnectionAsync();
        await using var cmd = conn.CreateCommand();
        cmd.CommandText = "SELECT id, text, source, category FROM document_chunks WHERE id = $1";
        cmd.Parameters.AddWithValue(id);
        
        await using var reader = await cmd.ExecuteReaderAsync();
        if (await reader.ReadAsync())
        {
            return new DocumentChunk
            {
                Id = reader.GetString(0),
                Text = reader.GetString(1),
                Source = reader.GetString(2),
                Category = reader.GetString(3)
            };
        }
        return null;
    }
}
```

### DI Registration

```csharp
// In your DI setup:
services.AddSingleton<IVectorRepository>(sp =>
    new PostgresVectorRepository(config["PostgreSQL:ConnectionString"]!));

// Swap to MongoDB by changing this one line:
// services.AddSingleton<IVectorRepository>(sp =>
//     new MongoVectorRepository(config["MongoDB:ConnectionString"]!));
```

---

## 🎉 Week 4 Complete!

You now understand:
- ✅ Vector database fundamentals and ANN algorithms
- ✅ MongoDB Atlas vector search
- ✅ PostgreSQL with pgvector
- ✅ Hybrid search (keyword + semantic)
- ✅ Clean Repository Pattern for vectors

**Next:** [Week 5: Building the RAG Pipeline](../../Week-05-RAG-Pipeline/README.md)
