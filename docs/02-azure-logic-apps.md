# Azure Logic Apps

## Overview

Azure Logic Apps is a cloud-based platform for creating and running automated workflows that integrate apps, data, services, and systems. It enables developers and IT professionals to automate business processes and integrate systems without writing code, using a visual designer with hundreds of pre-built connectors.

Logic Apps supports both low-code and pro-code approaches, making it accessible for citizen developers while providing the power needed for enterprise integration scenarios.

## Hosting Options

Azure Logic Apps offers two main hosting options:

```mermaid
graph TB
    subgraph "Azure Logic Apps"
        subgraph "Consumption (Multitenant)"
            C1[Single Workflow per App]
            C2[Pay per Execution]
            C3[Managed Connectors]
        end
        
        subgraph "Standard (Single-tenant)"
            S1[Multiple Workflows per App]
            S2[Fixed Pricing + Execution]
            S3[Built-in + Managed Connectors]
            S4[VNet Integration]
            S5[Local Development]
        end
    end
    
    style C1 fill:#50E6FF
    style C2 fill:#50E6FF
    style C3 fill:#50E6FF
    style S1 fill:#0078D4,color:#fff
    style S2 fill:#0078D4,color:#fff
    style S3 fill:#0078D4,color:#fff
    style S4 fill:#0078D4,color:#fff
    style S5 fill:#0078D4,color:#fff
```

### Comparison: Consumption vs Standard

| Feature | Consumption | Standard |
|---------|-------------|----------|
| **Workflows per app** | 1 | Multiple |
| **Pricing model** | Pay per execution | Fixed + execution |
| **Environment** | Multitenant | Single-tenant |
| **VNet integration** | Limited | Full support |
| **Local development** | Limited | VS Code support |
| **Deployment** | Azure Portal, ARM | CI/CD, containers |
| **Stateful/Stateless** | Stateful only | Both |
| **Performance** | Good | Better (proximity) |
| **Built-in connectors** | Limited | Extensive |

## Key Concepts

### Workflow Structure

```mermaid
flowchart TB
    subgraph "Logic App Workflow"
        T[Trigger] --> A1[Action 1]
        A1 --> C{Condition}
        C -->|Yes| A2[Action 2]
        C -->|No| A3[Action 3]
        A2 --> A4[Action 4]
        A3 --> A4
    end
    
    style T fill:#FF6B6B,color:#fff
    style A1 fill:#0078D4,color:#fff
    style A2 fill:#0078D4,color:#fff
    style A3 fill:#0078D4,color:#fff
    style A4 fill:#0078D4,color:#fff
    style C fill:#FFE66D
```

### Triggers

Triggers start your workflow and come in several types:

| Trigger Type | Description | Example |
|--------------|-------------|---------|
| **Recurrence** | Time-based schedule | Run every hour |
| **Request** | HTTP endpoint | Webhook receiver |
| **Polling** | Check for changes | New email arrives |
| **Push** | Event-based | Service Bus message |
| **Manual** | On-demand | Test or admin trigger |

### Actions

Actions perform tasks in your workflow:

| Action Type | Description | Examples |
|-------------|-------------|----------|
| **Built-in** | Native to Logic Apps | HTTP, Condition, Loop |
| **Managed** | Microsoft-hosted connectors | Office 365, SQL |
| **Custom** | Your own APIs | Custom connector |
| **Control** | Flow control | Switch, ForEach, Scope |

### Connectors

