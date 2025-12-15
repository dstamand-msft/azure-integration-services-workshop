# Azure Event Grid

## Overview

Azure Event Grid is a highly scalable, fully managed event routing service that enables event-driven architectures using a publish-subscribe model. It simplifies event consumption and lowers costs by eliminating the need for constant polling, providing reliable delivery with retry and dead-letter capabilities.

Event Grid connects event sources to event handlers, allowing you to build reactive applications that respond to state changes across Azure services, custom applications, and partner solutions.

## Core Concepts

```mermaid
graph LR
    subgraph "Event Sources"
        ES1[Azure Services]
        ES2[Custom Applications]
        ES3[Partner Events]
    end
    
    subgraph "Event Grid"
        TOPIC[Topic/Domain]
        SUB1[Subscription 1]
        SUB2[Subscription 2]
        SUB3[Subscription 3]
    end
    
    subgraph "Event Handlers"
        EH1[Azure Functions]
        EH2[Logic Apps]
        EH3[Webhooks]
        EH4[Event Hubs]
    end
    
    ES1 & ES2 & ES3 --> TOPIC
    TOPIC --> SUB1 --> EH1
    TOPIC --> SUB2 --> EH2
    TOPIC --> SUB3 --> EH3
    
    style TOPIC fill:#0078D4,color:#fff
```

### Key Components

| Component | Description |
|-----------|-------------|
| **Event** | What happened (smallest unit of information) |
| **Event Source** | Where the event originated |
| **Topic** | Endpoint for receiving events |
| **Subscription** | Route events to handlers |
| **Event Handler** | Processes the event |

## Event Structure

### CloudEvents Schema (Recommended)

```json
{
    "specversion": "1.0",
    "type": "com.example.order.created",
    "source": "/mycontext/orders",
    "subject": "order/12345",
    "id": "A234-1234-1234",
    "time": "2024-01-15T12:00:00Z",
    "datacontenttype": "application/json",
    "data": {
        "orderId": "12345",
        "customerId": "C-100",
        "total": 150.00
    }
}
```

### Event Grid Schema

```json
{
    "id": "unique-event-id",
    "topic": "/subscriptions/{id}/resourceGroups/{rg}/...",
    "subject": "orders/12345",
    "eventType": "Order.Created",
    "eventTime": "2024-01-15T12:00:00Z",
    "data": {
        "orderId": "12345",
        "status": "new"
    },
    "dataVersion": "1.0"
}
```

## Topic Types

### System Topics (Azure Services)

```mermaid
graph TB
    subgraph "Azure Services"
        BLOB[Blob Storage]
        COSMOS[Cosmos DB]
        SB[Service Bus]
        RG[Resource Groups]
        AKV[Key Vault]
    end
    
    subgraph "System Topic"
        ST[Automatic Topic]
    end
    
    BLOB --> ST
    COSMOS --> ST
    SB --> ST
    RG --> ST
    AKV --> ST
    
    ST --> SUB[Subscriptions]
    
    style ST fill:#0078D4,color:#fff
```

**Supported Azure Event Sources:**
- Storage (Blob, Files, Data Lake)
- Container Registry
- Event Hubs
- IoT Hub
- Key Vault
- Azure Maps
- Media Services
- Resource Groups
- Service Bus
- App Configuration
- Communication Services
- Machine Learning
- And many more...

### Custom Topics

```mermaid
flowchart LR
    APP1[App 1] -->|HTTP POST| CT[Custom Topic]
    APP2[App 2] -->|HTTP POST| CT
    APP3[App 3] -->|HTTP POST| CT
    
    CT --> S1[Subscription 1]
    CT --> S2[Subscription 2]
    
    S1 --> H1[Handler 1]
    S2 --> H2[Handler 2]
    
    style CT fill:#50E6FF
```

### Domain Topics

```mermaid
graph TB
    subgraph "Event Grid Domain"
        D[Domain: E-Commerce]
        DT1[Topic: Orders]
        DT2[Topic: Inventory]
        DT3[Topic: Shipping]
    end
    
    D --> DT1 & DT2 & DT3
    
    PUB1[Order Service] --> DT1
    PUB2[Inventory Service] --> DT2
    PUB3[Shipping Service] --> DT3
    
    DT1 --> SUB1[Order Handlers]
    DT2 --> SUB2[Inventory Handlers]
    DT3 --> SUB3[Shipping Handlers]
    
    style D fill:#0078D4,color:#fff
```

