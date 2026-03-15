# Tools & Extensions

## VS Code Extensions

### Essential
| Extension | ID | Purpose |
|-----------|-----|---------|
| C# Dev Kit | `ms-dotnettools.csdevkit` | Full C# language support |
| .NET Runtime Install | `ms-dotnettools.vscode-dotnet-runtime` | .NET runtime management |
| REST Client | `humao.rest-client` | Test APIs directly from VS Code |
| Polyglot Notebooks | `ms-dotnettools.dotnet-interactive-vscode` | Interactive C# notebooks |

### Recommended
| Extension | ID | Purpose |
|-----------|-----|---------|
| Markdown All in One | `yzhang.markdown-all-in-one` | Better markdown editing |
| Mermaid Markdown | `bierner.markdown-mermaid` | Preview Mermaid diagrams |
| Docker | `ms-azuretools.vscode-docker` | Docker container management |
| GitLens | `eamodio.gitlens` | Enhanced Git integration |
| Azure Tools | `ms-vscode.vscode-node-azure-pack` | Azure resource management |
| Thunder Client | `rangav.vscode-thunder-client` | API testing (Postman alternative) |

### Install All at Once
```powershell
code --install-extension ms-dotnettools.csdevkit
code --install-extension ms-dotnettools.vscode-dotnet-runtime
code --install-extension humao.rest-client
code --install-extension ms-dotnettools.dotnet-interactive-vscode
code --install-extension yzhang.markdown-all-in-one
code --install-extension bierner.markdown-mermaid
code --install-extension ms-azuretools.vscode-docker
code --install-extension eamodio.gitlens
```

---

## Recommended NuGet Packages by Week

### Week 1: AI Fundamentals
```xml
<PackageReference Include="Azure.AI.OpenAI" Version="2.*" />
<PackageReference Include="Microsoft.Extensions.AI" Version="9.0.0-preview.*" />
<PackageReference Include="Microsoft.Extensions.AI.OpenAI" Version="9.0.0-preview.*" />
<PackageReference Include="Microsoft.Extensions.Configuration.UserSecrets" Version="8.*" />
```

### Week 2: Semantic Kernel
```xml
<PackageReference Include="Microsoft.SemanticKernel" Version="1.*" />
<PackageReference Include="Microsoft.SemanticKernel.Connectors.OpenAI" Version="1.*" />
<PackageReference Include="Microsoft.SemanticKernel.PromptTemplates.Handlebars" Version="1.*" />
```

### Week 3: Embeddings
```xml
<PackageReference Include="Microsoft.Extensions.AI" Version="9.0.0-preview.*" />
<PackageReference Include="UglyToad.PdfPig" Version="0.1.*" />
<PackageReference Include="Microsoft.ML.Tokenizers" Version="0.22.*" />
```

### Week 4: Vector Databases
```xml
<PackageReference Include="MongoDB.Driver" Version="2.*" />
<PackageReference Include="Npgsql" Version="8.*" />
<PackageReference Include="Pgvector" Version="0.2.*" />
<PackageReference Include="Microsoft.SemanticKernel.Connectors.MongoDB" Version="1.*" />
```

### Weeks 5-6: RAG & Agents
```xml
<PackageReference Include="Microsoft.SemanticKernel" Version="1.*" />
<PackageReference Include="Microsoft.SemanticKernel.Planners.OpenAI" Version="1.*" />
```

---

## Online Tools

| Tool | URL | Purpose |
|------|-----|---------|
| OpenAI Playground | platform.openai.com/playground | Test prompts interactively |
| Azure AI Studio | ai.azure.com | Manage Azure AI resources |
| Tokenizer | platform.openai.com/tokenizer | Visualize token counts |
| Embedding Projector | projector.tensorflow.org | Visualize embeddings in 3D |
| Mermaid Live Editor | mermaid.live | Create architecture diagrams |

---

## Useful CLI Commands

```powershell
# Create a new console project
dotnet new console -n ProjectName

# Create a new Web API project
dotnet new webapi -n ProjectName

# Add a NuGet package
dotnet add package PackageName

# Initialize user secrets
dotnet user-secrets init

# Run with watch (hot reload)
dotnet watch run

# Build in Release mode
dotnet build -c Release
```
