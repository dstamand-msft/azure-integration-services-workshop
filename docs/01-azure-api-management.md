# Azure API Management (APIM)

## Overview

Azure API Management (APIM) is a hybrid, multicloud management platform for APIs across all environments. As a platform-as-a-service, API Management supports the complete API lifecycle from design, development, publishing, operations, to retirement.

APIM helps organizations publish APIs to external, partner, and internal developers to unlock the potential of their data and services. It provides tools for securing, managing, and scaling API calls while offering a unified management experience and full observability.

## Key Components

```mermaid
graph TB
    subgraph "Azure API Management"
        GW[API Gateway]
        DP[Developer Portal]
        MP[Management Plane]
        
        subgraph "Gateway Features"
            POL[Policies]
            CACHE[Caching]
            TRANS[Transformation]
            AUTH[Authentication]
        end
        
        GW --> POL
        GW --> CACHE
        GW --> TRANS
        GW --> AUTH
    end
    
    CLIENT[Client Apps] --> GW
    GW --> BACKEND[Backend APIs]
    DEV[Developers] --> DP
    ADMIN[Administrators] --> MP
    
    style GW fill:#0078D4,color:#fff
    style DP fill:#50E6FF
    style MP fill:#50E6FF
```

### 1. API Gateway
- Routes API calls to backends
- Verifies API keys and credentials
- Enforces usage quotas and rate limits
- Transforms requests and responses
- Caches responses
- Logs for analytics

### 2. Developer Portal
- Auto-generated, customizable portal
- API documentation and interactive console
- Self-service subscription management
- Analytics dashboards

### 3. Management Plane
- API definition and configuration
- User and group management
- Policy configuration
- Analytics and reporting

## Service Tiers

| Tier | Description | Use Case | Key Features |
|------|-------------|----------|--------------|
| **Consumption** | Serverless, pay-per-execution | Variable traffic, microservices | Auto-scaling, no idle cost |
| **Developer** | Non-production | Development/testing | No SLA, lower cost |
| **Basic** | Entry-level production | Small workloads | SLA, limited scale |
| **Basic v2** | Development and testing | Dev/test with SLA | Fast provisioning, SLA |
| **Standard** | Production | Medium workloads | Azure AD, higher limits |
| **Standard v2** | Production | VNet integration needed | Network isolation, fast scaling |
| **Premium** | Enterprise | Multi-region, VNET | Multi-region, availability zones |
| **Premium v2** | Enterprise | Full VNet isolation | Workspaces, 30 units max |

### V2 Tiers Benefits
- **Faster deployment** - Production-ready in minutes
- **Simplified networking** - VNet integration and private endpoints
- **Better scaling** - Up to 10 units (Basic/Standard v2) or 30 units (Premium v2)
- **Developer portal options** - Enable when needed

## How APIM Works

```mermaid
sequenceDiagram
    participant C as Client
    participant G as API Gateway
    participant P as Policies
    participant B as Backend API
    
    C->>G: API Request
    G->>P: Apply Inbound Policies
    Note over P: Validate JWT<br/>Check rate limit<br/>Transform request
    P->>B: Forward Request
    B->>P: Response
    P->>P: Apply Outbound Policies
    Note over P: Transform response<br/>Cache response
    P->>G: Processed Response
    G->>C: API Response
```

## Policies

Policies are a powerful capability that allows publishers to change API behavior through configuration. They execute sequentially on the request or response.

### Policy Scopes

```mermaid
graph TB
    subgraph "Policy Hierarchy"
        GL[Global Policies]
        PR[Product Policies]
        AP[API Policies]
        OP[Operation Policies]
        
        GL --> PR --> AP --> OP
    end
    
    style GL fill:#0078D4,color:#fff
    style PR fill:#2B88D8,color:#fff
    style AP fill:#50E6FF
    style OP fill:#9EE7FF
```

### Common Policy Categories