**Domain Benefits:**
- Up to 100,000 topics per domain
- Single endpoint for all topics
- Centralized access control
- Organized event management

### Partner Topics

Pre-integrated SaaS partner events:
- Auth0
- Microsoft Graph
- SAP
- Twilio

## Event Handlers

```mermaid
graph TB
    EG[Event Grid] --> AZ_FUNC[Azure Functions]
    EG --> LOGIC[Logic Apps]
    EG --> WEBHOOK[Webhooks]
    EG --> EH[Event Hubs]
    EG --> SB[Service Bus]
    EG --> STORAGE[Storage Queues]
    EG --> RELAY[Hybrid Connections]
    EG --> PA[Power Automate]
    
    style EG fill:#0078D4,color:#fff
```

### Handler Comparison

| Handler | Latency | Use Case |
|---------|---------|----------|
| **Azure Functions** | Low | Serverless processing |
| **Logic Apps** | Medium | Workflow orchestration |
| **Webhooks** | Low | Custom applications |
| **Event Hubs** | Low | Stream processing |
| **Service Bus** | Low | Queue-based processing |
| **Storage Queues** | Medium | Simple queuing |

## Delivery Modes

### Push Delivery (Default)

```mermaid
sequenceDiagram
    participant Source as Event Source
    participant EG as Event Grid
    participant Handler as Event Handler
    
    Source->>EG: Publish Event
    EG->>Handler: Push Event (HTTP)
    Handler->>EG: 200 OK
    Note over EG: Event Delivered
```

### Pull Delivery (Namespaces)

```mermaid
sequenceDiagram
    participant Source as Event Source
    participant NS as Namespace Topic
    participant Consumer as Consumer
    
    Source->>NS: Publish Event
    loop Polling
        Consumer->>NS: Receive Events
        NS->>Consumer: Events Batch
        Consumer->>NS: Acknowledge
    end
```

**Pull Delivery Benefits:**
- Consumer controls rate
- Batch processing
- Better for high-volume scenarios
- CloudEvents support

## Event Grid Namespaces

```mermaid
graph TB
    subgraph "Namespace"
        NT[Namespace Topic]
        ES1[Event Subscription]
        ES2[Event Subscription]
    end
    
    subgraph "Features"
        MQTT[MQTT Broker]
        PULL[Pull Delivery]
        CLOUD[CloudEvents Native]
    end
    
    PUB[Publisher] --> NT
    NT --> ES1 --> CONS1[Consumer 1]
    NT --> ES2 --> CONS2[Consumer 2]
    
    MQTT --> NT
    
    style NT fill:#50E6FF
```

### MQTT Support
- IoT device communication
- Publish/subscribe over MQTT
- QoS 0 and 1 support
- Topic spaces for routing

## Filtering

### Event Type Filtering

```mermaid
flowchart TB
    EVENT[Incoming Events] --> TOPIC[Topic]
    TOPIC --> FILTER{Event Type<br/>Filter}
    FILTER -->|BlobCreated| S1[Subscription 1]
    FILTER -->|BlobDeleted| S2[Subscription 2]
    
    style TOPIC fill:#0078D4,color:#fff
```

```json
{
    "filter": {
        "includedEventTypes": [
            "Microsoft.Storage.BlobCreated",
            "Microsoft.Storage.BlobDeleted"
        ]
    }
}
```

### Subject Filtering

```json
{
    "filter": {
        "subjectBeginsWith": "/blobServices/default/containers/orders",
        "subjectEndsWith": ".json"
    }
}
```

### Advanced Filtering

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "NumberGreaterThan",
                "key": "data.amount",
                "value": 1000
            },
            {
                "operatorType": "StringContains",
                "key": "data.region",
                "values": ["US", "EU"]
            },
            {
                "operatorType": "IsNotNull",
                "key": "data.customerId"
            }
        ]
    }
}
```

### Filter Operators

| Operator | Description |
|----------|-------------|
| **NumberIn/NumberNotIn** | Value in/not in list |
| **NumberGreaterThan** | Value comparison |
| **NumberLessThan** | Value comparison |
| **BoolEquals** | Boolean check |
| **StringContains** | Text contains |
| **StringBeginsWith** | Text prefix |
| **StringEndsWith** | Text suffix |
| **IsNullOrUndefined** | Null check |
| **IsNotNull** | Not null check |

## Reliability Features

### Retry Policy

```mermaid
flowchart TB
    EVENT[Event] --> DELIVER{Deliver}
    DELIVER -->|Success| DONE[Complete]
    DELIVER -->|Failure| RETRY{Retry?}
    RETRY -->|Yes| WAIT[Exponential Backoff]
    WAIT --> DELIVER
    RETRY -->|Max Retries| DLQ[Dead Letter]
    
    style DELIVER fill:#0078D4,color:#fff
    style DLQ fill:#FF6B6B,color:#fff
