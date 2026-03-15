# Day 2: Semantic Functions (Prompt Templates)

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Create prompt templates as reusable functions
- Use Handlebars and SK syntax for templates
- Load templates from files (not hardcoded strings)
- Pass variables into prompts dynamically

---

## 📖 What is a Semantic Function?

A Semantic Function is a **prompt template wrapped as a callable function**. Instead of hardcoding prompts in your C# code, you write them as templates that accept variables.

```
Traditional approach:
C# code → string concatenation → API call

Semantic Function approach:
Template file → Kernel loads it → Variables injected → API call
```

---

## 💻 Inline Semantic Functions

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.SemanticKernel;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();
var apiKey = config["OpenAI:ApiKey"]!;

var kernel = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion("gpt-4o-mini", apiKey)
    .Build();

// =====================================================
// Method 1: Inline prompt template
// =====================================================
var summarizeFunction = kernel.CreateFunctionFromPrompt("""
    Summarize the following text in {{$bulletCount}} bullet points.
    Each bullet should be under 15 words.
    
    TEXT:
    {{$input}}
    
    SUMMARY:
    """);

var result = await kernel.InvokeAsync(summarizeFunction, new KernelArguments
{
    ["input"] = """
        Microsoft Semantic Kernel is an open-source SDK that lets you easily build
        AI agents and integrate AI models into your C# applications. It provides
        plugin architecture, prompt templates, and built-in planners that enable
        AI to automatically orchestrate multiple steps to complete complex tasks.
        """,
    ["bulletCount"] = "3"
});

Console.WriteLine("=== Inline Template ===");
Console.WriteLine(result);

// =====================================================
// Method 2: From prompt template config (YAML)
// =====================================================
var translateFunction = kernel.CreateFunctionFromPrompt(new PromptTemplateConfig
{
    Name = "Translate",
    Description = "Translates text to a target language",
    Template = """
        Translate the following text to {{$targetLanguage}}.
        Preserve the formatting and technical terms.
        
        Text: {{$input}}
        
        Translation:
        """,
    InputVariables = [
        new() { Name = "input", Description = "Text to translate" },
        new() { Name = "targetLanguage", Description = "Target language", Default = "Spanish" }
    ],
    ExecutionSettings = new Dictionary<string, PromptExecutionSettings>
    {
        ["default"] = new() { ExtensionData = new Dictionary<string, object> { ["temperature"] = 0.3 } }
    }
});

var translation = await kernel.InvokeAsync(translateFunction, new KernelArguments
{
    ["input"] = "Dependency Injection is a design pattern used to achieve Inversion of Control.",
    ["targetLanguage"] = "French"
});

Console.WriteLine("\n=== Translated ===");
Console.WriteLine(translation);
```

---

## 📁 File-Based Prompt Templates

Create a folder structure for your prompts:

```
Plugins/
└── WritingPlugin/
    ├── Summarize/
    │   ├── skprompt.txt      ← The prompt template
    │   └── config.json       ← Settings (temperature, etc.)
    └── Translate/
        ├── skprompt.txt
        └── config.json
```

### `Plugins/WritingPlugin/Summarize/skprompt.txt`
```
Summarize the following text in {{$style}} style.
Keep it under {{$maxWords}} words.

Text to summarize:
{{$input}}

Summary:
```

### `Plugins/WritingPlugin/Summarize/config.json`
```json
{
  "schema": 1,
  "description": "Summarizes text in a given style",
  "execution_settings": {
    "default": {
      "temperature": 0.5,
      "max_tokens": 300
    }
  },
  "input_variables": [
    { "name": "input", "description": "The text to summarize" },
    { "name": "style", "description": "Style: formal, casual, technical", "default": "formal" },
    { "name": "maxWords", "description": "Maximum word count", "default": "100" }
  ]
}
```

### Loading from files:
```csharp
// Load all plugins from the Plugins directory
var plugins = kernel.ImportPluginFromPromptDirectory("./Plugins/WritingPlugin");

// Invoke a specific function from the loaded plugin
var summary = await kernel.InvokeAsync(
    plugins["Summarize"],
    new KernelArguments
    {
        ["input"] = longText,
        ["style"] = "technical",
        ["maxWords"] = "50"
    });
```

---

## 🔑 Key Takeaways

- Semantic Functions = **reusable prompt templates**
- Use `{{$variable}}` syntax for template variables
- Store prompts in files for better separation of concerns
- `config.json` sets default temperature, max tokens, and variable descriptions
- File-based prompts are **hot-reloadable** — change the prompt without recompiling!

---

## ➡️ Next

Continue to **[Day 3: Native C# Plugins](../Day-03-Native-CSharp-Plugins/README.md)**
