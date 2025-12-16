# Azure Service Bus

## Overview

Azure Service Bus is a fully managed enterprise message broker that provides reliable cloud messaging as a service (MaaS) and simple hybrid integration. It supports both queue-based point-to-point messaging and publish-subscribe patterns through topics and subscriptions.

Service Bus is designed for enterprise applications requiring high reliability, ordered delivery, and transactional support. It's a core component of Azure Integration Services, enabling asynchronous and decoupled communication between applications.

## Core Concepts

```mermaid
graph TB
    subgraph "Azure Service Bus Namespace"
        subgraph "Queues"
            Q1[Queue 1]
            Q2[Queue 2]
        end
        
        subgraph "Topics"
            T1[Topic 1]
            SUB1[Subscription A]
            SUB2[Subscription B]
            T1 --> SUB1
            T1 --> SUB2
        end
    end
    
    SENDER1[Sender] --> Q1
    Q1 --> RECEIVER1[Receiver]
    
    SENDER2[Publisher] --> T1
    SUB1 --> SUB_REC1[Subscriber 1]
    SUB2 --> SUB_REC2[Subscriber 2]
    
    style Q1 fill:#0078D4,color:#fff
    style Q2 fill:#0078D4,color:#fff
    style T1 fill:#50E6FF
```

### Queues vs Topics

| Feature | Queues | Topics |
|---------|--------|--------|
| **Pattern** | Point-to-point | Publish-subscribe |
| **Receivers** | Single receiver | Multiple subscriptions |
| **Message delivery** | One consumer | Each subscription gets copy |
| **Use case** | Load leveling, decoupling | Broadcasting, fan-out |

## Messaging Patterns

### Pattern 1: Point-to-Point (Queues)

```mermaid
sequenceDiagram
    participant Sender
    participant Queue as Service Bus Queue
    participant Receiver
    
    Sender->>Queue: Send Message
    Queue-->>Queue: Store Message
    Receiver->>Queue: Receive Message
    Queue->>Receiver: Deliver Message
    Receiver->>Queue: Complete/Abandon
```

### Pattern 2: Publish-Subscribe (Topics)

```mermaid
sequenceDiagram
    participant Publisher
    participant Topic as Service Bus Topic
    participant SubA as Subscription A
    participant SubB as Subscription B
    participant ConsumerA
    participant ConsumerB
    
    Publisher->>Topic: Publish Message
    Topic->>SubA: Copy Message
    Topic->>SubB: Copy Message
    ConsumerA->>SubA: Receive
    ConsumerB->>SubB: Receive
```

### Pattern 3: Request-Reply

```mermaid
flowchart LR
    subgraph "Request-Reply Pattern"
        REQ[Request Queue]
        RESP[Reply Queue]
    end
    
    CLIENT[Client] -->|1. Send Request| REQ
    REQ -->|2. Process| SERVICE[Service]
    SERVICE -->|3. Send Reply| RESP
    RESP -->|4. Receive Reply| CLIENT
    
    style REQ fill:#0078D4,color:#fff
    style RESP fill:#50E6FF
```

## Message Features

### Message Structure

```mermaid
graph TB
    subgraph "Service Bus Message"
        subgraph "System Properties"
            SP1[MessageId]
            SP2[CorrelationId]
            SP3[ContentType]
            SP4[TimeToLive]
            SP5[ScheduledEnqueueTime]
        end
        
        subgraph "Custom Properties"
            CP1[User-defined key-value pairs]
        end
        
        subgraph "Body"
            BODY[Message Payload]
        end
    end
```

### Key Message Properties

| Property | Description | Use Case |
|----------|-------------|----------|
| **MessageId** | Unique identifier | Deduplication |
| **CorrelationId** | Correlation tracking | Request-reply |
| **SessionId** | Session identifier | FIFO, grouping |
| **TimeToLive** | Message expiration | Cleanup |
| **ScheduledEnqueueTime** | Delayed delivery | Scheduled processing |
| **ReplyTo** | Reply destination | Request-reply |
| **Label** | Message label/subject | Filtering |

## Advanced Features

### Sessions (FIFO Guaranteed)

```mermaid
flowchart TB
    subgraph "Session-Enabled Queue"
        S1[Session: OrderA]
        S2[Session: OrderB]
        S3[Session: OrderC]
    end
    
    SENDER[Sender] -->|SessionId=OrderA| S1
    SENDER -->|SessionId=OrderB| S2
    
    S1 --> R1[Receiver 1<br/>Locks OrderA]
    S2 --> R2[Receiver 2<br/>Locks OrderB]
    
    style S1 fill:#0078D4,color:#fff
    style S2 fill:#0078D4,color:#fff
    style S3 fill:#0078D4,color:#fff
```

