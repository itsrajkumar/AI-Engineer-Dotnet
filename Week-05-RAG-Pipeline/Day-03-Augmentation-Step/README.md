# Day 3: The Augmentation Step

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Construct a strict RAG prompt template
- Inject retrieved context into the prompt
- Add guardrails to prevent hallucination
- Use Semantic Kernel for prompt construction

---

## 💻 Code Sample

```csharp
using Microsoft.SemanticKernel;

// =====================================================
// The RAG Prompt Template (The "Augmentation")
// =====================================================
// This is the most critical prompt in your entire RAG system.
// It tells the LLM: "ONLY use the provided context. Don't make stuff up."

var ragPromptTemplate = """
    <system>
    You are a helpful assistant that answers questions about our internal documentation.

    CRITICAL RULES:
    1. Answer the user's question using ONLY the context provided below.
    2. If the context does not contain enough information to answer, say:
       "I don't have enough information in our documentation to answer this question."
    3. NEVER make up information that is not in the context.
    4. Always cite which source you used by referencing [Source N].
    5. Keep answers concise but complete.
    6. If multiple sources are relevant, synthesize information from all of them.
    </system>

    CONTEXT FROM OUR DOCUMENTATION:
    ===
    {{$context}}
    ===

    USER QUESTION: {{$question}}

    Answer (cite sources):
    """;

// =====================================================
// Using with Semantic Kernel
// =====================================================
var kernel = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion("gpt-4o-mini", apiKey)
    .Build();

var ragFunction = kernel.CreateFunctionFromPrompt(ragPromptTemplate);

// Simulate retrieved context (from Day 2's retrieval step)
var retrievedContext = """
    [Source 1: dependency-injection.md]
    Dependency Injection (DI) is a software design pattern that implements
    Inversion of Control (IoC). In .NET, DI is built into the framework
    via Microsoft.Extensions.DependencyInjection. Services are registered
    in the DI container using AddSingleton, AddScoped, or AddTransient.
    
    [Source 2: aspnet-di.md]
    ASP.NET Core uses DI extensively. Controllers, middleware, and services
    all receive their dependencies through constructor injection. The built-in
    container supports constructor injection only; property injection requires
    third-party containers like Autofac.
    """;

var answer = await kernel.InvokeAsync(ragFunction, new KernelArguments
{
    ["context"] = retrievedContext,
    ["question"] = "How does Dependency Injection work in .NET?"
});

Console.WriteLine(answer);
// Expected output cites [Source 1] and [Source 2]
```

---

## 🔑 Prompt Template Best Practices

| Practice | Why |
|----------|-----|
| Use XML tags for sections | Clear separation of instructions vs. content |
| "ONLY use the context" | Prevents hallucination |
| "Say you don't know" | Graceful failure mode |
| "Cite sources" | Traceability and trust |
| Wrap context in delimiters | Prevents prompt injection from doc content |

---

## ➡️ Next

Continue to **[Day 4: End-to-End RAG](../Day-04-End-to-End-RAG/README.md)**
