# Day 2: Document DB & Vector Search (MongoDB Atlas)

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Set up MongoDB Atlas with vector search capability
- Store embeddings alongside document metadata
- Use `$vectorSearch` aggregation pipeline
- Build a C# semantic search service

---

## 📖 MongoDB Atlas Vector Search

MongoDB Atlas has **native vector search** built in. You store embeddings as regular `double[]` fields, create a vector search index, and query using `$vectorSearch`.

```
┌──────────────────────────────────────┐
│ MongoDB Document                      │
│                                       │
│ {                                     │
│   "_id": "chunk-001",                │
│   "text": "DI is a pattern...",      │
│   "source": "dotnet-guide.pdf",      │
│   "page": 5,                         │
│   "embedding": [0.12, -0.34, ...]    │  ◄── 1536 floats
│ }                                     │
└──────────────────────────────────────┘
```

---

## 💻 Setup

### 1. Create MongoDB Atlas Cluster (Free Tier)
1. Go to [cloud.mongodb.com](https://cloud.mongodb.com)
2. Create a free M0 cluster
3. Create a database user
4. Get connection string

### 2. Create Vector Search Index
In Atlas UI → Search Indexes → Create Search Index:

```json
{
  "fields": [
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 1536,
      "similarity": "cosine"
    }
  ]
}
```

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;
using MongoDB.Driver;
using OpenAI;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

// MongoDB connection
var mongoClient = new MongoClient(config["MongoDB:ConnectionString"]);
var database = mongoClient.GetDatabase("ai_engineer");
var collection = database.GetCollection<DocumentChunk>("chunks");

// Embedding generator
IEmbeddingGenerator<string, Embedding<float>> embedder =
    new OpenAIClient(config["OpenAI:ApiKey"]!).AsEmbeddingGenerator("text-embedding-3-small");

// =====================================================
// Store documents with embeddings
// =====================================================
var documents = new[]
{
    ("Dependency Injection is a design pattern used throughout .NET.", "patterns"),
    ("Entity Framework Core maps C# objects to database tables.", "orm"),
    ("ASP.NET Core Minimal APIs provide a simplified approach to building web APIs.", "web"),
    ("xUnit is the recommended testing framework for .NET projects.", "testing"),
    ("Azure Functions enable serverless computing for event-driven applications.", "cloud"),
};

Console.WriteLine("Indexing documents...\n");

foreach (var (text, category) in documents)
{
    var embedding = await embedder.GenerateEmbeddingAsync(text);
    
    var chunk = new DocumentChunk
    {
        Text = text,
        Category = category,
        Embedding = embedding.Vector.ToArray().Select(f => (double)f).ToArray(),
        CreatedAt = DateTime.UtcNow
    };
    
    await collection.InsertOneAsync(chunk);
    Console.WriteLine($"  ✅ Indexed: {text[..50]}...");
}

// =====================================================
// Semantic search using $vectorSearch
// =====================================================
Console.WriteLine("\n=== Semantic Search ===\n");

var query = "How do I query a database in C#?";
var queryEmbedding = await embedder.GenerateEmbeddingAsync(query);

var pipeline = new BsonDocument[]
{
    new("$vectorSearch", new BsonDocument
    {
        { "index", "vector_index" },
        { "path", "embedding" },
        { "queryVector", new BsonArray(queryEmbedding.Vector.ToArray().Select(f => (double)f)) },
        { "numCandidates", 50 },
        { "limit", 3 }
    }),
    new("$project", new BsonDocument
    {
        { "text", 1 },
        { "category", 1 },
        { "score", new BsonDocument("$meta", "vectorSearchScore") }
    })
};

var results = await collection.Aggregate<BsonDocument>(pipeline).ToListAsync();

Console.WriteLine($"Query: \"{query}\"\n");
foreach (var result in results)
{
    Console.WriteLine($"  Score: {result["score"]:F4}");
    Console.WriteLine($"  Text: {result["text"]}");
    Console.WriteLine($"  Category: {result["category"]}\n");
}

// =====================================================
// Model
// =====================================================
public class DocumentChunk
{
    [BsonId]
    [BsonRepresentation(BsonType.ObjectId)]
    public string? Id { get; set; }
    
    [BsonElement("text")]
    public string Text { get; set; } = "";
    
    [BsonElement("category")]
    public string Category { get; set; } = "";
    
    [BsonElement("embedding")]
    public double[] Embedding { get; set; } = [];
    
    [BsonElement("createdAt")]
    public DateTime CreatedAt { get; set; }
}
```

---

## ➡️ Next

Continue to **[Day 3: Relational DB & Vectors](../Day-03-Relational-DB-Vectors/README.md)**