| Category | Policies | Description |
|----------|----------|-------------|
| **Access Restriction** | IP filtering, validate JWT, check HTTP header | Control access to APIs |
| **Authentication** | Basic auth, certificate auth, managed identity | Authenticate to backends |
| **Caching** | Store/retrieve from cache | Reduce backend load |
| **Transformation** | Set/rewrite headers, convert XML to JSON | Modify requests/responses |
| **Rate Limiting** | Rate limit by key/subscription, quota | Protect backends |
| **Advanced** | Mock responses, retry, send requests | Complex scenarios |

### Policy Example: Rate Limiting

```xml
<policies>
    <inbound>
        <base />
        <!-- Limit to 100 calls per minute per subscription -->
        <rate-limit calls="100" renewal-period="60" />
        <!-- Set quota of 10000 calls per week -->
        <quota calls="10000" renewal-period="604800" />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

### Policy Example: JWT Validation

```xml
<policies>
    <inbound>
        <validate-jwt header-name="Authorization" 
                      failed-validation-httpcode="401"
                      failed-validation-error-message="Unauthorized">
            <openid-config url="https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration" />
            <audiences>
                <audience>api://your-api-id</audience>
            </audiences>
            <required-claims>
                <claim name="roles" match="any">
                    <value>API.Reader</value>
                    <value>API.Writer</value>
                </claim>
            </required-claims>
        </validate-jwt>
        <base />
    </inbound>
</policies>
```

## Authentication & Security

### Authentication Methods

```mermaid
graph LR
    subgraph "Client to APIM"
        A1[Subscription Key]
        A2[OAuth 2.0 / JWT]
        A3[Client Certificate]
        A4[IP Filtering]
    end
    
    subgraph "APIM to Backend"
        B1[Basic Auth]
        B2[Client Certificate]
        B3[Managed Identity]
        B4[OAuth 2.0]
    end
    
    A1 --> GW[API Gateway]
    A2 --> GW
    A3 --> GW
    A4 --> GW
    
    GW --> B1
    GW --> B2
    GW --> B3
    GW --> B4
    
    style GW fill:#0078D4,color:#fff
```

### Security Best Practices

1. **Use OAuth 2.0/OpenID Connect** for API authorization
2. **Enable HTTPS only** - disable HTTP
3. **Use managed identities** to authenticate to backends
4. **Store secrets in Key Vault** - reference certificates and secrets
5. **Implement rate limiting** to prevent abuse
6. **Use VNet integration** for network isolation
7. **Enable diagnostic logging** to Azure Monitor

## Architecture Patterns

### Pattern 1: API Gateway for Microservices

```mermaid
graph TB
    subgraph "Clients"
        WEB[Web App]
        MOB[Mobile App]
        IOT[IoT Devices]
    end
    
    subgraph "Azure API Management"
        GW[Gateway]
        
        subgraph "APIs"
            API1[Orders API]
            API2[Products API]
            API3[Users API]
        end
    end
    
    subgraph "Microservices"
        MS1[Orders Service]
        MS2[Products Service]
        MS3[Users Service]
    end
    
    WEB --> GW
    MOB --> GW
    IOT --> GW
    
    GW --> API1 --> MS1
    GW --> API2 --> MS2
    GW --> API3 --> MS3
    
    style GW fill:#0078D4,color:#fff
```

### Pattern 2: API Façade for Legacy Systems

```mermaid
graph LR
    subgraph "Modern Clients"
        C1[REST Client]
        C2[GraphQL Client]
    end
    
    subgraph "API Management"
        APIM[Gateway]
        subgraph "Transformation"
            T1[REST to SOAP]
            T2[JSON to XML]
        end
    end
    
    subgraph "Legacy Systems"
        L1[SOAP Service]
        L2[XML Service]
    end
    
    C1 --> APIM
    C2 --> APIM
    APIM --> T1 --> L1
    APIM --> T2 --> L2
    
    style APIM fill:#0078D4,color:#fff
```

### Pattern 3: Multi-Region Deployment

```mermaid
graph TB
    subgraph "Global"
        TM[Traffic Manager]
    end
    
    subgraph "Region 1 - West US"
        APIM1[APIM Primary]
        BE1[Backend Services]
        APIM1 --> BE1
    end
    
    subgraph "Region 2 - East US"
        APIM2[APIM Secondary]
        BE2[Backend Services]
        APIM2 --> BE2
    end
    
    TM --> APIM1
    TM --> APIM2
    
    style APIM1 fill:#0078D4,color:#fff
    style APIM2 fill:#0078D4,color:#fff
    style TM fill:#50E6FF