```mermaid
graph TB
    subgraph "Connector Types"
        subgraph "Built-in"
            BI1[HTTP/HTTPS]
            BI2[Schedule]
            BI3[Request/Response]
            BI4[Data Operations]
        end
        
        subgraph "Managed - Standard"
            MS1[Office 365]
            MS2[SQL Server]
            MS3[Azure Services]
            MS4[Social Media]
        end
        
        subgraph "Managed - Enterprise"
            ME1[SAP]
            ME2[IBM MQ]
            ME3[EDIFACT/X12]
        end
        
        subgraph "Custom"
            CU1[Custom APIs]
            CU2[On-premises]
        end
    end
    
    style BI1 fill:#50E6FF
    style BI2 fill:#50E6FF
    style BI3 fill:#50E6FF
    style BI4 fill:#50E6FF
    style MS1 fill:#0078D4,color:#fff
    style MS2 fill:#0078D4,color:#fff
    style MS3 fill:#0078D4,color:#fff
    style MS4 fill:#0078D4,color:#fff
    style ME1 fill:#7719AA,color:#fff
    style ME2 fill:#7719AA,color:#fff
    style ME3 fill:#7719AA,color:#fff
```

## Workflow Patterns

### Pattern 1: Request-Response

```mermaid
sequenceDiagram
    participant Client
    participant LogicApp as Logic App
    participant Service as Backend Service
    
    Client->>LogicApp: HTTP Request
    LogicApp->>Service: Process Request
    Service->>LogicApp: Response
    LogicApp->>Client: HTTP Response
```

```json
{
    "definition": {
        "triggers": {
            "manual": {
                "type": "Request",
                "kind": "Http",
                "inputs": {
                    "schema": {}
                }
            }
        },
        "actions": {
            "Response": {
                "type": "Response",
                "inputs": {
                    "statusCode": 200,
                    "body": "@triggerBody()"
                }
            }
        }
    }
}
```

### Pattern 2: Event-Driven Processing

```mermaid
flowchart LR
    SB[Service Bus] --> LA[Logic App]
    LA --> DB[(Database)]
    LA --> EMAIL[Send Email]
    LA --> TEAMS[Post to Teams]
    
    style LA fill:#0078D4,color:#fff
    style SB fill:#50E6FF
```

### Pattern 3: Scheduled Data Sync

```mermaid
flowchart TB
    SCHED[Schedule Trigger] --> GET[Get Data from Source]
    GET --> TRANSFORM[Transform Data]
    TRANSFORM --> FOREACH[For Each Record]
    FOREACH --> UPSERT[Upsert to Destination]
    UPSERT --> FOREACH
    FOREACH --> NOTIFY[Send Completion Report]
    
    style SCHED fill:#FF6B6B,color:#fff
    style GET fill:#0078D4,color:#fff
    style TRANSFORM fill:#0078D4,color:#fff
    style FOREACH fill:#FFE66D
    style UPSERT fill:#0078D4,color:#fff
    style NOTIFY fill:#0078D4,color:#fff
```

### Pattern 4: Approval Workflow

```mermaid
flowchart TB
    REQ[Request Received] --> VALIDATE[Validate Request]
    VALIDATE --> APPROVE{Approval<br/>Required?}
    APPROVE -->|Yes| SEND_APPROVAL[Send Approval Email]
    APPROVE -->|No| PROCESS[Process Request]
    SEND_APPROVAL --> WAIT[Wait for Response]
    WAIT --> CHECK{Approved?}
    CHECK -->|Yes| PROCESS
    CHECK -->|No| REJECT[Send Rejection]
    PROCESS --> COMPLETE[Complete Request]
    
    style REQ fill:#FF6B6B,color:#fff
    style APPROVE fill:#FFE66D
    style CHECK fill:#FFE66D
```

## Control Flow Actions

### Conditions

```json
{
    "Condition": {
        "type": "If",
        "expression": {
            "and": [
                {
                    "greater": ["@body('Get_item')?['amount']", 1000]
                }
            ]
        },
        "actions": {
            "High_value_processing": {}
        },
        "else": {
            "actions": {
                "Standard_processing": {}
            }
        }
    }
}
```

### Loops

| Loop Type | Description | Use Case |
|-----------|-------------|----------|
| **For Each** | Iterate over arrays | Process each item |
| **Until** | Loop until condition | Retry logic |
| **Do Until** | Loop with condition check | Polling scenarios |

### Scopes and Error Handling

