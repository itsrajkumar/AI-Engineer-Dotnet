# Day 3: Multi-Plugin Environments

> **Type:** 💻 Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Give an agent access to multiple specialized plugins
- See the agent choose and chain tools from different domains
- Create plugins for Inventory, Tax, and Customer History
- Build a realistic enterprise agent scenario

---

## 💻 Code Sample

```csharp
using Microsoft.SemanticKernel;
using System.ComponentModel;

// =====================================================
// Plugin 1: Inventory Management
// =====================================================
public class InventoryManagementPlugin
{
    private readonly Dictionary<string, int> _inventory = new()
    {
        ["Laptop Pro 15"] = 25,
        ["Wireless Mouse"] = 150,
        ["USB-C Hub"] = 78,
        ["Mechanical Keyboard"] = 42,
        ["4K Monitor"] = 12
    };
    
    [KernelFunction, Description("Checks if a product is in stock and returns quantity")]
    public string CheckStock([Description("Product name")] string productName)
    {
        var match = _inventory.FirstOrDefault(p => 
            p.Key.Contains(productName, StringComparison.OrdinalIgnoreCase));
        return match.Key != null
            ? $"{match.Key}: {match.Value} units in stock"
            : $"Product '{productName}' not found in inventory";
    }
    
    [KernelFunction, Description("Lists all products in inventory with stock counts")]
    public string ListAllProducts()
    {
        return string.Join("\n", _inventory.Select(p => $"- {p.Key}: {p.Value} units"));
    }
    
    [KernelFunction, Description("Reserves units of a product for a customer order")]
    public string ReserveProduct(
        [Description("Product name")] string productName,
        [Description("Number of units to reserve")] int quantity)
    {
        var match = _inventory.FirstOrDefault(p =>
            p.Key.Contains(productName, StringComparison.OrdinalIgnoreCase));
        if (match.Key == null) return "Product not found";
        if (match.Value < quantity) return $"Insufficient stock. Only {match.Value} available.";
        _inventory[match.Key] -= quantity;
        return $"Reserved {quantity} units of {match.Key}. Remaining: {_inventory[match.Key]}";
    }
}

// =====================================================
// Plugin 2: Tax Calculation
// =====================================================
public class TaxCalculationPlugin
{
    private readonly Dictionary<string, double> _taxRates = new()
    {
        ["california"] = 7.25,
        ["texas"] = 6.25,
        ["new york"] = 8.0,
        ["florida"] = 6.0,
        ["washington"] = 6.5,
    };
    
    [KernelFunction, Description("Gets the tax rate for a US state")]
    public string GetTaxRate([Description("US state name")] string state)
    {
        return _taxRates.TryGetValue(state.ToLower(), out var rate)
            ? $"Tax rate for {state}: {rate}%"
            : $"Tax rate not found for {state}";
    }
    
    [KernelFunction, Description("Calculates total price including tax")]
    public string CalculateWithTax(
        [Description("Subtotal amount")] double subtotal,
        [Description("Tax rate percentage")] double taxRate)
    {
        var tax = Math.Round(subtotal * taxRate / 100, 2);
        var total = subtotal + tax;
        return $"Subtotal: ${subtotal:F2}, Tax ({taxRate}%): ${tax:F2}, Total: ${total:F2}";
    }
}

// =====================================================
// Plugin 3: Customer History
// =====================================================
public class CustomerHistoryPlugin
{
    [KernelFunction, Description("Gets purchase history for a customer by name or ID")]
    public string GetCustomerHistory([Description("Customer name or ID")] string customer)
    {
        return customer.ToLower() switch
        {
            var c when c.Contains("john") => """
                Customer: John Smith (ID: C-001)
                - 2024-01-15: Laptop Pro 15 × 1 ($1,299.00)
                - 2024-03-22: Wireless Mouse × 2 ($49.98)
                - 2024-06-10: USB-C Hub × 1 ($79.99)
                Lifetime Value: $1,428.97
                Loyalty Tier: Gold
                """,
            var c when c.Contains("jane") => """
                Customer: Jane Doe (ID: C-002)
                - 2024-02-28: 4K Monitor × 1 ($499.99)
                - 2024-05-15: Mechanical Keyboard × 1 ($149.99)
                Lifetime Value: $649.98
                Loyalty Tier: Silver
                """,
            _ => "Customer not found in our records."
        };
    }
    
    [KernelFunction, Description("Gets applicable discount based on customer loyalty tier")]
    public string GetLoyaltyDiscount([Description("Loyalty tier: Bronze, Silver, Gold, Platinum")] string tier)
    {
        return tier.ToLower() switch
        {
            "bronze" => "2% discount",
            "silver" => "5% discount",
            "gold" => "10% discount",
            "platinum" => "15% discount",
            _ => "No discount available"
        };
    }
}
```

### Wiring It All Together

```csharp
var kernel = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion("gpt-4o-mini", apiKey)
    .Build();

// Register ALL three plugins
kernel.Plugins.AddFromType<InventoryManagementPlugin>();
kernel.Plugins.AddFromType<TaxCalculationPlugin>();
kernel.Plugins.AddFromType<CustomerHistoryPlugin>();

// Agent system prompt
var systemPrompt = """
    You are a retail operations agent. You have access to:
    1. Inventory Management - check stock, list products, reserve units
    2. Tax Calculation - get tax rates, calculate totals
    3. Customer History - look up purchases, loyalty discounts
    
    When processing orders, always:
    - Check inventory first
    - Look up customer loyalty discount
    - Apply discount before tax
    - Calculate final total with appropriate state tax
    """;

// Complex multi-step request
var userRequest = """
    Process an order for John Smith: 
    2 Wireless Mouse units, shipping to California.
    Apply his loyalty discount to the order.
    """;

// The agent will autonomously:
// 1. Check John's customer history → Gold tier
// 2. Get Gold loyalty discount → 10%
// 3. Check Wireless Mouse stock → 150 units
// 4. Reserve 2 units
// 5. Calculate price: 2 × $24.99 = $49.98
// 6. Apply 10% discount: $44.98
// 7. Get California tax: 7.25%
// 8. Calculate total with tax: $48.24
```

---

## ➡️ Next

Continue to **[Day 4: Human-in-the-Loop](../Day-04-Human-in-the-Loop/README.md)**
