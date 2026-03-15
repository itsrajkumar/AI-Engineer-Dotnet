# Day 2: Prompt Engineering Basics

> **Type:** 📖 Theory + Practice | **Time:** ~3 hours | **Code:** Practice in ChatGPT/Gemini UI

---

## 🎯 Learning Objectives

- Understand why prompt engineering is the #1 skill for AI Engineers
- Master Zero-shot, Few-shot, and Chain-of-Thought prompting
- Learn advanced patterns: Role prompting, Output formatting, Guardrails
- Practice structuring prompts that produce consistent, high-quality outputs

---

## 📖 Why Prompt Engineering Matters

> **"The prompt IS the program."**

In traditional software, you write imperative code: `if (x > 5) return "high"`. With LLMs, you write **natural language instructions** that achieve the same result. The quality of your output depends entirely on the quality of your input.

### .NET Analogy
```csharp
// Traditional approach: Write explicit code
public string ClassifyTicket(string text)
{
    if (text.Contains("invoice") || text.Contains("payment"))
        return "Billing";
    if (text.Contains("error") || text.Contains("bug"))
        return "Technical";
    return "General";
}

// AI approach: Write a prompt
var prompt = """
    Classify the following support ticket into one category:
    Billing, Technical, or General.
    
    Ticket: {{input}}
    
    Category:
    """;
// The LLM handles ALL the edge cases you'd never think of
```

---

## 🎯 Prompt Engineering Patterns

### 1. Zero-Shot Prompting

**No examples provided.** You rely entirely on the model's pre-trained knowledge.

```
Prompt: "Translate the following English text to French: 'Hello, how are you?'"

Response: "Bonjour, comment allez-vous ?"
```

**When to use:** Simple, well-defined tasks where the model clearly understands what you want.

**When it fails:** Ambiguous tasks, domain-specific formatting, or when you need a specific output structure.

---

### 2. Few-Shot Prompting

**Provide examples** of input → output to teach the model the pattern.

```
Prompt:
"Classify the sentiment of the following product reviews.

Review: 'This laptop is amazing, best purchase ever!'
Sentiment: Positive

Review: 'Terrible battery life, don't waste your money.'
Sentiment: Negative

Review: 'It's okay, nothing special but gets the job done.'
Sentiment: Neutral

Review: 'The screen resolution blew me away, truly gorgeous display.'
Sentiment: "

Response: "Positive"
```

**When to use:** 
- When you need a specific output format
- When the task is domain-specific
- When zero-shot gives inconsistent results