```

**Retry Configuration:**
- Maximum retry attempts (1-30)
- Event time-to-live (1 minute - 24 hours)
- Exponential backoff

### Dead-Letter Destination

```mermaid
flowchart LR
    EG[Event Grid] -->|Delivery Failed| BLOB[Blob Storage]
    BLOB --> REVIEW[Admin Review]
    REVIEW --> REPLAY[Replay Events]
    
    style EG fill:#0078D4,color:#fff
    style BLOB fill:#FF6B6B,color:#fff
```

```json
{
    "deadLetterDestination": {
        "endpointType": "StorageBlob",
        "properties": {
            "resourceId": "/subscriptions/.../storageAccounts/mystorage",
            "blobContainerName": "deadletters"
        }
    }
}
```

### Batching

```json
{
    "deliveryPolicy": {
        "maxEventsPerBatch": 50,
        "preferredBatchSizeInKilobytes": 64
    }
}
```

## Security

### Authentication

| Method | Description | Use Case |
|--------|-------------|----------|
| **Webhook Validation** | Validation handshake | Custom endpoints |
| **Azure AD** | OAuth token | Enterprise apps |
| **SAS Token** | Shared access | Publishing events |
| **Managed Identity** | Azure identity | Azure services |

### Webhook Validation

```mermaid
sequenceDiagram
    participant EG as Event Grid
    participant WH as Webhook
    
    EG->>WH: Validation Event
    Note over WH: validationCode in request
    WH->>EG: Return validationCode
    Note over EG: Subscription Validated
    
    EG->>WH: Events (normal delivery)
```

### Network Security

```mermaid
graph TB
    subgraph "Security Options"
        IP[IP Firewall]
        PE[Private Endpoints]
        MI[Managed Identity]
    end
    
    subgraph "Custom/Domain Topics"
        CT[Topic]
    end
    
    IP --> CT
    PE --> CT
    MI --> CT
    
    style CT fill:#0078D4,color:#fff
```

## Architecture Patterns

### Pattern 1: Event-Driven Microservices

```mermaid
flowchart TB
    subgraph "Microservices"
        OS[Order Service]
        IS[Inventory Service]
        NS[Notification Service]
        AS[Analytics Service]
    end
    
    subgraph "Event Grid"
        T[Custom Topic]
    end
    
    OS -->|Order.Created| T
    T --> IS
    T --> NS
    T --> AS
    
    style T fill:#0078D4,color:#fff
```

### Pattern 2: Multi-System Integration

```mermaid
flowchart LR
    subgraph "Azure"
        BLOB[Blob Storage]
        ST[System Topic]
    end
    
    subgraph "Event Grid"
        EG[Event Grid]
    end
    
    subgraph "Handlers"
        FUNC[Azure Function]
        LOGIC[Logic App]
        WEBHOOK[External System]
    end
    
    BLOB --> ST --> EG
    EG --> FUNC
    EG --> LOGIC
    EG --> WEBHOOK
    
    style EG fill:#0078D4,color:#fff
```

### Pattern 3: Fan-Out Processing

```mermaid
flowchart TB
    SOURCE[Event Source] --> TOPIC[Topic]
    
    TOPIC --> S1[Subscription: Process]
    TOPIC --> S2[Subscription: Archive]
    TOPIC --> S3[Subscription: Notify]
    TOPIC --> S4[Subscription: Audit]
    
    S1 --> FUNC[Function App]
    S2 --> STORAGE[Blob Storage]
    S3 --> LOGIC[Logic App]
    S4 --> EH[Event Hubs]
    
    style TOPIC fill:#50E6FF
```

### Pattern 4: Azure Resource Events

```mermaid
flowchart TB
    subgraph "Resource Events"
        VM[VM Created/Deleted]
        RG[Resource Group Changes]
        POLICY[Policy Violations]
    end
    
    subgraph "System Topic"
        ST[Azure Subscription Topic]
    end
    
    subgraph "Automation"
        LA[Logic App]
        FUNC[Function]
    end
    
    VM & RG & POLICY --> ST
    ST --> LA --> TEAMS[Teams Notification]
    ST --> FUNC --> REMEDIATE[Auto-Remediate]
    
    style ST fill:#0078D4,color:#fff
