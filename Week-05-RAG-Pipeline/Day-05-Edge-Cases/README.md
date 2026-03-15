# Day 5: Handling Edge Cases

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Handle "no relevant results" gracefully
- Detect and prevent prompt injection attacks
- Implement answer validation and fallback logic
- Add logging and telemetry for RAG pipelines

---

## 💻 Edge Case Handling

### 1. No Relevant Results

```csharp
public async Task<RagResponse> AskWithFallbackAsync(string question)
{
    var result = await _retrieval.RetrieveAsync(question, minimumScore: 0.7f);
    
    if (!result.HasRelevantResults)
    {
        // Strategy 1: Lower the threshold and try again
        result = await _retrieval.RetrieveAsync(question, minimumScore: 0.5f);
        
        if (!result.HasRelevantResults)
        {
            // Strategy 2: Gracefully decline
            return new RagResponse
            {
                Answer = "I don't have information about this topic in our documentation. " +
                         "Please contact support or rephrase your question.",
                Grounded = false,
                FallbackUsed = true
            };
        }
        
        // Strategy 3: Answer with lower confidence warning
        var answer = await GenerateAnswer(result.Context, question);
        return new RagResponse
        {
            Answer = $"⚠️ Low confidence answer (limited relevant docs found):\n\n{answer}",
            Grounded = true,
            Confidence = "Low"
        };
    }
    
    var highConfAnswer = await GenerateAnswer(result.Context, question);
    return new RagResponse { Answer = highConfAnswer, Grounded = true, Confidence = "High" };
}
```

### 2. Prompt Injection Prevention

```csharp
public static class PromptGuard
{
    private static readonly string[] DangerousPatterns =
    [
        "ignore previous instructions",
        "ignore all previous",
        "disregard your instructions",
        "forget your rules",
        "you are now",
        "new system prompt",
        "override system",
    ];
    
    /// <summary>
    /// Checks if user input contains prompt injection attempts.
    /// </summary>
    public static bool IsSafe(string userInput)
    {
        var lower = userInput.ToLowerInvariant();
        return !DangerousPatterns.Any(pattern => lower.Contains(pattern));
    }
    
    /// <summary>
    /// Sanitizes retrieved context to prevent injection from documents.
    /// </summary>
    public static string SanitizeContext(string context)
    {
        // Wrap in XML tags that the model can distinguish from instructions
        return $"<retrieved_document>\n{context}\n</retrieved_document>";
    }
}
```

### 3. Answer Quality Validation

```csharp
public class AnswerValidator
{
    private readonly IChatClient _chatClient;
    
    public async Task<bool> ValidateGroundednessAsync(string answer, string context)
    {
        var validationPrompt = $"""
            You are a fact-checker. Determine if the ANSWER below is fully 
            supported by the CONTEXT. Return only "YES" or "NO".
            
            CONTEXT: {context}
            ANSWER: {answer}
            
            Is the answer fully supported by the context? (YES/NO):
            """;
        
        var result = await _chatClient.CompleteAsync(validationPrompt);
        return result.Message.Text?.Trim().ToUpper() == "YES";
    }
}
```

### 4. Logging & Telemetry

```csharp
public class InstrumentedRagService
{
    private readonly RagService _inner;
    private readonly ILogger<InstrumentedRagService> _logger;
    
    public async Task<RagResponse> AskAsync(string question)
    {
        var stopwatch = System.Diagnostics.Stopwatch.StartNew();
        
        _logger.LogInformation("RAG query: {Question}", question);
        
        var response = await _inner.AskAsync(question);
        
        stopwatch.Stop();
        
        _logger.LogInformation(
            "RAG completed in {ElapsedMs}ms. Grounded: {Grounded}, Sources: {SourceCount}",
            stopwatch.ElapsedMilliseconds,
            response.Grounded,
            response.Sources.Count);
        
        return response;
    }
}
```

---

## 🎉 Week 5 Complete!

You now have a production-quality RAG pipeline:
- ✅ Complete architecture understanding
- ✅ Retrieval with relevance filtering
- ✅ Augmentation with anti-hallucination prompts
- ✅ End-to-end implementation
- ✅ Edge case handling and security

**Next:** [Week 6: Autonomous AI Agents](../../Week-06-Autonomous-AI-Agents/README.md)
