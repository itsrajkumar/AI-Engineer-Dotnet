# 📋 Prerequisites — Environment Setup

Before starting the 6-week roadmap, ensure your development environment is properly configured.

---

## ✅ Checklist

- [ ] .NET 8 SDK installed
- [ ] IDE configured (Visual Studio 2022 / VS Code / Rider)
- [ ] Azure account with OpenAI access (or OpenAI API key)
- [ ] Git installed
- [ ] Docker Desktop installed (for vector databases)
- [ ] Required VS Code extensions installed

---

## 📖 Setup Guides

| Guide | Description |
|-------|-------------|
| [.NET 8 Setup](./dotnet-8-setup.md) | Install .NET 8 SDK and verify installation |
| [Azure Account Setup](./azure-account-setup.md) | Create Azure account, deploy OpenAI resource |
| [Environment Setup](./environment-setup.md) | Configure IDE, extensions, and developer tools |
| [Tools & Extensions](./tools-and-extensions.md) | Recommended tools, NuGet packages, and extensions |

---

## 🔑 API Keys You'll Need

### Option A: Azure OpenAI (Recommended for Enterprise)
1. Azure Subscription
2. Azure OpenAI resource with deployed models:
   - `gpt-4o` or `gpt-4o-mini` (for chat completions)
   - `text-embedding-3-small` (for embeddings)

### Option B: OpenAI API (Simpler Setup)
1. OpenAI account at [platform.openai.com](https://platform.openai.com)
2. API key with billing enabled

### Option C: Local Models (Free, No API Keys)
1. [Ollama](https://ollama.com/) installed locally
2. Models: `llama3.1`, `nomic-embed-text`

> **💡 Tip:** You can start with Option B or C and migrate to Azure later. The code samples support all three options.

---

## 📦 Core NuGet Packages

These packages will be used throughout the roadmap:

```xml
<!-- AI Abstraction Layer -->
<PackageReference Include="Microsoft.Extensions.AI" Version="9.0.0-preview.*" />
<PackageReference Include="Microsoft.Extensions.AI.OpenAI" Version="9.0.0-preview.*" />

<!-- Azure OpenAI SDK -->
<PackageReference Include="Azure.AI.OpenAI" Version="2.*" />

<!-- Microsoft Semantic Kernel -->
<PackageReference Include="Microsoft.SemanticKernel" Version="1.*" />
<PackageReference Include="Microsoft.SemanticKernel.Connectors.OpenAI" Version="1.*" />

<!-- Vector Databases -->
<PackageReference Include="MongoDB.Driver" Version="2.*" />
<PackageReference Include="Npgsql" Version="8.*" />

<!-- Utilities -->
<PackageReference Include="Microsoft.Extensions.Configuration" Version="8.*" />
<PackageReference Include="Microsoft.Extensions.Configuration.UserSecrets" Version="8.*" />
```

---

## 🔒 Managing Secrets

**Never commit API keys to source control.** Use .NET User Secrets:

```bash
# Initialize user secrets in a project
dotnet user-secrets init

# Set your API key
dotnet user-secrets set "OpenAI:ApiKey" "sk-your-key-here"
dotnet user-secrets set "OpenAI:ModelId" "gpt-4o-mini"

# For Azure OpenAI
dotnet user-secrets set "AzureOpenAI:Endpoint" "https://your-resource.openai.azure.com/"
dotnet user-secrets set "AzureOpenAI:ApiKey" "your-azure-key"
dotnet user-secrets set "AzureOpenAI:DeploymentName" "gpt-4o"
```

---

## ➡️ Next Step

Once your environment is ready, proceed to **[Week 1, Day 1: AI Theory & Terminology](../Week-01-AI-Fundamentals-and-DotNet-API-Layer/Day-01-AI-Theory-and-Terminology/README.md)**.
