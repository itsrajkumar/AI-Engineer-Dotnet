# 📋 Prerequisites — Environment Setup

Before starting the 14-week roadmap, ensure your development environment is properly configured.

---

## ✅ Checklist

- [ ] .NET 10 SDK (LTS) installed
- [ ] IDE configured (Visual Studio 2026 / VS Code / Rider)
- [ ] Azure account with OpenAI access (or OpenAI API key)
- [ ] Git installed
- [ ] Docker Desktop installed (for vector databases)
- [ ] Required VS Code extensions installed

---

## 📖 Setup Guides

| Guide | Description |
|-------|-------------|
| [.NET 10 Setup](./dotnet-10-setup.md) | Install .NET 10 SDK and verify installation |
| [Azure Account Setup](./azure-account-setup.md) | Create Azure account, deploy OpenAI resource |
| [Environment Setup](./environment-setup.md) | Configure IDE, extensions, and developer tools |
| [Tools & Extensions](./tools-and-extensions.md) | Recommended tools, NuGet packages, and extensions |

---

## 🔑 API Keys You'll Need

### Option A: Azure OpenAI (Recommended for Enterprise)
1. Azure Subscription
2. Azure OpenAI resource with deployed models:
   - a current GPT-5-class deployment, such as `gpt-5.4-mini`, for chat completions
   - `text-embedding-3-large` or your current embedding deployment for embeddings

### Option B: OpenAI API (Simpler Setup)
1. OpenAI account at [platform.openai.com](https://platform.openai.com)
2. API key with billing enabled

### Option C: Local Models (Free, No API Keys)
1. [Ollama](https://ollama.com/) installed locally
2. Models: `llama4-scout`, `deepseek-v4-flash`, `mistral-small-4`, `phi-4-mini`, and `bge-m3`
3. Foundry Local v1.2 GA (optional, for enterprise simulation)

> **💡 Tip:** You can start with Option B or C and migrate to Azure later. The code samples support all three options.

### Option D: Python (For Weeks 11-12 Only)
1. Python 3.12+ (Only needed if you choose to run the Hugging Face reference scripts in Weeks 11-12)
2. Hugging Face CLI

---

## 📦 Core NuGet Packages

These packages will be used throughout the roadmap:

```xml
<!-- AI Abstraction Layer -->
<PackageReference Include="Microsoft.Extensions.AI" Version="10.7.0" />
<PackageReference Include="Microsoft.Extensions.AI.OpenAI" Version="10.7.0" />
<PackageReference Include="Microsoft.Extensions.AI.Ollama" Version="10.7.0" />

<!-- Azure OpenAI SDK -->
<PackageReference Include="Azure.AI.OpenAI" Version="2.*" />

<!-- Microsoft Agent Framework (Replaces Semantic Kernel) -->
<PackageReference Include="Microsoft.AgentFramework" Version="1.0.0" />
<PackageReference Include="Microsoft.AgentFramework.OpenAI" Version="1.0.0" />

<!-- Vector Databases -->
<PackageReference Include="MongoDB.Driver" Version="3.*" />
<PackageReference Include="Npgsql" Version="9.*" />

<!-- Protocols -->
<PackageReference Include="ModelContextProtocol.AspNetCore" Version="1.4.0" />

<!-- Utilities -->
<PackageReference Include="Microsoft.Extensions.Configuration" Version="10.*" />
<PackageReference Include="Microsoft.Extensions.Configuration.UserSecrets" Version="10.*" />
```

---

## 🔒 Managing Secrets

**Never commit API keys to source control.** Use .NET User Secrets:

```bash
# Initialize user secrets in a project
dotnet user-secrets init

# Set your API key
dotnet user-secrets set "OpenAI:ApiKey" "sk-your-key-here"
dotnet user-secrets set "OpenAI:ModelId" "gpt-5.4-mini"

# For Azure OpenAI
dotnet user-secrets set "AzureOpenAI:Endpoint" "https://your-resource.openai.azure.com/"
dotnet user-secrets set "AzureOpenAI:ApiKey" "your-azure-key"
dotnet user-secrets set "AzureOpenAI:DeploymentName" "gpt-5.4-mini"
```

---

## ➡️ Next Step

Once your environment is ready, proceed to **[Week 1, Day 1: AI Theory & Terminology](../Week-01-AI-Fundamentals-and-DotNet-API-Layer/Day-01-AI-Theory-and-Terminology/README.md)**.
