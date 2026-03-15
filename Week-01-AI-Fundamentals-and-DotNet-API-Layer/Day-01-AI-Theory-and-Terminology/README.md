# Day 1: AI Theory & Terminology

> **Type:** 📖 Theory | **Time:** ~3 hours | **Code:** None (pure theory)

---

## 🎯 Learning Objectives

- Understand what Generative AI is and how it differs from traditional ML
- Know the difference between LLMs and SLMs
- Understand tokens, context windows, and temperature
- Map AI terminology to concepts you already know from .NET

---

## 📖 What is Artificial Intelligence?

AI is a broad field. Here is where our focus area sits:

```
Artificial Intelligence (AI)
├── Machine Learning (ML)
│   ├── Supervised Learning (Classification, Regression)
│   ├── Unsupervised Learning (Clustering, Dimensionality Reduction)
│   └── Deep Learning
│       ├── Convolutional Neural Networks (CNNs) — Images
│       ├── Recurrent Neural Networks (RNNs) — Sequences
│       └── Transformers ◄── THIS IS OUR FOCUS
│           ├── Encoder-only (BERT) — Understanding text
│           ├── Decoder-only (GPT) — Generating text
│           └── Encoder-Decoder (T5) — Translation, Summarization
└── Generative AI ◄── THIS IS WHERE WE BUILD
    ├── Large Language Models (LLMs) — GPT-4, Claude, Gemini
    ├── Small Language Models (SLMs) — Phi-3, Llama 3.1 8B
    └── Image Generation — DALL-E, Stable Diffusion
```

---

## 🧠 Generative AI vs. Traditional ML

| Aspect | Traditional ML | Generative AI |
|--------|---------------|---------------|
| **Training data** | Labeled datasets for specific tasks | Massive unlabeled text from the internet |
| **Output** | Predictions (numbers, classes) | New content (text, images, code) |
| **Flexibility** | One model = one task | One model = many tasks via prompting |
| **Customization** | Retrain the model | Change the prompt |
| **.NET Analogy** | Like a typed `Func<Input, Output>` | Like a dynamic `Object` that can do anything |

### Real-World Example
- **Traditional ML:** Train a model on 10,000 support tickets to classify them as "Billing" or "Technical."
- **Generative AI:** Give GPT-4 a support ticket and ask: *"Classify this ticket and draft a response."* No training needed.

---

## 🔤 Large Language Models (LLMs) vs. Small Language Models (SLMs)

### LLMs — The Heavy Hitters
- **Examples:** GPT-4o, Claude 3.5 Sonnet, Gemini 1.5 Pro
- **Parameters:** 100B+ parameters
- **Hosting:** Cloud-only (too large for local machines)
- **Strengths:** Best reasoning, code generation, multi-step tasks
- **Cost:** $ per API call

### SLMs — The Efficient Alternatives
- **Examples:** Phi-3 (Microsoft), Llama 3.1 8B (Meta), Mistral 7B
- **Parameters:** 1B-13B parameters
- **Hosting:** Can run locally on a good laptop/desktop
- **Strengths:** Fast, private, cheap; good for focused tasks
- **Cost:** Free (local inference)

### When to Use Which?

```
Decision Tree:
                    ┌─────────────────────┐
                    │ Do you need the      │
                    │ absolute best quality?│
                    └──────┬───────┬───────┘
                     Yes   │       │  No
                           ▼       ▼
                    ┌──────────┐  ┌──────────────────┐
                    │ Use LLM  │  │ Is data privacy   │
                    │ (GPT-4o) │  │ a concern?        │
                    └──────────┘  └──┬─────────┬─────┘
                                Yes │         │ No
                                    ▼         ▼
                             ┌──────────┐ ┌──────────┐
                             │ Use SLM  │ │ Use SLM  │
                             │ (Local)  │ │ via API  │
                             └──────────┘ │ (Cheaper)│
                                          └──────────┘
```

---

## 🔡 Tokens — The Currency of AI

