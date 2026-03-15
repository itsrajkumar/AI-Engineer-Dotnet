# Deployment Guide

## Local Development

### Prerequisites
- .NET 8 SDK
- Docker Desktop
- API key (OpenAI or Azure OpenAI)

### Quick Start

```powershell
# 1. Start infrastructure
docker-compose up -d

# 2. Navigate to API project
cd src/RetailAssistant.API

# 3. Set secrets
dotnet user-secrets init
dotnet user-secrets set "OpenAI:ApiKey" "sk-your-key"
dotnet user-secrets set "ConnectionStrings:PostgreSQL" "Host=localhost;Database=retail;Username=postgres;Password=postgres"

# 4. Run migrations
dotnet ef database update

# 5. Run the API
dotnet run
```

## Docker Compose (Full Stack)

### docker-compose.yml

```yaml
version: '3.8'

services:
  api:
    build:
      context: ./src
      dockerfile: RetailAssistant.API/Dockerfile
    ports:
      - "5000:8080"
    environment:
      - OpenAI__ApiKey=${OPENAI_API_KEY}
      - ConnectionStrings__PostgreSQL=Host=postgres;Database=retail;Username=postgres;Password=postgres
    depends_on:
      - postgres

  postgres:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: retail
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### Running

```powershell
# Set environment variable
$env:OPENAI_API_KEY = "sk-your-key"

# Start everything
docker-compose up --build

# API available at http://localhost:5000
```

## Azure Deployment

### Resources Needed
1. Azure App Service (B1 tier or higher)
2. Azure Database for PostgreSQL (Flexible Server with pgvector)
3. Azure OpenAI Service
4. Azure Key Vault (for secrets)

### Steps

```powershell
# 1. Create resource group
az group create -n rg-retail-assistant -l eastus

# 2. Create PostgreSQL
az postgres flexible-server create `
  -g rg-retail-assistant `
  -n retail-postgres `
  --sku-name Standard_B1ms

# 3. Enable pgvector extension
az postgres flexible-server parameter set `
  -g rg-retail-assistant `
  -s retail-postgres `
  -n azure.extensions `
  -v vector

# 4. Deploy App Service
az webapp up -g rg-retail-assistant -n retail-assistant-api --runtime "DOTNETCORE:8.0"
```

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `OpenAI:ApiKey` | OpenAI API key | Yes* |
| `AzureOpenAI:Endpoint` | Azure OpenAI endpoint | Yes* |
| `AzureOpenAI:ApiKey` | Azure OpenAI key | Yes* |
| `ConnectionStrings:PostgreSQL` | PostgreSQL connection string | Yes |

*Either OpenAI or AzureOpenAI keys required, not both.
