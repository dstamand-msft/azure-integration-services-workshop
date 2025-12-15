# Azure Integration Services - Comparison Guide

## Overview

This document provides a comprehensive comparison of Azure Integration Services to help you choose the right service for your integration scenarios. Each service has distinct strengths and is optimized for specific use cases.

## Service-at-a-Glance

```mermaid
graph TB
    subgraph "API & Gateway"
        APIM[API Management]
    end
    
    subgraph "Workflow & Orchestration"
        LOGIC[Logic Apps]
        ADF[Data Factory]
    end
    
    subgraph "Messaging & Events"
        SB[Service Bus]
        EG[Event Grid]
    end
    
    subgraph "Compute"
        FUNC[Azure Functions]
    end
    
    style APIM fill:#0078D4,color:#fff
    style LOGIC fill:#50E6FF
    style ADF fill:#7719AA,color:#fff
    style SB fill:#FF6B6B,color:#fff
    style EG fill:#90EE90
    style FUNC fill:#FFE66D
```

## Quick Reference Table

| Service | Primary Purpose | Best For | Pricing Model |
|---------|-----------------|----------|---------------|
| **API Management** | API gateway & management | API publishing, security, monetization | Per-call + base |
| **Logic Apps** | Workflow automation | Business process automation, B2B | Per-execution |
| **Service Bus** | Enterprise messaging | Reliable messaging, decoupling | Per-operation |
| **Event Grid** | Event routing | Reactive architectures, event distribution | Per-event |
| **Azure Functions** | Serverless compute | Event-driven processing, APIs | Per-execution |
| **Data Factory** | Data integration | ETL/ELT, data pipelines | Per-activity |

## Decision Framework

### When to Use Each Service

```mermaid
flowchart TB
    START[Integration Need] --> Q1{API-centric?}
    
    Q1 -->|Yes| Q2{Need gateway<br/>features?}
    Q1 -->|No| Q3{Data movement<br/>focus?}
    
    Q2 -->|Yes| APIM[API Management]
    Q2 -->|No| FUNC[Azure Functions]
    
    Q3 -->|Yes| ADF[Data Factory]
    Q3 -->|No| Q4{Event or<br/>Message?}
    
    Q4 -->|Event| Q5{Multiple<br/>subscribers?}
    Q4 -->|Message| Q6{Need queuing<br/>features?}
    
    Q5 -->|Yes| EG[Event Grid]
    Q5 -->|No| FUNC2[Azure Functions]
    
    Q6 -->|Yes| SB[Service Bus]
    Q6 -->|No| Q7{Complex<br/>workflow?}
    
    Q7 -->|Yes| LOGIC[Logic Apps]
    Q7 -->|No| FUNC3[Azure Functions]
    
    style APIM fill:#0078D4,color:#fff
    style LOGIC fill:#50E6FF
    style SB fill:#FF6B6B,color:#fff
    style EG fill:#90EE90
    style FUNC fill:#FFE66D
    style FUNC2 fill:#FFE66D
    style FUNC3 fill:#FFE66D
    style ADF fill:#7719AA,color:#fff
```

## Events vs Messages

Understanding when to use events versus messages is crucial for choosing between Service Bus and Event Grid.

### Comparison

| Aspect | Events (Event Grid) | Messages (Service Bus) |
|--------|---------------------|------------------------|
| **Purpose** | Notification of state change | Command or data transfer |
| **Content** | What happened | Work to be done |
| **Publisher expectation** | Fire and forget | Expects processing |
| **Consumer count** | Multiple (broadcast) | Single/competing |
| **Coupling** | Very loose | Can be tighter |
| **Ordering** | Not guaranteed | Guaranteed (sessions) |
| **Retention** | Short-term | Configurable |
| **Size** | Small (64KB-1MB) | Up to 100MB (Premium) |

### Decision Matrix

```mermaid
flowchart TB
    MSG[Need Communication?] --> Q1{What's the intent?}
    
    Q1 -->|Notify about<br/>something| EVENT[Use Event Grid]
    Q1 -->|Request action<br/>or send data| Q2{Need reliability<br/>guarantees?}
    
    Q2 -->|Yes| SB[Use Service Bus]
    Q2 -->|No| Q3{High volume<br/>streaming?}
    
    Q3 -->|Yes| EH[Consider Event Hubs]
    Q3 -->|No| SB2[Use Service Bus]
    
    style EVENT fill:#90EE90
    style SB fill:#FF6B6B,color:#fff
    style SB2 fill:#FF6B6B,color:#fff
```

### Examples

