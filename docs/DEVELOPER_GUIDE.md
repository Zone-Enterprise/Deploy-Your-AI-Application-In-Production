# Developer Guide

## Table of Contents
1. [Getting Started](#getting-started)
2. [Development Environment Setup](#development-environment-setup)
3. [Infrastructure Customization](#infrastructure-customization)
4. [AI Model Configuration](#ai-model-configuration)
5. [Application Development](#application-development)
6. [Testing and Validation](#testing-and-validation)
7. [Deployment Patterns](#deployment-patterns)
8. [Troubleshooting](#troubleshooting)

## Getting Started

### Prerequisites for Developers

```mermaid
graph TD
    subgraph "Development Prerequisites"
        A[Azure Subscription<br/>Contributor Access]
        B[Development Tools<br/>VS Code + Extensions]
        C[Azure CLI + AZD<br/>Latest Versions]
        D[GitHub Account<br/>For Codespaces]
    end
    
    subgraph "Optional Tools"
        E[Docker Desktop<br/>For Container Development]
        F[Python 3.9+<br/>For Sample Apps]
        G[Node.js 18+<br/>For Web Development]
    end
    
    A --> C
    B --> C
    C --> D
    
    classDef required fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef optional fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    
    class A,B,C,D required
    class E,F,G optional
```

### Quick Development Setup

1. **Choose Your Development Environment**:
   - **GitHub Codespaces** (Recommended): Pre-configured environment
   - **Local Development**: Your machine with required tools
   - **Dev Containers**: Consistent containerized environment

2. **Clone and Initialize**:
```bash
git clone https://github.com/Zone-Enterprise/Deploy-Your-AI-Application-In-Production.git
cd Deploy-Your-AI-Application-In-Production
```

3. **Configure Environment**:
```bash
# Initialize Azure Developer CLI
azd auth login
azd init

# Set environment variables
azd env set AZURE_LOCATION "eastus2"
azd env set AZURE_SUBSCRIPTION_ID "your-subscription-id"
```

## Development Environment Setup

### GitHub Codespaces (Recommended)

Codespaces provides a fully configured development environment in the cloud:

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant CS as Codespace
    participant AZ as Azure
    
    Dev->>GH: Open in Codespaces
    GH->>CS: Create environment
    CS->>CS: Install dependencies
    CS->>CS: Configure tools
    Dev->>CS: Start development
    CS->>AZ: Deploy infrastructure
    AZ-->>CS: Deployment status
    CS-->>Dev: Ready for development
```

**Features**:
- Pre-installed Azure CLI, AZD, and development tools
- Configured VS Code with extensions
- Docker support for containerized development
- Integrated terminal and debugging

### Local Development Environment

Set up your local machine for development:

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Install Azure Developer CLI
curl -fsSL https://aka.ms/install-azd.sh | bash

# Install VS Code extensions
code --install-extension ms-vscode.vscode-bicep
code --install-extension ms-azure-devops.azure-devops
code --install-extension ms-vscode.azure-account
```

### Development Container

Use the provided dev container for consistent environment:

```json
// .devcontainer/devcontainer.json
{
    "name": "AI Foundry Development",
    "image": "mcr.microsoft.com/devcontainers/universal:latest",
    "features": {
        "ghcr.io/devcontainers/features/azure-cli:1": {},
        "ghcr.io/azure/azure-dev-cli/azd:latest": {}
    },
    "customizations": {
        "vscode": {
            "extensions": [
                "ms-vscode.vscode-bicep",
                "ms-azure-devops.azure-devops"
            ]
        }
    }
}
```

## Infrastructure Customization

### Bicep Template Structure

```mermaid
graph TB
    subgraph "Infrastructure Components"
        MAIN[main.bicep<br/>Entry Point]
        
        subgraph "Core Modules"
            NET[virtualNetwork.bicep]
            VM[virtualMachine.bicep]
            KV[keyvault.bicep]
            ST[storageAccount.bicep]
        end
        
        subgraph "AI Services"
            CS[cognitive-services/main.bicep]
            AIF[ai-foundry-project/main.bicep]
            SEARCH[aisearch.bicep]
        end
        
        subgraph "Optional Services"
            APIM[apim.bicep]
            COS[cosmosDb.bicep]
            SQL[sqlServer.bicep]
            APP[appservice.bicep]
        end
    end
    
    MAIN --> NET
    MAIN --> VM
    MAIN --> KV
    MAIN --> ST
    MAIN --> CS
    MAIN --> AIF
    
    NET -.-> SEARCH
    NET -.-> APIM
    NET -.-> COS
    NET -.-> SQL
    NET -.-> APP
    
    classDef core fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef ai fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef optional fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    
    class MAIN,NET,VM,KV,ST core
    class CS,AIF,SEARCH ai
    class APIM,COS,SQL,APP optional
```

### Key Configuration Files

| File | Purpose | Customization Options |
|------|---------|----------------------|
| `main.bicep` | Primary template | Resource naming, feature flags |
| `main.parameters.json` | Deployment parameters | Environment-specific values |
| `azure.yaml` | AZD configuration | Deployment hooks, environment settings |

### Common Customizations

#### 1. Adding Custom AI Models

```bicep
// In main.bicep
param customModelDeployments modelDeploymentType[] = [
  {
    name: 'custom-gpt-4'
    modelName: 'gpt-4'
    version: '0613'
    capacity: 50
  }
]
```

#### 2. Custom Network Configuration

```bicep
// Custom subnet configuration
param customSubnets array = [
  {
    name: 'snet-custom-app'
    addressPrefix: '10.0.3.0/24'
    networkSecurityGroupId: customAppNsg.outputs.resourceId
  }
]
```

#### 3. Additional Storage Accounts

```bicep
// Add specialized storage
module dataLakeStorage 'modules/storageAccount.bicep' = {
  name: 'data-lake-storage'
  params: {
    storageName: 'stdl${sanitizedName}${resourceToken}'
    kind: 'StorageV2'
    sku: 'Standard_LRS'
    isHnsEnabled: true  // Data Lake Gen2
  }
}
```

## AI Model Configuration

### Supported AI Models

```mermaid
graph TB
    subgraph "Text Generation"
        GPT4[GPT-4o<br/>Latest Model]
        GPT35[GPT-3.5-turbo<br/>Cost Effective]
    end
    
    subgraph "Embeddings"
        EMB3[text-embedding-3-small<br/>High Performance]
        EMB2[text-embedding-ada-002<br/>Standard]
    end
    
    subgraph "Specialized Models"
        DALLE[DALL-E 3<br/>Image Generation]
        WHISPER[Whisper<br/>Speech Recognition]
        TTS[Text-to-Speech<br/>Voice Synthesis]
    end
    
    subgraph "Custom Models"
        FINE[Fine-tuned Models<br/>Domain Specific]
        BYOM[Bring Your Own Model<br/>Custom Deployments]
    end
    
    classDef text fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef embed fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef special fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef custom fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    
    class GPT4,GPT35 text
    class EMB3,EMB2 embed
    class DALLE,WHISPER,TTS special
    class FINE,BYOM custom
```

### Model Deployment Configuration

```bicep
// Define AI model deployments
var aiModelDeployments = [
  {
    name: 'gpt-4o'
    model: {
      name: 'gpt-4o'
      version: '2024-05-13'
      format: 'OpenAI'
    }
    sku: {
      name: 'GlobalStandard'
      capacity: 150
    }
  }
  {
    name: 'text-embedding-3-small'
    model: {
      name: 'text-embedding-3-small'
      version: '1'
      format: 'OpenAI'
    }
    sku: {
      name: 'GlobalStandard'
      capacity: 100
    }
  }
]
```

### Quota Management

Before deploying, check quota availability:

```bash
# Check current quota usage
az cognitiveservices usage list --location "eastus2"

# Request quota increase
az cognitiveservices account create \
  --name "quota-check" \
  --resource-group "temp-rg" \
  --kind "OpenAI" \
  --sku "S0" \
  --location "eastus2" \
  --custom-domain "quota-check-$(date +%s)"
```

## Application Development

### Sample Application Architecture

```mermaid
graph TB
    subgraph "Frontend Layer"
        WEB[React Web App<br/>User Interface]
        API_CLIENT[API Client<br/>Authentication]
    end
    
    subgraph "API Layer"
        APIM[API Management<br/>Gateway]
        AUTH[Azure AD<br/>Authentication]
    end
    
    subgraph "Application Layer"
        APP[App Service<br/>Python/Node.js]
        FUNC[Azure Functions<br/>Serverless Logic]
    end
    
    subgraph "AI Layer"
        AIP[AI Foundry Project<br/>Model Management]
        AOI[Azure OpenAI<br/>Language Models]
        AIS[AI Search<br/>Vector Search]
    end
    
    subgraph "Data Layer"
        COS[Cosmos DB<br/>Chat History]
        ST[Storage Account<br/>Documents]
        REDIS[Redis Cache<br/>Session Data]
    end
    
    WEB --> API_CLIENT
    API_CLIENT --> APIM
    APIM --> AUTH
    APIM --> APP
    APP --> FUNC
    FUNC --> AIP
    AIP --> AOI
    AIP --> AIS
    APP --> COS
    APP --> ST
    APP --> REDIS
    
    classDef frontend fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef api fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef app fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef ai fill:#ffebee,stroke:#d32f2f,stroke-width:2px
    classDef data fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    
    class WEB,API_CLIENT frontend
    class APIM,AUTH api
    class APP,FUNC app
    class AIP,AOI,AIS ai
    class COS,ST,REDIS data
```

### Development Workflow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Code as VS Code/Codespace
    participant AZD as Azure Developer CLI
    participant AZ as Azure
    participant AI as AI Foundry
    
    Dev->>Code: Write application code
    Code->>Code: Local testing
    Dev->>AZD: azd deploy
    AZD->>AZ: Deploy infrastructure
    AZ->>AI: Configure AI services
    AI-->>AZ: Service endpoints
    AZ-->>AZD: Deployment complete
    AZD->>AZ: Deploy application
    AZ-->>Dev: Application URL
    
    Note over Dev,AI: Iterative Development
    Dev->>Code: Update code
    Dev->>AZD: azd deploy --skip-infra
    AZD->>AZ: Update application only
    AZ-->>Dev: Updated application
```

### Code Structure

```
src/
├── backend/
│   ├── app.py                 # Main application
│   ├── models/                # Data models
│   ├── services/              # Business logic
│   ├── api/                   # API endpoints
│   └── utils/                 # Utilities
├── frontend/
│   ├── src/
│   │   ├── components/        # React components
│   │   ├── services/          # API services
│   │   ├── utils/             # Utilities
│   │   └── App.js             # Main app
│   └── public/
└── shared/
    ├── models/                # Shared data models
    └── constants/             # Constants
```

### Environment Configuration

```python
# app.py - Environment configuration
import os
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

class Config:
    def __init__(self):
        self.credential = DefaultAzureCredential()
        self.key_vault_url = os.getenv("AZURE_KEY_VAULT_URL")
        self.secret_client = SecretClient(
            vault_url=self.key_vault_url,
            credential=self.credential
        )
    
    def get_secret(self, secret_name):
        return self.secret_client.get_secret(secret_name).value
    
    @property
    def openai_endpoint(self):
        return self.get_secret("openai-endpoint")
    
    @property
    def search_endpoint(self):
        return self.get_secret("search-endpoint")
```

## Testing and Validation

### Testing Strategy

```mermaid
graph TB
    subgraph "Testing Pyramid"
        UNIT[Unit Tests<br/>Individual Functions]
        INT[Integration Tests<br/>Service Interactions]
        E2E[End-to-End Tests<br/>Complete Workflows]
        PERF[Performance Tests<br/>Load & Stress]
    end
    
    subgraph "Testing Tools"
        PYTEST[pytest<br/>Python Testing]
        JEST[Jest<br/>JavaScript Testing]
        PLAYWRIGHT[Playwright<br/>E2E Testing]
        LOCUST[Locust<br/>Load Testing]
    end
    
    UNIT --> PYTEST
    INT --> PYTEST
    E2E --> PLAYWRIGHT
    PERF --> LOCUST
    
    classDef test fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef tool fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    
    class UNIT,INT,E2E,PERF test
    class PYTEST,JEST,PLAYWRIGHT,LOCUST tool
```

### Infrastructure Testing

```bash
# Test infrastructure deployment
azd provision --dry-run

# Validate Bicep templates
az bicep build --file infra/main.bicep

# Test with Azure Resource Manager What-If
az deployment group what-if \
  --resource-group "test-rg" \
  --template-file "infra/main.bicep" \
  --parameters "@infra/main.parameters.json"
```

### Application Testing

```python
# test_ai_service.py
import pytest
from services.ai_service import AIService

class TestAIService:
    def setup_method(self):
        self.ai_service = AIService()
    
    def test_generate_response(self):
        prompt = "Hello, how are you?"
        response = self.ai_service.generate_response(prompt)
        assert response is not None
        assert len(response) > 0
    
    def test_search_documents(self):
        query = "AI deployment best practices"
        results = self.ai_service.search_documents(query)
        assert isinstance(results, list)
        assert len(results) > 0
```

### Security Testing

```bash
# Network security validation
nmap -sS -O target-ip-range

# SSL/TLS certificate validation
openssl s_client -connect your-app.azurewebsites.net:443

# Authentication testing
curl -H "Authorization: Bearer invalid-token" \
     https://your-api.azure-api.net/api/chat
```

## Deployment Patterns

### GitOps Deployment Flow

```mermaid
graph TB
    subgraph "Development"
        DEV[Feature Branch]
        PR[Pull Request]
        MAIN[Main Branch]
    end
    
    subgraph "CI/CD Pipeline"
        BUILD[Build & Test]
        VALIDATE[Validate Infrastructure]
        DEPLOY_DEV[Deploy to Dev]
        DEPLOY_STAGING[Deploy to Staging]
        DEPLOY_PROD[Deploy to Production]
    end
    
    subgraph "Environments"
        ENV_DEV[Development Environment]
        ENV_STAGING[Staging Environment]
        ENV_PROD[Production Environment]
    end
    
    DEV --> PR
    PR --> BUILD
    BUILD --> VALIDATE
    VALIDATE --> DEPLOY_DEV
    DEPLOY_DEV --> ENV_DEV
    
    PR --> MAIN
    MAIN --> DEPLOY_STAGING
    DEPLOY_STAGING --> ENV_STAGING
    ENV_STAGING --> DEPLOY_PROD
    DEPLOY_PROD --> ENV_PROD
    
    classDef dev fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef pipeline fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef env fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    
    class DEV,PR,MAIN dev
    class BUILD,VALIDATE,DEPLOY_DEV,DEPLOY_STAGING,DEPLOY_PROD pipeline
    class ENV_DEV,ENV_STAGING,ENV_PROD env
```

### Environment Configuration

```yaml
# azure.yaml - Multi-environment configuration
name: ai-foundry-app
services:
  backend:
    project: ./src/backend
    language: python
    host: appservice
  frontend:
    project: ./src/frontend
    language: js
    host: staticwebapp

environments:
  development:
    location: eastus2
    parameters:
      networkIsolation: false
      vmSize: Standard_B2s
  
  staging:
    location: eastus2
    parameters:
      networkIsolation: true
      vmSize: Standard_D2s_v3
  
  production:
    location: eastus2
    parameters:
      networkIsolation: true
      vmSize: Standard_D4s_v3
      redundancy: enabled
```

### Blue-Green Deployment

```bash
# Deploy to staging slot
azd deploy --environment staging

# Test staging deployment
./scripts/test-deployment.sh staging

# Swap to production
az webapp deployment slot swap \
  --name myapp \
  --resource-group myrg \
  --slot staging
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Deployment Failures

```bash
# Check deployment status
az deployment group show \
  --resource-group "your-rg" \
  --name "main"

# View deployment logs
az monitor activity-log list \
  --resource-group "your-rg" \
  --offset 1h
```

#### 2. Network Connectivity Issues

```bash
# Test private endpoint connectivity
nslookup your-service.privatelink.openai.azure.com

# Check network security group rules
az network nsg rule list \
  --resource-group "your-rg" \
  --nsg-name "default-nsg"
```

#### 3. AI Service Authentication

```python
# Debug authentication issues
from azure.identity import DefaultAzureCredential
from azure.core.exceptions import ClientAuthenticationError

try:
    credential = DefaultAzureCredential()
    token = credential.get_token("https://cognitiveservices.azure.com/.default")
    print(f"Token acquired: {token.token[:20]}...")
except ClientAuthenticationError as e:
    print(f"Authentication failed: {e}")
```

### Monitoring and Debugging

```mermaid
graph TB
    subgraph "Monitoring Tools"
        PORTAL[Azure Portal<br/>Central Management]
        INSIGHTS[Application Insights<br/>Performance Monitoring]
        LOGS[Log Analytics<br/>Centralized Logging]
        ALERTS[Azure Alerts<br/>Proactive Monitoring]
    end
    
    subgraph "Debugging Sources"
        APP_LOGS[Application Logs]
        INFRA_LOGS[Infrastructure Logs]
        NETWORK_LOGS[Network Logs]
        SECURITY_LOGS[Security Logs]
    end
    
    APP_LOGS --> INSIGHTS
    INFRA_LOGS --> LOGS
    NETWORK_LOGS --> LOGS
    SECURITY_LOGS --> LOGS
    
    INSIGHTS --> ALERTS
    LOGS --> ALERTS
    ALERTS --> PORTAL
    
    classDef monitor fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef debug fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    
    class PORTAL,INSIGHTS,LOGS,ALERTS monitor
    class APP_LOGS,INFRA_LOGS,NETWORK_LOGS,SECURITY_LOGS debug
```

### Performance Optimization

#### 1. AI Model Performance

```python
# Implement caching for frequent requests
from functools import lru_cache
import hashlib

class AIService:
    @lru_cache(maxsize=100)
    def get_cached_response(self, prompt_hash):
        # Return cached response if available
        pass
    
    def generate_response(self, prompt):
        prompt_hash = hashlib.md5(prompt.encode()).hexdigest()
        return self.get_cached_response(prompt_hash)
```

#### 2. Database Optimization

```python
# Optimize Cosmos DB queries
from azure.cosmos import CosmosClient

def get_conversation_history(user_id, limit=10):
    query = """
    SELECT TOP @limit c.id, c.message, c.timestamp
    FROM conversations c
    WHERE c.userId = @userId
    ORDER BY c.timestamp DESC
    """
    
    parameters = [
        {"name": "@userId", "value": user_id},
        {"name": "@limit", "value": limit}
    ]
    
    return list(container.query_items(
        query=query,
        parameters=parameters,
        enable_cross_partition_query=True
    ))
```

### Best Practices for Development

1. **Security First**: Always use managed identities and Key Vault for secrets
2. **Monitor Everything**: Implement comprehensive logging and monitoring
3. **Test Thoroughly**: Use automated testing at all levels
4. **Document Changes**: Keep infrastructure and application documentation updated
5. **Version Control**: Use semantic versioning for releases
6. **Environment Parity**: Keep development, staging, and production environments consistent

---

## Next Steps

1. **Start Development**: Use the provided templates and examples
2. **Customize Infrastructure**: Modify Bicep templates for your requirements
3. **Implement Monitoring**: Set up comprehensive monitoring and alerting
4. **Security Review**: Conduct security assessment and implement hardening
5. **Performance Testing**: Test under expected load conditions
6. **Production Deployment**: Follow deployment best practices for production

For additional support, refer to the [Technical Architecture Documentation](TECHNICAL_ARCHITECTURE.md) and [community resources](../README.md#community--support).