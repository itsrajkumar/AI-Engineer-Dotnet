# Day 5: Synthetic Data Generation

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** SynthDataGen
>
> 🆕 *New for v4.0 (2026)*

---

## 🎯 Learning Objectives

- Understand why Synthetic Data is the 2026 standard for fine-tuning.
- Build a synthetic data generation pipeline using MAF and MEAI.
- Generate high-quality JSONL datasets for fine-tuning.
- Prevent mode collapse in synthetic data.

---

## 📖 The Synthetic Data Revolution

As we learned on Day 2, SFT requires 1,000+ extremely high-quality examples. Writing these by hand takes weeks of expensive human labor.

In 2026, the industry standard is **Synthetic Data Generation (SDG)**.
We use a frontier model (like GPT-5 or Claude 5) as a "Teacher" to generate training data for a smaller "Student" model (like an 8B parameter model). This allows us to distill the reasoning capabilities of a massive, expensive API into a small, fast model we can run locally for free.

---

## 💻 Code Sample: Synthetic Data Generator with MAF

We can use the Microsoft Agent Framework to build an automated data generation pipeline in .NET.

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Agents.Framework;
using System.Text.Json;

// =====================================================
// 1. Setup the Teacher Model (e.g., GPT-5)
// =====================================================
var teacherClient = new OpenAIClient(apiKey).AsChatClient("gpt-5");

// We ask the teacher to output structured data
var options = new ChatOptions 
{ 
    Temperature = 0.8f, // High temperature for diverse data
    ResponseFormat = ChatResponseFormat.Json
};

// =====================================================
// 2. The Generation Loop
// =====================================================
var topics = new[] { "Azure deployment", "C# Garbage Collection", "Dependency Injection" };
var dataset = new List<DatasetRow>();

foreach (var topic in topics)
{
    Console.WriteLine($"Generating synthetic data for: {topic}");

    var prompt = $"""
        You are an expert curriculum designer.
        Generate a highly complex, advanced user question about '{topic}'.
        Then, generate a perfect, highly detailed, step-by-step response.
        
        Output JSON in this format:
        {{ "UserPrompt": "...", "AssistantResponse": "..." }}
        """;

    var response = await teacherClient.GetResponseAsync<SyntheticExample>(prompt, options);
    
    if (response.Result != null)
    {
        dataset.Add(new DatasetRow(
            Prompt: response.Result.UserPrompt,
            Response: response.Result.AssistantResponse
        ));
    }
}

// =====================================================
// 3. Export to JSONL format for Fine-tuning
// =====================================================
using var stream = File.CreateText("synthetic_dataset.jsonl");
foreach (var row in dataset)
{
    // Convert to standard ChatML / OpenAI fine-tuning format
    var jsonlRow = new
    {
        messages = new[]
        {
            new { role = "system", content = "You are a .NET expert." },
            new { role = "user", content = row.Prompt },
            new { role = "assistant", content = row.Response }
        }
    };
    
    await stream.WriteLineAsync(JsonSerializer.Serialize(jsonlRow));
}

// DTOs
record SyntheticExample(string UserPrompt, string AssistantResponse);
record DatasetRow(string Prompt, string Response);
```

### ⚠️ Warning: Mode Collapse

When generating synthetic data, you must be careful of **Mode Collapse**. If you just ask GPT-5 to "write 100 questions about C#", they will all look exactly the same (same tone, same structure, same vocabulary). A model fine-tuned on this data will become robotic and repetitive.

**Solution:** Use Persona Injection.
*"Generate this data acting as an angry senior developer."*
*"Generate this data acting as a confused junior intern."*
*"Generate this data acting as a project manager."*
Injecting diverse personas into the generation prompt forces the Teacher model to generate highly diverse training data.

---

## 🔑 Key Takeaways

- **Synthetic Data** allows you to build custom, specialized models in hours instead of months.
- Use a large, smart model (GPT-5) to generate training data for a smaller, fast model (Llama 4 8B).
- **JSONL (JSON Lines)** is the standard file format for fine-tuning datasets.
- Ensure diversity in your synthetic data to prevent the student model from collapsing into repetitive patterns.

---

## 🏆 Week 12 Complete!

You now understand how raw models are transformed into helpful assistants!

Continue to **[Week 13: Advanced RAG and Agent Swarms](../../Week-13-Advanced-RAG-and-Agents/README.md)** to explore the absolute cutting-edge of production AI architectures.