```mermaid
flowchart TB
    subgraph SCOPE[Scope: Try]
        A1[Action 1]
        A2[Action 2]
        A3[Action 3]
    end
    
    subgraph CATCH[Scope: Catch]
        LOG[Log Error]
        NOTIFY[Send Alert]
    end
    
    SCOPE -->|Failed| CATCH
    SCOPE -->|Succeeded| CONTINUE[Continue Workflow]
    
    style SCOPE fill:#90EE90
    style CATCH fill:#FF6B6B,color:#fff
```

## Enterprise Integration

### B2B Capabilities

```mermaid
graph TB
    subgraph "Integration Account"
        TP[Trading Partners]
        AG[Agreements]
        MAPS[Maps/Transforms]
        SCHEMA[Schemas]
    end
    
    subgraph "B2B Connectors"
        AS2[AS2 Connector]
        X12[X12 Connector]
        EDI[EDIFACT Connector]
    end
    
    subgraph "Logic App Workflow"
        REC[Receive Message]
        DEC[Decode]
        PROC[Process]
        ENC[Encode]
        SEND[Send Response]
    end
    
    TP --> AG
    AG --> AS2 & X12 & EDI
    AS2 --> REC
    REC --> DEC --> PROC --> ENC --> SEND
    SCHEMA --> DEC
    MAPS --> PROC
    
    style REC fill:#0078D4,color:#fff
    style DEC fill:#0078D4,color:#fff
    style PROC fill:#0078D4,color:#fff
    style ENC fill:#0078D4,color:#fff
    style SEND fill:#0078D4,color:#fff
```

### Integration Account Features

| Feature | Description |
|---------|-------------|
| **Partners** | Define trading partner identities |
| **Agreements** | Protocol settings (AS2, X12, EDIFACT) |
| **Schemas** | XML/JSON schemas for validation |
| **Maps** | XSLT/Liquid transforms |
| **Certificates** | Security certificates for B2B |

## Standard Logic Apps - Advanced Features

### Stateless Workflows
- No run history persistence
- Lower latency
- Higher throughput
- Ideal for request-response scenarios

### Built-in Connectors (Standard)
More connectors run in-process for better performance:
- Azure Service Bus
- Azure Event Hubs
- Azure Storage
- SQL Server
- Azure Cosmos DB
- HTTP operations

### Local Development with VS Code

```mermaid
flowchart LR
    subgraph "Development"
        VSC[VS Code]
        EXT[Logic Apps Extension]
        EMU[Storage Emulator]
    end
    
    subgraph "Deployment"
        GIT[GitHub/Azure DevOps]
        PIPE[CI/CD Pipeline]
        AZURE[Azure Logic App]
    end
    
    VSC --> EXT --> EMU
    EMU --> GIT --> PIPE --> AZURE
    
    style VSC fill:#0078D4,color:#fff
    style AZURE fill:#0078D4,color:#fff
```

## Security

### Authentication Options

| Method | Description | Use Case |
|--------|-------------|----------|
| **Managed Identity** | Azure AD identity | Access Azure resources |
| **OAuth 2.0** | Token-based auth | API connections |
| **API Key** | Shared secret | Simple authentication |
| **Client Certificate** | Certificate-based | High-security scenarios |
| **Azure AD** | Enterprise identity | Internal applications |

### Security Best Practices

1. **Use Managed Identities** for Azure resource access
2. **Secure HTTP triggers** with Azure AD or SAS tokens
3. **Enable VNet integration** for network isolation (Standard)
4. **Store secrets in Key Vault** and reference them
5. **Enable diagnostic logging** for audit trails
6. **Use IP restrictions** where applicable

## Monitoring and Observability

### Run History

```mermaid
flowchart LR
    subgraph "Monitoring"
        RH[Run History]
        AI[Application Insights]
        AM[Azure Monitor]
        LA[Log Analytics]
    end
    
    LOGICAPP[Logic App] --> RH
    LOGICAPP --> AI
    AI --> AM --> LA
    
    style LOGICAPP fill:#0078D4,color:#fff
```