| Scenario | Service | Reason |
|----------|---------|--------|
| "Blob was created" | Event Grid | Notification of state change |
| "Process this order" | Service Bus | Work item requiring action |
| "Configuration changed" | Event Grid | Multiple systems need to react |
| "Send invoice to customer" | Service Bus | Reliable delivery required |
| "User logged in" | Event Grid | Audit/notification |
| "Calculate monthly bill" | Service Bus | Task to be processed |

## Logic Apps vs Azure Functions

### Feature Comparison

| Feature | Logic Apps | Azure Functions |
|---------|------------|-----------------|
| **Development** | Visual designer | Code-first |
| **Connectors** | 400+ pre-built | Bindings + custom |
| **State management** | Built-in | Durable Functions |
| **Long-running** | Native support | Durable Functions |
| **B2B/EDI** | Native support | Custom code |
| **Monitoring** | Run history | Application Insights |
| **Cold start** | Possible | Plan-dependent |
| **Cost model** | Per-action | Per-execution |

### When to Choose

```mermaid
flowchart TB
    NEED[Automation Need] --> Q1{Custom code<br/>required?}
    
    Q1 -->|Complex logic| FUNC[Azure Functions]
    Q1 -->|Standard patterns| Q2{B2B/EDI<br/>integration?}
    
    Q2 -->|Yes| LOGIC[Logic Apps]
    Q2 -->|No| Q3{Visual design<br/>preference?}
    
    Q3 -->|Yes| LOGIC2[Logic Apps]
    Q3 -->|No| Q4{Citizen<br/>developer?}
    
    Q4 -->|Yes| LOGIC3[Logic Apps]
    Q4 -->|No| Q5{Performance<br/>critical?}
    
    Q5 -->|Yes| FUNC2[Azure Functions]
    Q5 -->|No| EITHER[Either works]
    
    style LOGIC fill:#50E6FF
    style LOGIC2 fill:#50E6FF
    style LOGIC3 fill:#50E6FF
    style FUNC fill:#FFE66D
    style FUNC2 fill:#FFE66D
```

### Scenario Examples

| Scenario | Recommendation | Reason |
|----------|----------------|--------|
| Office 365 automation | Logic Apps | Pre-built connectors |
| High-performance API | Azure Functions | Lower latency |
| SAP integration | Logic Apps | Enterprise connectors |
| Complex algorithms | Azure Functions | Full code control |
| Approval workflows | Logic Apps | Built-in patterns |
| Real-time processing | Azure Functions | Event triggers |

## Data Factory vs Logic Apps

### Feature Comparison

| Feature | Data Factory | Logic Apps |
|---------|--------------|------------|
| **Primary focus** | Data movement | Workflow automation |
| **Data transformation** | Native (Data Flows) | Limited |
| **Connectors** | 90+ data sources | 400+ services |
| **Scale** | Petabyte scale | Business workflow scale |
| **On-premises** | Self-hosted IR | On-premises gateway |
| **Monitoring** | Pipeline monitoring | Run history |
| **CI/CD** | Git integration | ARM/CI integration |

### Decision Matrix

| Scenario | Data Factory | Logic Apps |
|----------|--------------|------------|
| ETL/ELT pipelines | ✓ | |
| Data warehouse loading | ✓ | |
| Scheduled data sync | ✓ | |
| Business workflow | | ✓ |
| B2B integration | | ✓ |
| API orchestration | | ✓ |
| Event reactions | | ✓ |
| Data transformation | ✓ | |

### Combined Architecture

Often these services work together:

```mermaid
flowchart TB
    subgraph "Data Integration"
        ADF[Data Factory]
    end
    
    subgraph "Process Automation"
        LOGIC[Logic Apps]
    end
    
    EVENT[Business Event] --> LOGIC
    LOGIC --> TRIGGER[Trigger ADF Pipeline]
    TRIGGER --> ADF
    ADF --> COMPLETE[Complete]
    COMPLETE --> LOGIC
    LOGIC --> NOTIFY[Notify Stakeholders]
    
    style ADF fill:#7719AA,color:#fff
    style LOGIC fill:#50E6FF
```

## API Management vs Azure Functions

### When to Use Together vs Separately

```mermaid
flowchart TB
    API[API Requirement] --> Q1{Need API<br/>management?}
    
    Q1 -->|Yes| Q2{What backend?}
    Q1 -->|No| FUNC_ONLY[Functions Only]
    
    Q2 -->|Serverless| APIM_FUNC[APIM + Functions]
    Q2 -->|Existing services| APIM_ONLY[APIM + Backend]
    Q2 -->|Mixed| APIM_MIXED[APIM + Multiple]
    
    style FUNC_ONLY fill:#FFE66D
    style APIM_FUNC fill:#0078D4,color:#fff
    style APIM_ONLY fill:#0078D4,color:#fff
    style APIM_MIXED fill:#0078D4,color:#fff
```

### Feature Matrix

