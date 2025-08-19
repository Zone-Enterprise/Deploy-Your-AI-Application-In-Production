# Technical Architecture Documentation

## Table of Contents
1. [System Overview](#system-overview)
2. [Network Architecture](#network-architecture)
3. [Data Flow](#data-flow)
4. [Deployment Architecture](#deployment-architecture)
5. [Service Dependencies](#service-dependencies)
6. [Security Architecture](#security-architecture)
7. [Monitoring and Observability](#monitoring-and-observability)
8. [Infrastructure Components](#infrastructure-components)

## System Overview

This solution provides a production-ready, secure deployment of Azure AI Foundry with comprehensive network isolation and monitoring capabilities.

```mermaid
graph TB
    subgraph "Azure Subscription"
        subgraph "Resource Group"
            subgraph "Virtual Network (10.0.0.0/16)"
                subgraph "Default Subnet (10.0.0.0/24)"
                    VM[Jump Box VM]
                    PE1[Private Endpoints]
                end
                subgraph "Bastion Subnet (10.0.1.0/27)"
                    BAS[Azure Bastion]
                end
                subgraph "Web Apps Subnet (10.0.2.0/24)"
                    APP[Sample App Service]
                end
            end
            
            subgraph "AI Services"
                AIF[AI Foundry Hub]
                AIP[AI Foundry Project]
                AOI[Azure OpenAI]
            end
            
            subgraph "Data Services"
                ST[Storage Account]
                KV[Key Vault]
                ACR[Container Registry]
                COS[Cosmos DB]
                SQL[SQL Server]
                AIS[AI Search]
            end
            
            subgraph "Management Services"
                APIM[API Management]
                LAW[Log Analytics]
                AI[Application Insights]
            end
        end
    end
    
    USER[User] --> BAS
    BAS --> VM
    VM --> PE1
    PE1 --> AIF
    PE1 --> ST
    PE1 --> KV
    PE1 --> ACR
    AIF --> AIP
    AIP --> AOI
    APP --> APIM
    APIM --> AIP
    
    classDef azure fill:#0078d4,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef network fill:#7fba00,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef ai fill:#ff6900,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef data fill:#00bcf2,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef mgmt fill:#a4c639,stroke:#ffffff,stroke-width:2px,color:#ffffff
    
    class AIF,AIP,AOI ai
    class ST,KV,ACR,COS,SQL,AIS data
    class APIM,LAW,AI mgmt
    class VM,BAS,PE1 network
```

## Network Architecture

The solution implements a hub-and-spoke network architecture with complete network isolation using private endpoints.

```mermaid
graph TB
    subgraph "Internet"
        USER[User/Developer]
        INET[Internet Traffic]
    end
    
    subgraph "Azure Virtual Network (10.0.0.0/16)"
        subgraph "AzureBastionSubnet (10.0.1.0/27)"
            BAS[Azure Bastion<br/>Secure RDP Gateway]
        end
        
        subgraph "Default Subnet (10.0.0.0/24)"
            VM[Jump Box VM<br/>Windows Server]
            PE_GROUP[Private Endpoints]
            
            subgraph "Private Endpoints"
                PE_KV[Key Vault PE]
                PE_ST[Storage PE]
                PE_ACR[Container Registry PE]
                PE_AIF[AI Foundry Hub PE]
                PE_AIS[AI Search PE]
                PE_COS[Cosmos DB PE]
                PE_SQL[SQL Server PE]
            end
        end
        
        subgraph "Web Apps Subnet (10.0.2.0/24)"
            WEB[App Service<br/>Sample Application]
        end
        
        subgraph "NAT Gateway"
            NAT[NAT Gateway<br/>Outbound Internet]
        end
    end
    
    subgraph "Azure Services (Private)"
        KV[Key Vault]
        ST[Storage Account]
        ACR[Container Registry]
        AIF[AI Foundry Hub]
        AIS[AI Search]
        COS[Cosmos DB]
        SQL[SQL Server]
        AOI[Azure OpenAI]
    end
    
    subgraph "Network Security"
        NSG1[Bastion NSG]
        NSG2[Default Subnet NSG]
        NSG3[Web Apps NSG]
        PDNS[Private DNS Zones]
    end
    
    %% Connections
    USER -->|HTTPS:443| BAS
    BAS -->|RDP:3389| VM
    VM --> PE_GROUP
    PE_KV -.->|Private Connection| KV
    PE_ST -.->|Private Connection| ST
    PE_ACR -.->|Private Connection| ACR
    PE_AIF -.->|Private Connection| AIF
    PE_AIS -.->|Private Connection| AIS
    PE_COS -.->|Private Connection| COS
    PE_SQL -.->|Private Connection| SQL
    
    VM -->|Outbound| NAT
    NAT -->|Internet Access| INET
    
    WEB --> PE_GROUP
    AIF --> AOI
    
    NSG1 -.->|Protects| BAS
    NSG2 -.->|Protects| VM
    NSG3 -.->|Protects| WEB
    PDNS -.->|DNS Resolution| PE_GROUP
    
    classDef user fill:#e1f5fe,stroke:#0277bd,stroke-width:2px
    classDef network fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef service fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef security fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef private fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    
    class USER user
    class BAS,VM,WEB,NAT network
    class KV,ST,ACR,AIF,AIS,COS,SQL,AOI service
    class NSG1,NSG2,NSG3,PDNS security
    class PE_KV,PE_ST,PE_ACR,PE_AIF,PE_AIS,PE_COS,PE_SQL private
```

## Data Flow

### AI Application Data Flow

```mermaid
sequenceDiagram
    participant U as User
    participant B as Azure Bastion
    participant VM as Jump Box VM
    participant APP as Sample App
    participant APIM as API Management
    participant AIP as AI Project
    participant AOI as Azure OpenAI
    participant AIS as AI Search
    participant COS as Cosmos DB
    participant ST as Storage Account
    
    U->>B: Connect via HTTPS
    B->>VM: Secure RDP connection
    VM->>APP: Access web application
    
    Note over APP,AIP: AI Application Flow
    APP->>APIM: API Request
    APIM->>AIP: Forward to AI Project
    AIP->>AOI: Generate AI Response
    AIP->>AIS: Search relevant data
    AIS-->>AIP: Return search results
    AOI-->>AIP: Return AI response
    AIP-->>APIM: Combined response
    APIM-->>APP: API Response
    
    Note over APP,ST: Data Persistence
    APP->>COS: Store conversation
    APP->>ST: Store documents/media
    
    Note over VM,ST: Management Operations
    VM->>AIP: Model deployment
    VM->>ST: Upload training data
    VM->>AIS: Index management
```

### Authentication and Authorization Flow

```mermaid
sequenceDiagram
    participant U as User/App
    participant AAD as Azure AD
    participant APIM as API Management
    participant AIP as AI Project
    participant KV as Key Vault
    participant MI as Managed Identity
    
    U->>AAD: Authenticate
    AAD-->>U: JWT Token
    
    U->>APIM: Request with JWT
    APIM->>AAD: Validate token
    AAD-->>APIM: Token valid
    
    APIM->>AIP: Forward request
    AIP->>MI: Use managed identity
    MI->>KV: Retrieve secrets
    KV-->>MI: Return secrets
    MI-->>AIP: Authenticated access
    
    AIP->>AIP: Process request
    AIP-->>APIM: Return response
    APIM-->>U: Final response
```

## Deployment Architecture

### Infrastructure Deployment Flow

```mermaid
graph TD
    subgraph "Deployment Process"
        START[Start Deployment] --> AZD[Azure Developer CLI]
        AZD --> BICEP[Bicep Templates]
        BICEP --> RG[Create Resource Group]
        
        RG --> NET[Deploy Network Infrastructure]
        NET --> VM_DEPLOY[Deploy Jump Box VM]
        NET --> BAS_DEPLOY[Deploy Azure Bastion]
        
        VM_DEPLOY --> CORE[Deploy Core Services]
        BAS_DEPLOY --> CORE
        
        CORE --> KV_DEPLOY[Key Vault]
        CORE --> ST_DEPLOY[Storage Account]
        CORE --> ACR_DEPLOY[Container Registry]
        CORE --> LAW_DEPLOY[Log Analytics]
        
        KV_DEPLOY --> AI_DEPLOY[Deploy AI Services]
        ST_DEPLOY --> AI_DEPLOY
        ACR_DEPLOY --> AI_DEPLOY
        LAW_DEPLOY --> AI_DEPLOY
        
        AI_DEPLOY --> AIF_DEPLOY[AI Foundry Hub]
        AI_DEPLOY --> AOI_DEPLOY[Azure OpenAI]
        
        AIF_DEPLOY --> AIP_DEPLOY[AI Foundry Project]
        AOI_DEPLOY --> AIP_DEPLOY
        
        AIP_DEPLOY --> OPT[Optional Services]
        OPT --> AIS_OPT[AI Search]
        OPT --> COS_OPT[Cosmos DB]
        OPT --> SQL_OPT[SQL Server]
        OPT --> APIM_OPT[API Management]
        
        AIS_OPT --> PE[Create Private Endpoints]
        COS_OPT --> PE
        SQL_OPT --> PE
        APIM_OPT --> PE
        
        PE --> CONFIG[Post-Deployment Configuration]
        CONFIG --> COMPLETE[Deployment Complete]
    end
    
    subgraph "Deployment Options"
        CODESPACE[GitHub Codespaces]
        LOCAL[Local Environment]
        ACTIONS[GitHub Actions]
    end
    
    CODESPACE --> AZD
    LOCAL --> AZD
    ACTIONS --> AZD
    
    classDef deploy fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef network fill:#f1f8e9,stroke:#388e3c,stroke-width:2px
    classDef service fill:#fff8e1,stroke:#f57c00,stroke-width:2px
    classDef option fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    
    class START,AZD,BICEP,COMPLETE deploy
    class NET,VM_DEPLOY,BAS_DEPLOY,PE network
    class CORE,KV_DEPLOY,ST_DEPLOY,ACR_DEPLOY,LAW_DEPLOY,AI_DEPLOY,AIF_DEPLOY,AOI_DEPLOY,AIP_DEPLOY service
    class OPT,AIS_OPT,COS_OPT,SQL_OPT,APIM_OPT,CODESPACE,LOCAL,ACTIONS option
```

## Service Dependencies

### Core Service Dependencies

```mermaid
graph TB
    subgraph "Foundation Layer"
        AAD[Azure Active Directory]
        SUB[Azure Subscription]
        RG[Resource Group]
        VNET[Virtual Network]
    end
    
    subgraph "Network Layer"
        NSG[Network Security Groups]
        PE[Private Endpoints]
        PDNS[Private DNS Zones]
        BAS[Azure Bastion]
        NAT[NAT Gateway]
    end
    
    subgraph "Security Layer"
        KV[Key Vault]
        MI[Managed Identities]
        RBAC[Role-Based Access Control]
    end
    
    subgraph "Monitoring Layer"
        LAW[Log Analytics Workspace]
        AI_INSIGHTS[Application Insights]
        DIAG[Diagnostic Settings]
    end
    
    subgraph "Storage Layer"
        ST[Storage Account]
        ACR[Container Registry]
        COS[Cosmos DB]
        SQL[SQL Server]
    end
    
    subgraph "AI Services Layer"
        AIF[AI Foundry Hub]
        AIP[AI Foundry Project]
        AOI[Azure OpenAI]
        AIS[AI Search]
        CS[Content Safety]
        VISION[AI Vision]
        SPEECH[AI Speech]
    end
    
    subgraph "Application Layer"
        APIM[API Management]
        APP[App Service]
        VM[Jump Box VM]
    end
    
    %% Dependencies
    SUB --> RG
    RG --> VNET
    VNET --> NSG
    VNET --> PE
    VNET --> PDNS
    VNET --> BAS
    VNET --> NAT
    
    AAD --> MI
    AAD --> RBAC
    
    VNET --> KV
    MI --> KV
    
    RG --> LAW
    LAW --> AI_INSIGHTS
    LAW --> DIAG
    
    KV --> ST
    VNET --> ST
    PE --> ST
    
    ST --> ACR
    VNET --> ACR
    PE --> ACR
    
    ST --> AIF
    KV --> AIF
    MI --> AIF
    VNET --> AIF
    PE --> AIF
    LAW --> AIF
    
    AIF --> AIP
    AIF --> AOI
    
    AIP --> AIS
    AIP --> COS
    AIP --> SQL
    
    AIP --> APIM
    VNET --> APIM
    
    APIM --> APP
    VNET --> APP
    
    VNET --> VM
    BAS --> VM
    
    classDef foundation fill:#e8eaf6,stroke:#3f51b5,stroke-width:2px
    classDef network fill:#e0f2f1,stroke:#00695c,stroke-width:2px
    classDef security fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef monitoring fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef storage fill:#e1f5fe,stroke:#0277bd,stroke-width:2px
    classDef ai fill:#ffebee,stroke:#c62828,stroke-width:2px
    classDef app fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    
    class AAD,SUB,RG,VNET foundation
    class NSG,PE,PDNS,BAS,NAT network
    class KV,MI,RBAC security
    class LAW,AI_INSIGHTS,DIAG monitoring
    class ST,ACR,COS,SQL storage
    class AIF,AIP,AOI,AIS,CS,VISION,SPEECH ai
    class APIM,APP,VM app
```

## Security Architecture

### Zero Trust Network Architecture

```mermaid
graph TB
    subgraph "Identity & Access Management"
        AAD[Azure Active Directory]
        MFA[Multi-Factor Authentication]
        CA[Conditional Access]
        PIM[Privileged Identity Management]
    end
    
    subgraph "Network Security"
        subgraph "Perimeter Defense"
            BAS[Azure Bastion]
            NSG[Network Security Groups]
            RULES[Security Rules]
        end
        
        subgraph "Internal Segmentation"
            VNET[Virtual Network]
            SUBNET[Subnets]
            PE[Private Endpoints]
            PDNS[Private DNS]
        end
    end
    
    subgraph "Data Protection"
        subgraph "Encryption"
            KV[Key Vault]
            CMK[Customer Managed Keys]
            TDE[Transparent Data Encryption]
            HTTPS[HTTPS/TLS]
        end
        
        subgraph "Access Control"
            RBAC[Role-Based Access Control]
            MI[Managed Identities]
            SP[Service Principals]
        end
    end
    
    subgraph "Monitoring & Compliance"
        subgraph "Security Monitoring"
            DEFENDER[Microsoft Defender]
            SENTINEL[Azure Sentinel]
            LAW[Log Analytics]
        end
        
        subgraph "Compliance"
            POLICY[Azure Policy]
            BLUEPRINT[Azure Blueprints]
            COMP[Compliance Dashboard]
        end
    end
    
    %% Security Flow
    AAD --> MFA
    MFA --> CA
    CA --> PIM
    
    BAS --> NSG
    NSG --> RULES
    VNET --> SUBNET
    SUBNET --> PE
    PE --> PDNS
    
    KV --> CMK
    KV --> TDE
    RBAC --> MI
    MI --> SP
    
    LAW --> DEFENDER
    LAW --> SENTINEL
    POLICY --> BLUEPRINT
    BLUEPRINT --> COMP
    
    %% Cross-layer connections
    AAD -.-> RBAC
    KV -.-> PE
    MI -.-> KV
    LAW -.-> NSG
    
    classDef identity fill:#e8eaf6,stroke:#3f51b5,stroke-width:3px
    classDef network fill:#e0f2f1,stroke:#00695c,stroke-width:3px
    classDef data fill:#fff3e0,stroke:#ef6c00,stroke-width:3px
    classDef monitoring fill:#f3e5f5,stroke:#7b1fa2,stroke-width:3px
    
    class AAD,MFA,CA,PIM identity
    class BAS,NSG,RULES,VNET,SUBNET,PE,PDNS network
    class KV,CMK,TDE,HTTPS,RBAC,MI,SP data
    class DEFENDER,SENTINEL,LAW,POLICY,BLUEPRINT,COMP monitoring
```

## Monitoring and Observability

### Monitoring Architecture

```mermaid
graph TB
    subgraph "Data Sources"
        VM[Virtual Machines]
        APP[Applications]
        AIF[AI Foundry]
        NET[Network]
        SEC[Security Events]
    end
    
    subgraph "Collection Layer"
        AGENT[Azure Monitor Agent]
        DIAG[Diagnostic Settings]
        METRICS[Platform Metrics]
        LOGS[Platform Logs]
    end
    
    subgraph "Storage & Processing"
        LAW[Log Analytics Workspace]
        STOR[Azure Storage]
        EH[Event Hubs]
    end
    
    subgraph "Analytics & Intelligence"
        KQL[Kusto Queries]
        WORKBOOK[Azure Workbooks]
        INSIGHTS[Application Insights]
        ML[Machine Learning]
    end
    
    subgraph "Alerting & Response"
        ALERTS[Azure Alerts]
        AG[Action Groups]
        EMAIL[Email Notifications]
        SMS[SMS Notifications]
        WEBHOOK[Webhooks]
        LOGIC[Logic Apps]
    end
    
    subgraph "Visualization"
        PORTAL[Azure Portal]
        GRAFANA[Grafana]
        POWERBI[Power BI]
        CUSTOM[Custom Dashboards]
    end
    
    %% Data Flow
    VM --> AGENT
    APP --> DIAG
    AIF --> DIAG
    NET --> METRICS
    SEC --> LOGS
    
    AGENT --> LAW
    DIAG --> LAW
    METRICS --> LAW
    LOGS --> LAW
    
    LAW --> STOR
    LAW --> EH
    
    LAW --> KQL
    LAW --> WORKBOOK
    LAW --> INSIGHTS
    LAW --> ML
    
    KQL --> ALERTS
    WORKBOOK --> ALERTS
    INSIGHTS --> ALERTS
    ML --> ALERTS
    
    ALERTS --> AG
    AG --> EMAIL
    AG --> SMS
    AG --> WEBHOOK
    AG --> LOGIC
    
    LAW --> PORTAL
    LAW --> GRAFANA
    LAW --> POWERBI
    LAW --> CUSTOM
    
    classDef source fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef collection fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef storage fill:#fff8e1,stroke:#f57c00,stroke-width:2px
    classDef analytics fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef alerting fill:#ffebee,stroke:#d32f2f,stroke-width:2px
    classDef visual fill:#e0f2f1,stroke:#00695c,stroke-width:2px
    
    class VM,APP,AIF,NET,SEC source
    class AGENT,DIAG,METRICS,LOGS collection
    class LAW,STOR,EH storage
    class KQL,WORKBOOK,INSIGHTS,ML analytics
    class ALERTS,AG,EMAIL,SMS,WEBHOOK,LOGIC alerting
    class PORTAL,GRAFANA,POWERBI,CUSTOM visual
```

## Infrastructure Components

### Core Infrastructure Components

| Component | Purpose | Configuration |
|-----------|---------|---------------|
| **Virtual Network** | Network isolation and segmentation | 10.0.0.0/16 address space |
| **Azure Bastion** | Secure RDP/SSH access | Deployed in dedicated subnet |
| **Jump Box VM** | Secure access to private resources | Windows Server with tools |
| **Private Endpoints** | Secure connectivity to PaaS services | One per service in default subnet |
| **Network Security Groups** | Network traffic filtering | Applied to each subnet |
| **NAT Gateway** | Outbound internet connectivity | Attached to default subnet |

### AI and Data Services

| Component | Purpose | Configuration |
|-----------|---------|---------------|
| **AI Foundry Hub** | Central hub for AI projects | Network isolated with private endpoint |
| **AI Foundry Project** | Individual AI project workspace | Connected to hub with service connections |
| **Azure OpenAI** | Large language models | GPT-4o and text-embedding-3-small |
| **Azure AI Search** | Cognitive search capabilities | Private endpoint enabled |
| **Cosmos DB** | NoSQL database | Optional, private endpoint enabled |
| **SQL Server** | Relational database | Optional, private endpoint enabled |

### Security and Management

| Component | Purpose | Configuration |
|-----------|---------|---------------|
| **Key Vault** | Secrets and key management | Private endpoint, RBAC enabled |
| **Container Registry** | Container image storage | Premium tier, private endpoint |
| **Storage Account** | File and blob storage | Private endpoint, encryption enabled |
| **Log Analytics** | Centralized logging | All services configured for logging |
| **Application Insights** | Application performance monitoring | Connected to Log Analytics |

### Optional Components

| Component | Purpose | When Deployed |
|-----------|---------|---------------|
| **API Management** | API gateway and management | When apiManagementEnabled = true |
| **App Service** | Sample application hosting | When appSampleEnabled = true |
| **Content Safety** | AI content moderation | When contentSafetyEnabled = true |
| **AI Vision** | Computer vision services | When visionEnabled = true |
| **AI Speech** | Speech services | When speechEnabled = true |
| **Translator** | Translation services | When translatorEnabled = true |
| **Document Intelligence** | Document processing | When documentIntelligenceEnabled = true |

---

## Next Steps

1. Review the [deployment guide](../README.md) for step-by-step instructions
2. Check the [quota requirements](quota_check.md) before deployment
3. Configure [additional services](add_additional_services.md) as needed
4. Follow [post-deployment steps](post_deployment_steps.md) for verification
5. Set up [monitoring and alerting](monitoring_setup.md) for production use

For detailed configuration options and customization, refer to the individual service documentation linked throughout this guide.