### Key Metrics to Monitor

| Metric | Description |
|--------|-------------|
| **Run success rate** | Percentage of successful runs |
| **Run latency** | Execution time |
| **Trigger success rate** | Trigger fire success |
| **Actions failed** | Number of failed actions |
| **Billable executions** | For cost tracking |

## Best Practices

### Design

| Practice | Description |
|----------|-------------|
| **Use scopes** | Group related actions for error handling |
| **Implement retry policies** | Handle transient failures |
| **Use inline code** | For simple transformations |
| **Parameterize values** | For environment-specific configs |
| **Design for idempotency** | Handle duplicate messages |

### Performance

| Practice | Description |
|----------|-------------|
| **Use batching** | Process multiple items together |
| **Optimize loops** | Use parallel processing |
| **Choose Standard** | For high-throughput scenarios |
| **Minimize connector calls** | Combine operations where possible |
| **Use built-in connectors** | Better performance than managed |

### Operations

| Practice | Description |
|----------|-------------|
| **Enable diagnostics** | Send logs to Log Analytics |
| **Set up alerts** | For failures and performance |
| **Use CI/CD** | Automated deployments |
| **Version workflows** | Track changes in source control |

## Architecture Patterns

### Pattern: Event-Driven Architecture

```mermaid
graph TB
    subgraph "Event Sources"
        ES1[Blob Storage]
        ES2[Service Bus]
        ES3[Event Grid]
    end
    
    subgraph "Logic Apps"
        LA1[Process Orders]
        LA2[Update Inventory]
        LA3[Send Notifications]
    end
    
    subgraph "Destinations"
        D1[(SQL Database)]
        D2[Email Service]
        D3[Teams Channel]
    end
    
    ES1 --> LA1 --> D1
    ES2 --> LA2 --> D1
    ES3 --> LA3 --> D2 & D3
    
    style LA1 fill:#0078D4,color:#fff
    style LA2 fill:#0078D4,color:#fff
    style LA3 fill:#0078D4,color:#fff
```

### Pattern: API Orchestration

```mermaid
flowchart TB
    API[API Request] --> LA[Logic App]
    LA --> P1[Call Service 1]
    LA --> P2[Call Service 2]
    LA --> P3[Call Service 3]
    P1 & P2 & P3 --> AGG[Aggregate Results]
    AGG --> RESP[Return Response]
    
    style LA fill:#0078D4,color:#fff
    style AGG fill:#50E6FF
```

## Pricing Considerations

### Consumption Logic Apps
- Pay per action execution
- Trigger executions
- Connector usage (Standard vs Enterprise)
- Integration account (for B2B)

### Standard Logic Apps
- Base compute cost (like App Service)
- Additional execution costs
- Storage costs
- VNet integration costs (if used)

## Hands-On Lab Ideas

1. **Create a request-response workflow** - Build an HTTP-triggered API
2. **Implement approval workflow** - Using Office 365 connectors
3. **Build data sync** - Schedule-based data movement
4. **B2B integration** - EDI message processing
5. **Event-driven processing** - Service Bus integration

---

## References

- [Azure Logic Apps Documentation](https://learn.microsoft.com/en-us/azure/logic-apps/)
- [What is Azure Logic Apps?](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-overview)
- [Single-tenant vs Multitenant](https://learn.microsoft.com/en-us/azure/logic-apps/single-tenant-overview-compare)
- [Connectors Reference](https://learn.microsoft.com/en-us/connectors/connector-reference/connector-reference-logicapps-connectors)
- [Logic Apps Pricing](https://azure.microsoft.com/en-us/pricing/details/logic-apps/)
- [Business Continuity and DR](https://learn.microsoft.com/en-us/azure/logic-apps/business-continuity-disaster-recovery-guidance)
- [Power Automate Migration Guide](https://learn.microsoft.com/en-us/azure/logic-apps/power-automate-migration)
