# Azure Account & OpenAI Setup

## Option A: Azure OpenAI (Recommended for Enterprise)

### Step 1: Create Azure Account
1. Go to [azure.microsoft.com/free](https://azure.microsoft.com/free/)
2. Sign up for a free account (includes $200 credit for 30 days)
3. Verify your identity with a phone number and credit card

### Step 2: Request Azure OpenAI Access
1. Navigate to [Azure OpenAI Service](https://azure.microsoft.com/products/ai-services/openai-service)
2. Apply for access at [aka.ms/oai/access](https://aka.ms/oai/access)
3. Fill out the registration form with your use case
4. Wait for approval (usually 1-2 business days)

### Step 3: Create an Azure OpenAI Resource
```
1. Go to Azure Portal → Create a resource
2. Search for "Azure OpenAI"
3. Click Create
4. Fill in:
   - Subscription: Your subscription
   - Resource group: Create new → "rg-ai-engineer-learning"
   - Region: East US (or closest to you with availability)
   - Name: "oai-ai-engineer-learning"
   - Pricing tier: Standard S0
5. Review + Create
```

### Step 4: Deploy Models
In the Azure OpenAI Studio:

```
1. Go to your resource → Model deployments → Manage Deployments
2. Deploy the following models:

Model 1 (Chat):
   - Model: gpt-4o-mini
   - Deployment name: gpt-4o-mini
   - Tokens per minute: 30K (free tier sufficient)

Model 2 (Embeddings):
   - Model: text-embedding-3-small
   - Deployment name: text-embedding-3-small
   - Tokens per minute: 120K
```

### Step 5: Get Your Credentials
```
Go to Azure Portal → Your OpenAI Resource → Keys and Endpoint

You'll need:
- Endpoint: https://your-resource-name.openai.azure.com/
- Key 1: (copy this)
- API Version: 2024-08-01-preview (or latest)
```

---

## Option B: OpenAI API (Simpler Setup)

### Step 1: Create Account
1. Go to [platform.openai.com](https://platform.openai.com)
2. Sign up with email or Google/Microsoft account

### Step 2: Add Billing
1. Go to Settings → Billing → Add payment method
2. Add $10-20 credit (this will last for weeks of learning)

### Step 3: Create API Key
1. Go to [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Click "Create new secret key"
3. Name: "AI-Engineer-Learning"
4. Copy and save the key (you won't see it again!)

### Models Available
- Chat: `gpt-4o-mini` (~$0.15 per 1M input tokens — very cheap for learning)
- Embeddings: `text-embedding-3-small` (~$0.02 per 1M tokens)

---

## Option C: Local Models with Ollama (Free)

### Step 1: Install Ollama
1. Download from [ollama.com](https://ollama.com/)
2. Run the installer
3. Verify: `ollama --version`

### Step 2: Pull Models
```powershell
# Chat model
ollama pull llama3.1:8b

# Embedding model
ollama pull nomic-embed-text

# List downloaded models
ollama list
```

### Step 3: Verify
```powershell
# Start a chat to verify
ollama run llama3.1:8b "Say hello in one sentence"

# The Ollama API runs at http://localhost:11434
```

> **⚠️ Note:** Local models are great for learning but may not support all features (like advanced function calling). The roadmap code samples include fallback configurations for Ollama where possible.

---

## Cost Estimation

For the complete 6-week roadmap using OpenAI API:

| Usage | Estimated Cost |
|-------|---------------|
| Chat completions (gpt-4o-mini) | ~$2-5 |
| Embeddings (text-embedding-3-small) | ~$0.50-1 |
| **Total** | **~$3-6** |

Using Ollama: **$0** (runs locally)
