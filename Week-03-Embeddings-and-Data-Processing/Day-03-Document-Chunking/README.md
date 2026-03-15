# Day 3: Document Chunking Strategies

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Understand why chunking is necessary for RAG
- Implement multiple chunking strategies in C#
- Handle PDFs, text files, and markdown documents
- Preserve context across chunk boundaries

---

## 📖 Why Chunking?

LLMs have context window limits. You can't embed an entire 500-page document as one vector. Instead, you split it into "chunks" and embed each chunk separately.

```
Large Document (100 pages)
├── Chunk 1: "Chapter 1: Introduction to .NET..."     → float[1536]
├── Chunk 2: "The framework provides dependency..."    → float[1536]
├── Chunk 3: "Entity Framework Core supports..."       → float[1536]
├── ...
└── Chunk 250: "In conclusion, .NET 8 represents..."   → float[1536]
```

---

## ⚠️ The Chunking Trade-off

| Too Small (50 tokens) | Too Large (2000 tokens) |
|----------------------|------------------------|
| Loses context | Hard to search precisely |
| "He said" — who? | Multiple topics in one chunk |
| High noise in search | Uses more context window |
| **More chunks = higher cost** | **Fewer chunks = less precise** |

**Sweet spot: 200-500 tokens per chunk with 10-20% overlap**

---

## 💻 Code Samples

### Strategy 1: Fixed-Size Chunking

```csharp
/// <summary>
/// Splits text into chunks of approximately the specified token size.
/// Includes overlap to preserve context across boundaries.
/// </summary>
public static class TextChunker
{
    public static List<string> ChunkByTokenCount(
        string text,
        int maxTokensPerChunk = 300,
        int overlapTokens = 50)
    {
        var words = text.Split(' ', StringSplitOptions.RemoveEmptyEntries);
        var chunks = new List<string>();
        
        // Rough approximation: 1 token ≈ 0.75 words
        int wordsPerChunk = (int)(maxTokensPerChunk * 0.75);
        int overlapWords = (int)(overlapTokens * 0.75);
        
        int start = 0;
        while (start < words.Length)
        {
            int end = Math.Min(start + wordsPerChunk, words.Length);
            var chunk = string.Join(" ", words[start..end]);
            chunks.Add(chunk);
            
            start = end - overlapWords; // Overlap with previous chunk
            if (start >= words.Length) break;
        }
        
        return chunks;
    }
}
```

### Strategy 2: Paragraph-Based Chunking

```csharp
public static List<string> ChunkByParagraph(
    string text,
    int maxTokensPerChunk = 500)
{
    var paragraphs = text.Split(
        new[] { "\n\n", "\r\n\r\n" },
        StringSplitOptions.RemoveEmptyEntries);
    
    var chunks = new List<string>();
    var currentChunk = new StringBuilder();
    int currentTokenEstimate = 0;
    
    foreach (var paragraph in paragraphs)
    {
        int paragraphTokens = EstimateTokens(paragraph);
        
        if (currentTokenEstimate + paragraphTokens > maxTokensPerChunk 
            && currentChunk.Length > 0)
        {
            chunks.Add(currentChunk.ToString().Trim());
            currentChunk.Clear();
            currentTokenEstimate = 0;
        }
        
        currentChunk.AppendLine(paragraph);
        currentTokenEstimate += paragraphTokens;
    }
    
    if (currentChunk.Length > 0)
        chunks.Add(currentChunk.ToString().Trim());
    
    return chunks;
}

private static int EstimateTokens(string text) => (int)(text.Split(' ').Length / 0.75);
```

### Strategy 3: Semantic Chunking (Markdown-Aware)

```csharp
public static List<DocumentChunk> ChunkMarkdown(string markdown)
{
    var chunks = new List<DocumentChunk>();
    var sections = markdown.Split(new[] { "\n## ", "\n# " }, StringSplitOptions.RemoveEmptyEntries);
    
    foreach (var section in sections)
    {
        var lines = section.Split('\n');
        var heading = lines[0].TrimStart('#', ' ');
        var content = string.Join("\n", lines.Skip(1)).Trim();
        
        if (!string.IsNullOrWhiteSpace(content))
        {
            chunks.Add(new DocumentChunk
            {
                Heading = heading,
                Content = content,
                TokenEstimate = EstimateTokens(content)
            });
        }
    }
    
    return chunks;
}

public record DocumentChunk
{
    public string Heading { get; init; } = "";
    public string Content { get; init; } = "";
    public int TokenEstimate { get; init; }
    public float[]? Embedding { get; set; }
}
```

---

## 📖 Chunking Decision Guide

```
What type of document?
├── Structured (Markdown, HTML) → Semantic Chunking (by headings/sections)
├── Semi-structured (PDF, DOCX) → Paragraph-based chunking
├── Unstructured (plain text) → Fixed-size with overlap
└── Code files → Function/class-based chunking
```

---

## ➡️ Next

Continue to **[Day 4: Cosine Similarity](../Day-04-Cosine-Similarity/README.md)**