**Session Benefits:**
- Guaranteed FIFO within session
- Session state management
- Exclusive receiver per session
- Session locking

### Dead-Letter Queue (DLQ)

The Dead-Letter Queue is a secondary sub-queue that holds messages which cannot be delivered or processed successfully. Every queue and topic subscription has an associated DLQ that is automatically created and managed by Service Bus.

```mermaid
flowchart TB
    Q[Main Queue] --> PROC{Processing}
    PROC -->|Success| COMPLETE[Complete]
    PROC -->|Fail| RETRY{Retry?}
    RETRY -->|Yes| Q
    RETRY -->|Max Retries| DLQ[Dead-Letter Queue]
    
    DLQ --> ADMIN[Admin Review]
    ADMIN -->|Resubmit| Q
    
    style Q fill:#0078D4,color:#fff
    style DLQ fill:#FF6B6B,color:#fff
```

#### When Messages Are Sent to DLQ

**Automatic Dead-Lettering:**

| Scenario | Description | Configuration |
|----------|-------------|---------------|
| **Max Delivery Count Exceeded** | Message received and abandoned more times than `MaxDeliveryCount` (default: 10) | Set `MaxDeliveryCount` on queue/subscription |
| **Message Expiration (TTL)** | Message exceeds its TimeToLive value | Enable `DeadLetteringOnMessageExpiration` |
| **Subscription Filter Errors** | Filter cannot be evaluated on a subscription | Enable `EnableDeadLetteringOnFilterEvaluationExceptions` |
| **Session State Size Exceeded** | Session state exceeds 1 MB limit | N/A - system enforced |

**Explicit Dead-Lettering:**
- Application code calls `DeadLetterAsync()` when business logic determines a message cannot be processed

#### Dead-Letter Reason Codes

| Reason | Description |
|--------|-------------|
| `MaxDeliveryCountExceeded` | Too many delivery attempts failed |
| `TTLExpiredException` | Message time-to-live expired |
| `HeaderSizeExceeded` | Message headers too large |
| `SessionIdIsMissing` | Session-enabled queue received non-session message |

#### Accessing the Dead-Letter Queue

The DLQ path follows the format: `<queue-name>/$deadletterqueue` or `<topic>/<subscription>/$deadletterqueue`

```csharp
// Create a receiver for the dead-letter queue
var dlqPath = ServiceBusReceiverClient.FormatDeadLetterPath(queueName);
var dlqReceiver = client.CreateReceiver(dlqPath);

// Receive dead-lettered messages
var messages = await dlqReceiver.ReceiveMessagesAsync(maxMessages: 10);

foreach (var message in messages)
{
    // Inspect dead-letter reason
    var reason = message.DeadLetterReason;
    var description = message.DeadLetterErrorDescription;
    
    Console.WriteLine($"DLQ Reason: {reason}, Description: {description}");
    
    // Process or resubmit the message
    await dlqReceiver.CompleteMessageAsync(message);
}
```

#### Explicitly Dead-Lettering Messages

```csharp
// Dead-letter a message with custom reason
await receiver.DeadLetterMessageAsync(
    message,
    deadLetterReason: "InvalidPayload",
    deadLetterErrorDescription: "Message format was invalid - missing required fields"
);
```

#### Best Practices for DLQ Management

- **Monitor DLQ regularly** - Set up alerts for messages in DLQ
- **Implement DLQ processors** - Create background jobs to handle dead-lettered messages
- **Log dead-letter reasons** - Track patterns to fix root causes
- **Set appropriate MaxDeliveryCount** - Balance between retry attempts and quick failure detection
- **Consider automatic cleanup** - Set TTL on DLQ messages to prevent unlimited growth

### Message Forwarding

```mermaid
flowchart LR
    Q1[Queue 1] -->|Auto-Forward| Q2[Queue 2]
    T1[Topic] --> SUB[Subscription]
    SUB -->|Auto-Forward| Q3[Target Queue]
    
    style Q1 fill:#0078D4,color:#fff
    style Q2 fill:#0078D4,color:#fff
    style Q3 fill:#0078D4,color:#fff
```

### Scheduled Messages

```csharp
// Schedule a message for future delivery
var message = new ServiceBusMessage("Scheduled content");
var scheduledTime = DateTimeOffset.UtcNow.AddHours(1);

long sequenceNumber = await sender.ScheduleMessageAsync(
    message, 
    scheduledTime
);

// Cancel if needed
await sender.CancelScheduledMessageAsync(sequenceNumber);
```

### Duplicate Detection