**Best practices:**
- Use 2-5 examples (more isn't always better)
- Cover edge cases in your examples
- Keep examples diverse but consistent

---

### 3. Chain-of-Thought (CoT) Prompting

**Force the model to "think step by step"** before giving a final answer. This dramatically improves accuracy on reasoning tasks.

```
BAD Prompt:
"A store has 23 shirts. They receive 2 shipments of 15 shirts each.
They sell 17 shirts. How many shirts remain?"

BAD Response: "36" (incorrect)

GOOD Prompt:
"A store has 23 shirts. They receive 2 shipments of 15 shirts each.
They sell 17 shirts. How many shirts remain?

Let's solve this step by step."

GOOD Response:
"Step 1: Start with 23 shirts.
Step 2: Receive 2 shipments of 15 = 30 additional shirts.
Step 3: Total = 23 + 30 = 53 shirts.
Step 4: Sell 17 shirts.
Step 5: 53 - 17 = 36 shirts remaining.

Answer: 36 shirts"
```

> Actually, both got 36 here. But for complex multi-step reasoning, CoT prevents errors by making the model show its work — like requiring your junior devs to explain their PR.

---

### 4. Role Prompting (Persona Pattern)

**Assign the AI a specific role** to change its knowledge focus and communication style.

```
System Prompt:
"You are a senior .NET architect with 15 years of experience.
You specialize in enterprise application design patterns,
microservices, and clean architecture.
When answering questions, always consider:
- Scalability implications
- SOLID principles
- Real-world production trade-offs"

User: "Should I use MediatR in my new project?"

Response: (Detailed, opinionated answer from an architect's perspective)
```

---

### 5. Output Format Control

**Explicitly specify the desired output structure.**

```
Prompt:
"Extract the following information from this customer email.
Return the result as a JSON object with these exact fields:

{
  "customerName": string,
  "issueType": "Billing" | "Technical" | "General",
  "urgency": "Low" | "Medium" | "High",
  "summary": string (max 50 words)
}

Email: 'Dear support, my name is John Smith and I've been charged
twice for my subscription this month. This is urgent as the extra
charge of $49.99 has put my account into overdraft. Please help ASAP.'"

Response:
{
  "customerName": "John Smith",
  "issueType": "Billing",
  "urgency": "High",
  "summary": "Customer charged twice for monthly subscription, extra $49.99 charge caused account overdraft, requesting immediate resolution."
}
```

**Pro tip for .NET developers:** When you get JSON back, you can deserialize it directly:
```csharp
var result = JsonSerializer.Deserialize<TicketClassification>(response);
```

---

### 6. Guardrails Pattern

**Set boundaries on what the AI should and should NOT do.**

```
System Prompt:
"You are a customer support assistant for Contoso Electronics.

RULES:
1. ONLY answer questions about Contoso products.
2. If asked about competitors, say: 'I can only help with Contoso products.'
3. NEVER provide legal or medical advice.
4. If you don't know the answer, say: 'Let me connect you with a human agent.'
5. NEVER make up product specifications.
6. Always include the product SKU if referencing a specific product."
```

---

## 🏗️ Prompt Structure Template

Here is a universal template for production-quality prompts:

```
┌─────────────────────────────────────────────┐
│  ROLE                                        │
│  "You are a [specific role]..."              │
├─────────────────────────────────────────────┤
│  CONTEXT                                     │
│  Background information the AI needs          │
├─────────────────────────────────────────────┤
│  TASK                                        │
│  Clear, specific instruction                  │
├─────────────────────────────────────────────┤
│  FORMAT                                      │
│  "Return as JSON / bullet points / table..." │
├─────────────────────────────────────────────┤
│  EXAMPLES (Few-shot)                         │
│  Input → Output pairs                         │
├─────────────────────────────────────────────┤
│  GUARDRAILS                                  │
│  "Do NOT...", "If unsure..."                  │
├─────────────────────────────────────────────┤
│  INPUT                                       │
│  The actual data/question to process          │
└─────────────────────────────────────────────┘
```

### Full Example Using the Template

```
ROLE: You are a senior C# code reviewer.

CONTEXT: The code being reviewed is part of a .NET 8 Web API for a financial 
services company. Code must follow Clean Architecture principles.

TASK: Review the following C# code. Identify bugs, performance issues, and 
code quality problems.

FORMAT: Return your review as a markdown list with these categories:
- 🐛 Bugs (critical issues)
- ⚡ Performance (optimization opportunities)
- 📝 Code Quality (style, naming, patterns)

For each item, include:
1. The line number or code snippet
2. The issue
3. A suggested fix with code

GUARDRAILS:
- Focus only on the code provided, don't suggest architectural changes
- Don't suggest changes that would break the existing public API
- Rate overall code quality on a scale of 1-10

INPUT:
[paste code here]
```

---

## 🧪 Practice Exercises

Try these in ChatGPT, Gemini, or Claude. **Compare results with different prompting strategies.**

### Exercise 1: Zero-Shot vs Few-Shot
```
Task: Convert natural language to SQL

Zero-shot: "Convert to SQL: Show me all orders over $100 from last month"

Few-shot: "Convert natural language to SQL.

Input: 'Show all customers from New York'
SQL: SELECT * FROM Customers WHERE City = 'New York';

Input: 'Count products by category'  
SQL: SELECT Category, COUNT(*) FROM Products GROUP BY Category;

Input: 'Show me all orders over $100 from last month'
SQL: "
```

### Exercise 2: Chain-of-Thought
```
Task: "A company has 3 warehouses. Warehouse A has 150 items,
B has 230 items, C has 180 items. If they need to redistribute
items equally and transfer costs $2 per item moved, what's the
minimum cost?"

Try WITHOUT "let's think step by step" and WITH it.
Compare the accuracy.
```

### Exercise 3: Output Format Control
```
Task: "Given this C# code, generate a markdown documentation 
block that includes: method name, parameters, return type, 
description, and a usage example."

Provide a C# method and see if the AI follows your format precisely.
```

### Exercise 4: Role + Guardrails
```
System: "You are a compliance officer for a bank. You can only 
discuss topics related to financial regulations and compliance. 
If asked about anything else, redirect to the appropriate department."

Then try asking it about cooking recipes and see if the guardrails hold.
```

---

## 💡 Pro Tips

1. **Be specific, not vague.** "Summarize this" → "Summarize this in 3 bullet points, each under 20 words."
2. **Use delimiters.** Wrap input data in triple backticks, XML tags, or quotes to separate it from instructions.
3. **Iterative refinement.** Start simple, test, refine. Prompt engineering is debugging for AI.
4. **Negative instructions work.** "Do NOT include code examples" is often more effective than "Only include text."
5. **Test with edge cases.** What happens with empty input? Malicious input? Unexpected languages?

---

## 📚 References

- [OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)
- [Anthropic's Prompt Engineering Guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering)
- [Microsoft Prompt Engineering Techniques](https://learn.microsoft.com/azure/ai-services/openai/concepts/prompt-engineering)
- [Chain-of-Thought Prompting Paper](https://arxiv.org/abs/2201.11903)

---

## ➡️ Next

Continue to **[Day 3: Microsoft.Extensions.AI](../Day-03-Microsoft-Extensions-AI/README.md)** — time to write C# code!
