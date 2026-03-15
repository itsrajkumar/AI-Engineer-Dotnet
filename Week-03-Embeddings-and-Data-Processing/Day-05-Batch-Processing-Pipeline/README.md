# Day 5: Batch Processing Pipeline

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Build a document ingestion pipeline in C#
- Read files from a directory, chunk them, and generate embeddings
- Handle rate limiting and errors gracefully
- Output processed data ready for vector storage (Week 4)

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;
using System.Text.Json;

// =====================================================
// Day 5: Batch Document Processing Pipeline
// =====================================================

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

IEmbeddingGenerator<string, Embedding<float>> embedder =
    new OpenAIClient(config["OpenAI:ApiKey"]!).AsEmbeddingGenerator("text-embedding-3-small");

// =====================================================
// Step 1: Read all text files from a directory
// =====================================================
var inputDir = args.Length > 0 ? args[0] : "./sample-docs";
Directory.CreateDirectory(inputDir);

// Create sample files if none exist
if (!Directory.GetFiles(inputDir, "*.txt").Any())
{
    await File.WriteAllTextAsync(Path.Combine(inputDir, "dotnet-overview.txt"),
        """
        .NET is a free, open-source development platform for building many kinds of apps.
        With .NET, you can use multiple languages, editors, and libraries to build for web,
        mobile, desktop, games, IoT, and more. .NET is supported on Windows, Linux, and macOS.
        
        .NET 8 is the latest LTS release. It brings performance improvements, new APIs,
        and enhanced support for cloud-native applications. The runtime is optimized for
        modern workloads including AI and machine learning scenarios.
        """);
    
    await File.WriteAllTextAsync(Path.Combine(inputDir, "ef-core-guide.txt"),
        """
        Entity Framework Core is a modern object-relational mapper for .NET.
        It supports LINQ queries, change tracking, updates, and schema migrations.
        EF Core works with SQL Server, SQLite, PostgreSQL, MySQL, and Cosmos DB.
        
        To get started, install the Microsoft.EntityFrameworkCore NuGet package.
        Define your model classes and a DbContext, then use migrations to create
        your database schema. EF Core supports both code-first and database-first.
        """);
    
    Console.WriteLine($"Created sample files in {inputDir}");
}

var files = Directory.GetFiles(inputDir, "*.txt");
Console.WriteLine($"Found {files.Length} files to process.\n");

// =====================================================
// Step 2: Chunk each file
// =====================================================
var allChunks = new List<ProcessedChunk>();

foreach (var file in files)
{
    var content = await File.ReadAllTextAsync(file);
    var fileName = Path.GetFileName(file);
    
    // Simple paragraph-based chunking
    var paragraphs = content.Split(
        new[] { "\n\n", "\r\n\r\n" },
        StringSplitOptions.RemoveEmptyEntries);
    
    for (int i = 0; i < paragraphs.Length; i++)
    {
        var text = paragraphs[i].Trim();
        if (string.IsNullOrWhiteSpace(text)) continue;
        
        allChunks.Add(new ProcessedChunk
        {
            Id = $"{fileName}:chunk-{i}",
            SourceFile = fileName,
            ChunkIndex = i,
            Text = text
        });
    }
    
    Console.WriteLine($"  📄 {fileName}: {paragraphs.Length} chunks");
}

Console.WriteLine($"\nTotal chunks: {allChunks.Count}");

// =====================================================
// Step 3: Generate embeddings in batches
// =====================================================
Console.WriteLine("\nGenerating embeddings...");

const int batchSize = 10;
var processed = 0;

for (int i = 0; i < allChunks.Count; i += batchSize)
{
    var batch = allChunks.Skip(i).Take(batchSize).ToList();
    var texts = batch.Select(c => c.Text).ToList();
    
    try
    {
        var embeddings = await embedder.GenerateAsync(texts);
        
        for (int j = 0; j < batch.Count; j++)
        {
            batch[j].Embedding = embeddings[j].Vector.ToArray();
        }
        
        processed += batch.Count;
        Console.WriteLine($"  ✅ Processed {processed}/{allChunks.Count} chunks");
    }
    catch (HttpRequestException ex) when (ex.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
    {
        Console.WriteLine("  ⏳ Rate limited, waiting 30 seconds...");
        await Task.Delay(TimeSpan.FromSeconds(30));
        i -= batchSize; // Retry this batch
    }
}

// =====================================================
// Step 4: Save processed data as JSON
// =====================================================
var outputFile = "./processed-documents.json";
var json = JsonSerializer.Serialize(allChunks, new JsonSerializerOptions 
{ 
    WriteIndented = true 
});
await File.WriteAllTextAsync(outputFile, json);

Console.WriteLine($"\n💾 Saved {allChunks.Count} processed chunks to {outputFile}");
Console.WriteLine("   Ready for vector database ingestion (Week 4)!");

// =====================================================
// Model
// =====================================================
public class ProcessedChunk
{
    public string Id { get; set; } = "";
    public string SourceFile { get; set; } = "";
    public int ChunkIndex { get; set; }
    public string Text { get; set; } = "";
    public float[]? Embedding { get; set; }
}
```

---

## 🔑 Pipeline Architecture

```
┌──────────┐    ┌──────────┐    ┌───────────┐    ┌──────────┐
│  Read    │    │  Chunk   │    │  Embed    │    │  Store   │
│  Files   │──►│  Text    │──►│  Vectors  │──►│  JSON    │
│          │    │          │    │ (batched) │    │  File    │
└──────────┘    └──────────┘    └───────────┘    └──────────┘
    Day 5           Day 3          Day 2          Week 4
```

---

## 🎉 Week 3 Complete!

You now understand:
- ✅ Embedding theory and vector spaces
- ✅ Generating embeddings with `IEmbeddingGenerator`
- ✅ Document chunking strategies
- ✅ Cosine similarity for semantic search
- ✅ Batch processing pipelines

**Next:** [Week 4: Vector Storage & Semantic Search](../../Week-04-Vector-Storage-and-Semantic-Search/README.md)