```

## Code Examples

### Publishing Events (C#)

```csharp
using Azure.Messaging.EventGrid;

var client = new EventGridPublisherClient(
    new Uri("https://mytopic.region.eventgrid.azure.net/api/events"),
    new AzureKeyCredential("your-key")
);

// Publish Event Grid format
var events = new List<EventGridEvent>
{
    new EventGridEvent(
        subject: "orders/12345",
        eventType: "Order.Created",
        dataVersion: "1.0",
        data: new { OrderId = "12345", Amount = 150.00 }
    )
};

await client.SendEventsAsync(events);

// Publish CloudEvents format
var cloudEvent = new CloudEvent(
    source: "/orders",
    type: "com.mycompany.order.created",
    jsonSerializableData: new { OrderId = "12345" }
);

await client.SendEventAsync(cloudEvent);
```

### Azure Function Handler

```csharp
[Function("ProcessOrderEvent")]
public async Task Run(
    [EventGridTrigger] CloudEvent cloudEvent,
    FunctionContext context)
{
    var logger = context.GetLogger("ProcessOrderEvent");
    
    var orderData = cloudEvent.Data.ToObjectFromJson<OrderData>();
    logger.LogInformation($"Processing order: {orderData.OrderId}");
    
    // Process the event
    await ProcessOrder(orderData);
}
```

## Monitoring

### Key Metrics

| Metric | Description |
|--------|-------------|
| **Publish Success Count** | Events successfully published |
| **Publish Failed Count** | Failed publish attempts |
| **Delivery Success Count** | Events delivered |
| **Delivery Failed Count** | Failed deliveries |
| **Matched Events** | Events matching filters |
| **Dead Lettered Events** | Events sent to DLQ |

```mermaid
flowchart LR
    EG[Event Grid] --> AM[Azure Monitor]
    AM --> LA[Log Analytics]
    AM --> ALERTS[Alerts]
    AM --> METRICS[Metrics Explorer]
    
    style EG fill:#0078D4,color:#fff
```

## Best Practices

### Design

| Practice | Description |
|----------|-------------|
| **Use CloudEvents** | Standard, portable schema |
| **Design idempotent handlers** | Handle duplicate events |
| **Keep events small** | Store details elsewhere |
| **Use domains** | For large applications |
| **Version event types** | For evolution |

### Performance

| Practice | Description |
|----------|-------------|
| **Use batching** | Reduce HTTP overhead |
| **Enable dead-lettering** | Don't lose events |
| **Set appropriate TTL** | Based on requirements |
| **Use namespaces** | For high-volume pull |

### Operations

| Practice | Description |
|----------|-------------|
| **Monitor delivery rates** | Track success/failure |
| **Alert on dead letters** | Investigate failures |
| **Use diagnostic logs** | For troubleshooting |
| **Test with Event Viewer** | Debug subscriptions |

## Events vs Messages

| Aspect | Event Grid (Events) | Service Bus (Messages) |
|--------|---------------------|------------------------|
| **Purpose** | Notification of state change | Work to be done |
| **Coupling** | Loose | Can be tighter |
| **Consumer** | Multiple (pub-sub) | Single (point-to-point) |
| **Content** | What happened | Instructions/data |
| **Response** | Optional | Often expected |
| **Retention** | Short-term | Configurable |

## Hands-On Lab Ideas

1. **React to blob storage events** - File processing pipeline
2. **Custom topic with filtering** - Multi-subscriber routing
3. **Resource group monitoring** - Governance automation
4. **Event Grid domain** - Multi-tenant application
5. **MQTT with namespaces** - IoT device integration

---

## References

- [Azure Event Grid Documentation](https://learn.microsoft.com/en-us/azure/event-grid/)
- [Event Grid Overview](https://learn.microsoft.com/en-us/azure/event-grid/overview)
- [System Topics](https://learn.microsoft.com/en-us/azure/event-grid/system-topics)
- [Custom Topics](https://learn.microsoft.com/en-us/azure/event-grid/custom-topics)
- [Event Handlers](https://learn.microsoft.com/en-us/azure/event-grid/event-handlers)
- [Event Filtering](https://learn.microsoft.com/en-us/azure/event-grid/event-filtering)
- [CloudEvents Schema](https://learn.microsoft.com/en-us/azure/event-grid/cloud-event-schema)
- [Event Grid Namespaces](https://learn.microsoft.com/en-us/azure/event-grid/concepts-event-grid-namespaces)