| Capability | Functions Alone | APIM + Functions |
|------------|-----------------|------------------|
| Basic HTTP APIs | ✓ | ✓ |
| Rate limiting | Limited | ✓ |
| API versioning | Manual | Built-in |
| OAuth validation | Code | Policy |
| Caching | Code | Policy |
| Analytics | App Insights | Built-in + App Insights |
| Developer portal | ✗ | ✓ |
| API monetization | ✗ | ✓ |

## Comprehensive Feature Matrix

### Core Capabilities

| Capability | APIM | Logic Apps | Service Bus | Event Grid | Functions | Data Factory |
|------------|------|------------|-------------|------------|-----------|--------------|
| **Serverless** | - | ✓ | - | ✓ | ✓ | Managed |
| **Visual design** | - | ✓ | - | - | - | ✓ |
| **Code-first** | - | ✓ | - | - | ✓ | - |
| **Pre-built connectors** | - | 400+ | - | - | Bindings | 90+ |
| **Event-driven** | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Schedule-driven** | - | ✓ | - | - | ✓ | ✓ |

### Enterprise Features

| Capability | APIM | Logic Apps | Service Bus | Event Grid | Functions | Data Factory |
|------------|------|------------|-------------|------------|-----------|--------------|
| **VNet Integration** | ✓ | Std | Premium | ✓ | Premium | ✓ |
| **Private Endpoints** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Managed Identity** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **RBAC** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Geo-redundancy** | ✓ | - | ✓ | ✓ | - | - |
| **Customer-managed keys** | ✓ | - | Premium | - | - | ✓ |

## Integration Patterns

### Pattern 1: API-First Architecture

```mermaid
flowchart TB
    CLIENTS[Clients] --> APIM[API Management]
    
    subgraph "Backend Services"
        FUNC[Azure Functions]
        LOGIC[Logic Apps]
        AKS[AKS/Apps]
    end
    
    APIM --> FUNC
    APIM --> LOGIC
    APIM --> AKS
    
    style APIM fill:#0078D4,color:#fff
```

**Services Used:** API Management + Azure Functions/Logic Apps

### Pattern 2: Event-Driven Microservices

```mermaid
flowchart TB
    subgraph "Services"
        SVC1[Order Service]
        SVC2[Inventory Service]
        SVC3[Notification Service]
    end
    
    SVC1 -->|Events| EG[Event Grid]
    EG --> SVC2
    EG --> SVC3
    
    SVC2 -->|Messages| SB[Service Bus]
    SB --> FUNC[Azure Function]
    
    style EG fill:#90EE90
    style SB fill:#FF6B6B,color:#fff
```

**Services Used:** Event Grid + Service Bus + Azure Functions

### Pattern 3: Enterprise Integration

```mermaid
flowchart LR
    subgraph "Partners"
        SAP[SAP]
        EDI[EDI Partners]
    end
    
    subgraph "Integration Layer"
        LOGIC[Logic Apps]
        SB[Service Bus]
    end
    
    subgraph "Internal"
        FUNC[Functions]
        DB[(Database)]
    end
    
    SAP --> LOGIC
    EDI --> LOGIC
    LOGIC <--> SB
    SB --> FUNC --> DB
    
    style LOGIC fill:#50E6FF
    style SB fill:#FF6B6B,color:#fff
```

**Services Used:** Logic Apps + Service Bus + Azure Functions

### Pattern 4: Data Integration

```mermaid
flowchart TB
    subgraph "Sources"
        SRC1[(On-Prem DB)]
        SRC2[APIs]
        SRC3[Files]
    end
    
    ADF[Data Factory] --> SRC1 & SRC2 & SRC3
    ADF --> LAKE[(Data Lake)]
    LAKE --> SYN[(Synapse)]
    
    EG[Event Grid] --> FUNC[Function]
    FUNC --> NOTIFY[Notifications]
    ADF --> EG
    
    style ADF fill:#7719AA,color:#fff
    style EG fill:#90EE90
```

**Services Used:** Data Factory + Event Grid + Azure Functions

## Cost Comparison

### Pricing Models

| Service | Model | Key Factors |
|---------|-------|-------------|
| **API Management** | Tier-based + calls | Tier, API calls, bandwidth |
| **Logic Apps** | Per-execution | Actions, connectors, runtime |
| **Service Bus** | Per-operation | Messages, tier, size |
| **Event Grid** | Per-event | Events published, operations |
| **Azure Functions** | Per-execution | Executions, GB-seconds |
| **Data Factory** | Per-activity | Activities, DIU, data flow |

### Cost Optimization Tips