```mermaid
flowchart TB
    M1[Message ID: ABC] --> Q[Queue]
    M2[Message ID: ABC] --> Q
    M3[Message ID: DEF] --> Q
    
    Q --> D{Duplicate<br/>Detection}
    D -->|First ABC| ACCEPT1[Accept]
    D -->|Duplicate ABC| REJECT[Reject/Ignore]
    D -->|DEF| ACCEPT2[Accept]
    
    style Q fill:#0078D4,color:#fff
    style REJECT fill:#FF6B6B,color:#fff
```

## Subscription Filters

### Filter Types

```mermaid
graph TB
    T[Topic] --> MSG[Incoming Message]
    MSG --> F1{SQL Filter}
    MSG --> F2{Correlation Filter}
    MSG --> F3{True Filter}
    
    F1 -->|"Priority > 5"| SUB1[High Priority Sub]
    F2 -->|"Type = 'Order'"| SUB2[Order Sub]
    F3 -->|All Messages| SUB3[Audit Sub]
    
    style T fill:#50E6FF
```

### SQL Filter Examples

```sql
-- Filter by custom property
Priority > 5

-- Filter by system property
sys.Label = 'Important'

-- Complex filter
Priority > 3 AND Region = 'US' AND Amount > 1000

-- NULL check
OrderType IS NOT NULL
```

### Correlation Filter

```csharp
// Efficient property-based filtering
var filter = new CorrelationRuleFilter
{
    ContentType = "application/json",
    Subject = "orders",
    ApplicationProperties =
    {
        { "Priority", "High" },
        { "Region", "US" }
    }
};
```

## Service Tiers

| Feature | Basic | Standard | Premium |
|---------|-------|----------|---------|
| **Queues** | ✓ | ✓ | ✓ |
| **Topics** | ✗ | ✓ | ✓ |
| **Max message size** | 256 KB | 256 KB | 100 MB |
| **Transactions** | ✗ | ✓ | ✓ |
| **Sessions** | ✗ | ✓ | ✓ |
| **Duplicate detection** | ✗ | ✓ | ✓ |
| **VNet integration** | ✗ | ✗ | ✓ |
| **Dedicated resources** | ✗ | ✗ | ✓ |
| **Geo-DR** | ✗ | ✓ | ✓ |

### Premium Tier Benefits

```mermaid
graph TB
    subgraph "Premium Namespace"
        MU[Messaging Units]
        VNET[VNet Integration]
        PE[Private Endpoints]
        CMEK[Customer-Managed Keys]
        LARGE[Large Messages 100MB]
    end
    
    style MU fill:#0078D4,color:#fff
    style VNET fill:#0078D4,color:#fff
    style PE fill:#0078D4,color:#fff
```

## Receive Modes

### Receive and Delete

```mermaid
sequenceDiagram
    participant Receiver
    participant Queue
    
    Receiver->>Queue: Receive
    Queue->>Receiver: Message (deleted immediately)
    Note over Queue: Message removed<br/>No retry possible
```

### Peek Lock (Recommended)

```mermaid
sequenceDiagram
    participant Receiver
    participant Queue
    
    Receiver->>Queue: Receive (PeekLock)
    Queue->>Receiver: Message + Lock Token
    Note over Queue: Message locked
    
    alt Success
        Receiver->>Queue: Complete(token)
        Note over Queue: Message deleted
    else Failure
        Receiver->>Queue: Abandon(token)
        Note over Queue: Message unlocked<br/>Available for redelivery
    end
```

## Security

### Authentication Options

| Method | Description | Use Case |
|--------|-------------|----------|
| **Managed Identity** | Azure AD identity | Azure resources |
| **SAS Token** | Shared Access Signature | Specific permissions |
| **Azure AD** | Role-based access | Enterprise security |
| **Connection String** | Namespace-level access | Development |

### Built-in RBAC Roles

| Role | Permissions |
|------|-------------|
| **Owner** | Full access including IAM |
| **Data Owner** | Full data access |
| **Data Sender** | Send messages only |
| **Data Receiver** | Receive messages only |

### Network Security

```mermaid
graph TB
    subgraph "Network Security Options"
        FW[IP Firewall Rules]
        VNET[VNet Service Endpoints]
        PE[Private Endpoints]
    end
    
    subgraph "Premium Features"
        VNET --> NS[Namespace]
        PE --> NS
    end
    
    style NS fill:#0078D4,color:#fff
    style PE fill:#50E6FF
```

## Integration Patterns

### Pattern: Load Leveling