### What is a Token?

A token is **not a character** and **not a word**. It's a sub-word unit that the model processes.

**Rules of thumb:**
- 1 token ≈ 4 characters in English
- 1 token ≈ ¾ of a word
- 100 tokens ≈ 75 words

### Tokenization Examples

```
Input: "Hello, world!"
Tokens: ["Hello", ",", " world", "!"]   → 4 tokens

Input: "C# is a great programming language"
Tokens: ["C", "#", " is", " a", " great", " programming", " language"]   → 7 tokens

Input: "Microsoft.Extensions.AI"
Tokens: ["Microsoft", ".", "Extensions", ".", "AI"]   → 5 tokens
```

### Why Tokens Matter for .NET Developers

```csharp
// You pay per token for API calls
// GPT-4o-mini pricing (as of 2024):
// Input:  $0.15 per 1M tokens
// Output: $0.60 per 1M tokens

// Example cost calculation:
var inputTokens = 500;   // Your prompt
var outputTokens = 200;  // Model's response
var costUsd = (inputTokens * 0.15 / 1_000_000) + (outputTokens * 0.60 / 1_000_000);
// Cost = $0.000195 — incredibly cheap!
```

### Token Counting in C#

```csharp
// Using Microsoft.ML.Tokenizers
using Microsoft.ML.Tokenizers;

var tokenizer = TiktokenTokenizer.CreateForModel("gpt-4o");
var tokens = tokenizer.CountTokens("Hello, how are you?");
Console.WriteLine($"Token count: {tokens}"); // Output: 6
```

---

## 📏 Context Window — The "Request Body Size Limit"

The context window is the **maximum number of tokens** the model can process in a single request (input + output combined).

| Model | Context Window | Approximate Pages |
|-------|---------------|-------------------|
| GPT-4o-mini | 128K tokens | ~200 pages |
| GPT-4o | 128K tokens | ~200 pages |
| Claude 3.5 Sonnet | 200K tokens | ~300 pages |
| Llama 3.1 8B | 128K tokens | ~200 pages |
| Phi-3 Mini | 4K-128K tokens | Varies |

### .NET Analogy
```csharp
// Think of it like a request size limit on your Web API:
builder.Services.Configure<KestrelServerOptions>(options =>
{
    options.Limits.MaxRequestBodySize = 128_000; // Like a 128K context window
});

// If your prompt + expected response exceeds this, the model will truncate!
```

### What Fills the Context Window?

```
[System Prompt]        ← Sets behavior (uses tokens!)
[Chat History]         ← Previous messages (grows over time!)
[Retrieved Context]    ← RAG content (Week 5)
[User's Current Query] ← The actual question
[Model's Response]     ← Generated output
─────────────────────
Total must be < Context Window size
```

> **⚠️ Key Insight:** As conversation history grows, you consume more and more of the context window. This is why **state management** (Week 1, Day 5) is critical.

---

## 🌡️ Temperature — The "Randomness Dial"

Temperature controls how "creative" or "deterministic" the model's responses are.

| Temperature | Behavior | Use Case |
|-------------|----------|----------|
| 0.0 | Deterministic — always picks the most likely next token | Code generation, data extraction, classifications |
| 0.3 | Slightly creative but still focused | Customer support, summarization |
| 0.7 | Good balance of creativity and coherence | General conversation, brainstorming |
| 1.0 | Maximum creativity, may hallucinate | Creative writing, idea generation |

### .NET Analogy
```csharp
// Temperature = 0 is like:
var result = items.OrderByDescending(x => x.Probability).First();

// Temperature = 1 is like:
var result = items.OrderBy(_ => Random.Shared.Next()).First();

// Temperature = 0.7 is somewhere in between:
// Higher probability items are still favored, but there's randomness
```

---

## 🔑 Other Important Parameters

### Top-P (Nucleus Sampling)
- Controls the cumulative probability threshold
- `top_p = 0.9` means consider only tokens whose cumulative probability is within the top 90%
- Usually don't change both temperature AND top_p

