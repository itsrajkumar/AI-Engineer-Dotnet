# Environment Setup

## IDE Options

### Visual Studio 2022 (Recommended for Full .NET Experience)
1. Download [Visual Studio 2022 Community](https://visualstudio.microsoft.com/downloads/) (Free)
2. Install with these workloads:
   - ✅ ASP.NET and web development
   - ✅ .NET desktop development
3. Recommended extensions:
   - GitHub Copilot (optional, for AI-assisted coding)
   - Markdown Editor

### VS Code (Lightweight, Cross-Platform)
1. Download [VS Code](https://code.visualstudio.com/)
2. Install extensions (see [Tools & Extensions](./tools-and-extensions.md))

### JetBrains Rider (Premium, Excellent IntelliSense)
1. Download [Rider](https://www.jetbrains.com/rider/)
2. Use the free 30-day trial or EAP version

---

## Docker Desktop

Required for running PostgreSQL with pgvector and MongoDB locally.

1. Download [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Install and start
3. Verify:
```powershell
docker --version
docker-compose --version
```

### Quick Docker Compose for Learning

Create this file at the repo root as `docker-compose.yml`:

```yaml
version: '3.8'

services:
  # PostgreSQL with pgvector extension
  postgres:
    image: pgvector/pgvector:pg16
    container_name: ai-engineer-postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: ai_engineer
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  # MongoDB for document + vector search
  mongodb:
    image: mongo:7
    container_name: ai-engineer-mongo
    ports:
      - "27017:27017"
    volumes:
      - mongodata:/data/db

volumes:
  pgdata:
  mongodata:
```

```powershell
# Start the databases
docker-compose up -d

# Verify they're running
docker ps
```

---

## Git Configuration

```powershell
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Use VS Code as default editor
git config --global core.editor "code --wait"
```

---

## Terminal Setup

Using **Windows Terminal** with PowerShell is recommended. Install from Microsoft Store if not already available.

Useful aliases for your `$PROFILE`:
```powershell
# Open PowerShell profile
notepad $PROFILE

# Add these aliases
function dn { dotnet run }
function dnb { dotnet build }
function dnt { dotnet test }
function dnw { dotnet watch run }
```