```

## Main Use Cases

### 1. API Monetization
- Publish APIs to partners and developers
- Implement usage-based billing
- Track API consumption

### 2. Internal API Gateway
- Standardize internal API access
- Implement consistent security
- Monitor API usage

### 3. Legacy Modernization
- Expose legacy services as REST APIs
- Transform protocols and formats
- Gradual migration path

### 4. Multi-Cloud API Management
- Self-hosted gateway for Kubernetes
- On-premises API management
- Hybrid cloud scenarios

## Best Practices

### Design

| Practice | Description |
|----------|-------------|
| **Version your APIs** | Use URL path or header versioning |
| **Use products** | Group APIs logically for subscription management |
| **Design for security** | Apply defense in depth |
| **Plan for scale** | Choose appropriate tier and configure auto-scaling |

### Operations

| Practice | Description |
|----------|-------------|
| **Monitor performance** | Use Azure Monitor and Application Insights |
| **Set up alerts** | For errors, latency, and capacity |
| **Backup configurations** | Regular exports and IaC |
| **Test policies** | Use test console and automated tests |

### Security

| Practice | Description |
|----------|-------------|
| **Use OAuth 2.0** | For API authorization |
| **Store secrets in Key Vault** | Never hardcode credentials |
| **Enable WAF** | Use Application Gateway with WAF |
| **Regular audits** | Review access and permissions |

## Integration with Other Services

```mermaid
graph TB
    APIM[API Management]
    
    subgraph "Identity"
        AAD[Azure AD / Entra ID]
        B2C[Azure AD B2C]
    end
    
    subgraph "Monitoring"
        AM[Azure Monitor]
        AI[Application Insights]
        LA[Log Analytics]
    end
    
    subgraph "Security"
        KV[Key Vault]
        AG[App Gateway + WAF]
    end
    
    subgraph "Integration"
        LA2[Logic Apps]
        AF[Azure Functions]
        SB[Service Bus]
    end
    
    AAD --> APIM
    B2C --> APIM
    APIM --> AM
    APIM --> AI
    AM --> LA
    KV --> APIM
    AG --> APIM
    APIM --> LA2
    APIM --> AF
    APIM --> SB
    
    style APIM fill:#0078D4,color:#fff
```

## Pricing Considerations

| Factor | Impact |
|--------|--------|
| **Tier selection** | Fixed monthly cost (Classic) vs. pay-per-call (Consumption) |
| **Scale units** | Additional cost per unit |
| **Self-hosted gateways** | Per gateway cost |
| **Workspaces** | Additional workspace gateway costs |
| **Data transfer** | Outbound data costs |

## Hands-On Lab Ideas

1. **Create your first API** - Import an OpenAPI specification
2. **Implement authentication** - Add OAuth 2.0 with Azure AD
3. **Configure policies** - Rate limiting and caching
4. **Set up monitoring** - Azure Monitor and alerts
5. **Deploy multi-region** - Traffic Manager integration

---

## References

- [Azure API Management Documentation](https://learn.microsoft.com/en-us/azure/api-management/)
- [API Management Key Concepts](https://learn.microsoft.com/en-us/azure/api-management/api-management-key-concepts)
- [API Management Policies Reference](https://learn.microsoft.com/en-us/azure/api-management/api-management-policies)
- [V2 Service Tiers](https://learn.microsoft.com/en-us/azure/api-management/v2-service-tiers-overview)
- [Authentication and Authorization](https://learn.microsoft.com/en-us/azure/api-management/authentication-authorization-overview)
- [API Management Best Practices](https://learn.microsoft.com/en-us/azure/well-architected/service-guides/azure-api-management)
- [Protect APIs with Application Gateway](https://learn.microsoft.com/en-us/azure/architecture/web-apps/api-management/architectures/protect-apis)