### Max Tokens
- Limits the response length
- Important for cost control and preventing runaway responses

### Frequency Penalty & Presence Penalty
- `frequency_penalty`: Penalizes tokens that appear frequently (reduces repetition)
- `presence_penalty`: Penalizes tokens that have appeared at all (encourages new topics)

---

## 📊 The Transformer Architecture (Simplified)

You don't need to understand the math, but knowing the architecture helps:

```
Input Text
    │
    ▼
┌──────────────┐
│  Tokenizer   │  Splits text into tokens
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Embeddings  │  Converts tokens to vectors (Week 3)
└──────┬───────┘
       │
       ▼
┌──────────────────────────────┐
│  Transformer Layers (×N)     │
│  ┌────────────────────────┐  │
│  │  Self-Attention        │  │  "Which other tokens are relevant to this one?"
│  └────────────────────────┘  │
│  ┌────────────────────────┐  │
│  │  Feed-Forward Network  │  │  "What should the next token be?"
│  └────────────────────────┘  │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────┐
│  Output Probabilities │  Ranked list of possible next tokens
└──────────────────────┘
```

### Key concept: **Self-Attention**
This is the breakthrough that made Transformers work. For each token, the model looks at ALL other tokens in the context to understand relationships.

Example: *"The **cat** sat on the mat because **it** was tired."*
- Self-attention helps the model understand that "it" refers to "cat", not "mat."

---

## 🗺️ The AI Application Stack

As an AI Engineer, you won't train models. You'll **build applications on top of them:**

```
┌─────────────────────────────────────┐
│      Your .NET AI Application       │  ◄── YOU BUILD THIS
├─────────────────────────────────────┤
│      Orchestration Layer            │  ◄── Semantic Kernel (Week 2)
│      (Prompt chaining, Plugins)     │
├─────────────────────────────────────┤
│      Data Layer                     │  ◄── Embeddings + Vector DBs (Weeks 3-4)
│      (RAG, Embeddings, Vectors)     │
├─────────────────────────────────────┤
│      Foundation Model (LLM/SLM)    │  ◄── GPT-4o, Llama, Phi (Pre-trained)
│      API Access Only               │
├─────────────────────────────────────┤
│      Infrastructure                 │  ◄── Azure, Local, Ollama
│      (Hosting, Scaling, Monitoring) │
└─────────────────────────────────────┘
```

---

## 📝 Self-Assessment Quiz

1. What is the difference between an LLM and an SLM?
2. If your prompt contains 500 tokens and the model responds with 300 tokens, how many total tokens were used?
3. When should you set temperature to 0?
4. What happens when conversation history exceeds the context window?
5. Why is an AI Engineer different from an ML Engineer?

<details>
<summary>📋 Answers</summary>

1. **LLM** (100B+ params, cloud-hosted, best quality, costs money) vs **SLM** (1-13B params, can run locally, faster/cheaper, good for focused tasks).
2. **800 tokens** total (input + output).
3. When you need **deterministic, consistent results** — code generation, data extraction, classification.
4. The model **truncates** older messages or errors out. You need a strategy to manage history (summarization, sliding window).
5. **ML Engineers** train models from data. **AI Engineers** build applications USING pre-trained models via APIs.

</details>

---

## 📚 References & Further Reading

- [What are Large Language Models?](https://learn.microsoft.com/azure/ai-services/openai/concepts/models) — Microsoft Learn
- [GPT-4 Technical Report](https://arxiv.org/abs/2303.08774) — OpenAI
- [Attention Is All You Need](https://arxiv.org/abs/1706.03762) — The original Transformer paper
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer) — Visualize tokens
- [AI for .NET Developers](https://learn.microsoft.com/dotnet/ai/) — Microsoft Learn

---

## ➡️ Next

Continue to **[Day 2: Prompt Engineering Basics](../Day-02-Prompt-Engineering-Basics/README.md)**