```mermaid
flowchart LR
    subgraph "Variable Load"
        C1[Client 1]
        C2[Client 2]
        C3[Client N]
    end
    
    C1 & C2 & C3 --> Q[Service Bus Queue]
    
    Q --> W1[Worker 1]
    Q --> W2[Worker 2]
    
    subgraph "Steady Processing"
        W1
        W2
    end
    
    style Q fill:#0078D4,color:#fff
```

### Pattern: Competing Consumers

```mermaid
flowchart TB
    Q[Queue] --> W1[Worker 1]
    Q --> W2[Worker 2]
    Q --> W3[Worker 3]
    
    W1 & W2 & W3 --> DB[(Database)]
    
    style Q fill:#0078D4,color:#fff
```

### Pattern: Event Distribution

```mermaid
flowchart TB
    PUB[Publisher] --> T[Topic]
    T --> S1[Subscription: Email]
    T --> S2[Subscription: SMS]
    T --> S3[Subscription: Audit]
    
    S1 --> EMAIL[Email Service]
    S2 --> SMS[SMS Service]
    S3 --> LOG[Log Storage]
    
    style T fill:#50E6FF
```

## Code Examples

### Sending Messages (C#)

```csharp
using Azure.Messaging.ServiceBus;

await using var client = new ServiceBusClient(connectionString);
await using var sender = client.CreateSender("my-queue");

// Send single message
var message = new ServiceBusMessage("Hello, Service Bus!");
message.ApplicationProperties["Priority"] = "High";
await sender.SendMessageAsync(message);

// Send batch
var batch = await sender.CreateMessageBatchAsync();
batch.TryAddMessage(new ServiceBusMessage("Message 1"));
batch.TryAddMessage(new ServiceBusMessage("Message 2"));
await sender.SendMessagesAsync(batch);
```

### Receiving Messages (C#)

```csharp
await using var client = new ServiceBusClient(connectionString);
await using var processor = client.CreateProcessor("my-queue", new ServiceBusProcessorOptions());

processor.ProcessMessageAsync += async args =>
{
    var body = args.Message.Body.ToString();
    Console.WriteLine($"Received: {body}");
    
    // Complete the message
    await args.CompleteMessageAsync(args.Message);
};

processor.ProcessErrorAsync += args =>
{
    Console.WriteLine($"Error: {args.Exception}");
    return Task.CompletedTask;
};

await processor.StartProcessingAsync();
```

## Best Practices

### Design

| Practice | Description |
|----------|-------------|
| **Use sessions for FIFO** | When order matters |
| **Enable duplicate detection** | For at-most-once delivery |
| **Set appropriate TTL** | Prevent queue buildup |
| **Use correlation IDs** | For request tracking |
| **Design for idempotency** | Handle redelivery |

### Performance

| Practice | Description |
|----------|-------------|
| **Use batching** | Group sends/receives |
| **Use prefetch** | Reduce round trips |
| **Choose Premium** | For consistent latency |
| **Use AMQP** | Better throughput than HTTP |
| **Connection pooling** | Reuse clients |

### Operations

| Practice | Description |
|----------|-------------|
| **Monitor DLQ** | Alert on dead letters |
| **Set up alerts** | Track queue depth |
| **Use Geo-DR** | For disaster recovery |
| **Auto-forward** | Simplify routing |

## Monitoring

### Key Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| **Active Messages** | Pending messages | Based on capacity |
| **Dead-lettered Messages** | Failed messages | > 0 |
| **Incoming Messages** | Rate of arrival | Baseline |
| **Outgoing Messages** | Rate of processing | Baseline |
| **Server Errors** | Backend failures | > 0 |

```mermaid
flowchart LR
    SB[Service Bus] --> AM[Azure Monitor]
    AM --> LA[Log Analytics]
    AM --> ALERT[Alerts]
    AM --> DASH[Dashboard]
    
    style SB fill:#0078D4,color:#fff
    style AM fill:#50E6FF
```

## Hands-On Lab Ideas

1. **Create queue and send/receive** - Basic messaging
2. **Implement pub-sub** - Topics and subscriptions
3. **Build retry pattern** - DLQ handling
4. **Session-based ordering** - FIFO processing
5. **Subscription filtering** - Message routing

---

## References

- [Azure Service Bus Documentation](https://learn.microsoft.com/en-us/azure/service-bus-messaging/)
- [Service Bus Messaging Overview](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview)
- [Queues, Topics, and Subscriptions](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions)
- [Service Bus Premium Tier](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-premium-messaging)
- [Service Bus .NET SDK](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues)
- [Best Practices](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-performance-improvements)
- [Message Sessions](https://learn.microsoft.com/en-us/azure/service-bus-messaging/message-sessions)
