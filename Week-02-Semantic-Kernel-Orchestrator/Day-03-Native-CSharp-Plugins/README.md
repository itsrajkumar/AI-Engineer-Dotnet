# Day 3: Native C# Plugins

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Write a C# class as a Semantic Kernel plugin
- Use `[KernelFunction]` and `[Description]` attributes
- Register plugins with the Kernel
- Understand how the AI "sees" your C# methods

---

## 📖 What Are Native Plugins?

A **Native Plugin** is a regular C# class whose methods are exposed to the AI. The AI can decide to call these methods when it needs to perform a specific task.

```
┌──────────────────────────┐
│  AI Model (GPT-4o)       │
│                          │
│  "I need to calculate    │
│   a 15% discount on      │
│   $199.99..."            │
│                          │
│  → Calls YOUR C# method! │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  Your C# Plugin          │
│                          │
│  [KernelFunction]        │
│  CalculateDiscount(      │
│    price: 199.99,        │
│    percent: 15           │
│  ) → returns 169.99     │
└──────────────────────────┘
```

---

## 💻 Code Samples

### Plugin Class

```csharp
using Microsoft.SemanticKernel;
using System.ComponentModel;

/// <summary>
/// A plugin that provides pricing and discount calculations.
/// The AI can invoke these methods when it needs to do math.
/// </summary>
public class PricingPlugin
{
    /// <summary>
    /// Calculates the discounted price for a product.
    /// </summary>
    [KernelFunction("CalculateDiscount")]
    [Description("Calculates the final price after applying a percentage discount")]
    public double CalculateDiscount(
        [Description("The original price in USD")] double originalPrice,
        [Description("The discount percentage (e.g., 15 for 15%)")] double discountPercent)
    {
        var discountAmount = originalPrice * (discountPercent / 100);
        var finalPrice = originalPrice - discountAmount;
        return Math.Round(finalPrice, 2);
    }

    /// <summary>
    /// Calculates tax for a given amount.
    /// </summary>
    [KernelFunction("CalculateTax")]
    [Description("Calculates the tax amount for a given price and tax rate")]
    public double CalculateTax(
        [Description("The price to calculate tax on")] double price,
        [Description("The tax rate percentage (e.g., 8.5 for 8.5%)")] double taxRate)
    {
        return Math.Round(price * (taxRate / 100), 2);
    }

    /// <summary>
    /// Gets the current promotion for a product category.
    /// </summary>
    [KernelFunction("GetCurrentPromotion")]
    [Description("Gets the current active promotion for a product category")]
    public string GetCurrentPromotion(
        [Description("Product category: Electronics, Clothing, Books, Home")] string category)
    {
        return category.ToLower() switch
        {
            "electronics" => "15% off all electronics this week!",
            "clothing" => "Buy 2 get 1 free on all clothing!",
            "books" => "20% off bestsellers!",
            "home" => "Free shipping on home items over $50!",
            _ => "No active promotions for this category."
        };
    }
}
```

### Registering the Plugin

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.SemanticKernel;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

var kernel = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion("gpt-4o-mini", config["OpenAI:ApiKey"]!)
    .Build();

// Register the plugin with the Kernel
kernel.Plugins.AddFromType<PricingPlugin>();

// Now the AI knows about CalculateDiscount, CalculateTax, and GetCurrentPromotion!

// Direct invocation (calling plugin manually):
var discountResult = await kernel.InvokeAsync<double>(
    "PricingPlugin", "CalculateDiscount",
    new KernelArguments
    {
        ["originalPrice"] = 199.99,
        ["discountPercent"] = 15
    });

Console.WriteLine($"Discounted price: ${discountResult}"); // $169.99

// The AI will learn to call these in Day 4 (Tool Calling)!
```

---

## 🔑 Important: The `[Description]` Attribute

The `[Description]` attribute is **critical**. The AI reads these descriptions to understand:
- What the function does
- What each parameter means
- When to call this function vs. another

```csharp
// BAD — AI won't understand when to use this
[KernelFunction("Calc")]
public double Calc(double x, double y) => x * y;

// GOOD — AI knows exactly when and how to use this
[KernelFunction("CalculateDiscount")]
[Description("Calculates the final price after applying a percentage discount")]
public double CalculateDiscount(
    [Description("The original price in USD")] double originalPrice,
    [Description("The discount percentage (e.g., 15 for 15%)")] double discountPercent)
{
    return Math.Round(originalPrice * (1 - discountPercent / 100), 2);
}
```

---

## ➡️ Next

Continue to **[Day 4: Tool Calling](../Day-04-Tool-Calling/README.md)** — where the AI actually CALLS these methods!