| Service | Tip |
|---------|-----|
| **APIM** | Choose appropriate tier, use caching |
| **Logic Apps** | Use Standard for high volume, built-in connectors |
| **Service Bus** | Batch messages, choose right tier |
| **Event Grid** | Filter events at subscription |
| **Functions** | Use consumption for sporadic, Premium for consistent |
| **Data Factory** | Right-size DIUs, use incremental loads |

## Migration Paths

### From On-Premises to Azure

| On-Premises | Azure Service | Migration Path |
|-------------|---------------|----------------|
| BizTalk Server | Logic Apps | Assess → Migrate connectors → Refactor workflows |
| SSIS | Data Factory | Azure-SSIS IR or refactor to ADF |
| MQ/MSMQ | Service Bus | Message mapping → Queue migration |
| Custom ETL | Data Factory | Pipeline-by-pipeline conversion |
| API Gateway | APIM | Import APIs → Migrate policies |

## Hybrid Scenarios

### Connectivity Options

| Service | Hybrid Connectivity |
|---------|---------------------|
| **API Management** | Self-hosted gateway, VPN |
| **Logic Apps** | On-premises data gateway |
| **Service Bus** | Hybrid Connections |
| **Event Grid** | Partner topics, webhooks |
| **Functions** | Hybrid Connections, VNet |
| **Data Factory** | Self-hosted Integration Runtime |

```mermaid
flowchart TB
    subgraph "Cloud"
        ADF[Data Factory]
        LOGIC[Logic Apps]
        SB[Service Bus]
    end
    
    subgraph "Hybrid Connectivity"
        SHIR[Self-Hosted IR]
        GW[Data Gateway]
        HC[Hybrid Connections]
    end
    
    subgraph "On-Premises"
        DB[(Databases)]
        FILES[File Systems]
        APPS[Applications]
    end
    
    ADF --> SHIR --> DB
    LOGIC --> GW --> DB & FILES
    SB --> HC --> APPS
    
    style SHIR fill:#50E6FF
    style GW fill:#50E6FF
    style HC fill:#50E6FF
```

## Recommendations by Scenario

### Web/Mobile Backend

**Recommended:** API Management + Azure Functions
- Functions for API logic
- APIM for management, security, caching

### Business Process Automation

**Recommended:** Logic Apps + Service Bus
- Logic Apps for orchestration
- Service Bus for reliable messaging

### Real-Time Event Processing

**Recommended:** Event Grid + Azure Functions
- Event Grid for event routing
- Functions for event processing

### Enterprise Data Integration

**Recommended:** Data Factory + Logic Apps
- Data Factory for ETL/ELT
- Logic Apps for workflow triggers

### B2B Integration

**Recommended:** Logic Apps (Standard)
- Enterprise connectors
- B2B protocols (EDI, AS2)
- Integration accounts

### IoT Integration

**Recommended:** Event Grid + Functions + Service Bus
- Event Grid for IoT Hub events
- Functions for processing
- Service Bus for command delivery

## Summary Decision Tree

```mermaid
flowchart TB
    START[Integration Need] --> TYPE{What type?}
    
    TYPE -->|API Management| APIM_Q{External<br/>exposure?}
    TYPE -->|Workflow| WORK_Q{Complexity?}
    TYPE -->|Messaging| MSG_Q{Pattern?}
    TYPE -->|Data Movement| ADF[Data Factory]
    TYPE -->|Compute| FUNC_Q{Workload?}
    
    APIM_Q -->|Yes| APIM[API Management]
    APIM_Q -->|No| FUNC1[Functions]
    
    WORK_Q -->|Complex/B2B| LOGIC[Logic Apps]
    WORK_Q -->|Simple| FUNC2[Functions]
    
    MSG_Q -->|Pub/Sub Events| EG[Event Grid]
    MSG_Q -->|Reliable Queue| SB[Service Bus]
    
    FUNC_Q -->|Event-driven| FUNC3[Functions]
    FUNC_Q -->|Scheduled| FUNC4[Functions + Timer]
    
    style APIM fill:#0078D4,color:#fff
    style LOGIC fill:#50E6FF
    style SB fill:#FF6B6B,color:#fff
    style EG fill:#90EE90
    style FUNC1 fill:#FFE66D
    style FUNC2 fill:#FFE66D
    style FUNC3 fill:#FFE66D
    style FUNC4 fill:#FFE66D
    style ADF fill:#7719AA,color:#fff
```

---

## References

- [Azure Integration Services](https://azure.microsoft.com/en-us/products/category/integration/)
- [Choose an Azure integration service](https://learn.microsoft.com/en-us/azure/architecture/guide/technology-choices/integration-services)
- [Enterprise Integration Patterns](https://learn.microsoft.com/en-us/azure/architecture/patterns/)
- [Azure Architecture Center - Integration](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven)
- [Events vs Messages](https://learn.microsoft.com/en-us/azure/event-grid/compare-messaging-services)
