# .NET 8 SDK Setup

## Installation

### Windows
1. Download from [dotnet.microsoft.com/download](https://dotnet.microsoft.com/download/dotnet/8.0)
2. Run the installer
3. Verify installation:
   ```powershell
   dotnet --version
   # Should output: 8.0.x
   
   dotnet --list-sdks
   # Should show 8.0.x installed
   ```

### Using winget (Windows Package Manager)
```powershell
winget install Microsoft.DotNet.SDK.8
```

## Verify Your Setup

Create a quick test project:

```powershell
# Create a new console app
dotnet new console -n TestSetup -o ./TestSetup
cd TestSetup

# Run it
dotnet run
# Should output: Hello, World!

# Clean up
cd ..
Remove-Item -Recurse -Force TestSetup
```

## Required .NET Workloads

```powershell
# Install ASP.NET Core (for Web API projects)
dotnet workload install aspire
```

## Global Tools (Optional but Recommended)

```powershell
# Entity Framework Core tools
dotnet tool install --global dotnet-ef

# HTTP REPL for testing APIs
dotnet tool install --global Microsoft.dotnet-httprepl

# User secrets manager (built-in, no install needed)
```

## C# 12 Features You Should Know

This roadmap uses modern C# features extensively:

```csharp
// Primary constructors
public class ChatService(ILogger<ChatService> logger)
{
    public void Log(string message) => logger.LogInformation(message);
}

// Collection expressions
int[] numbers = [1, 2, 3, 4, 5];

// Raw string literals (C# 11+, used heavily for prompts)
var prompt = """
    You are a helpful AI assistant.
    Answer the user's question clearly and concisely.
    """;

// Pattern matching
var result = response switch
{
    { StatusCode: 200 } => "Success",
    { StatusCode: 429 } => "Rate limited",
    _ => "Unknown error"
};
```
