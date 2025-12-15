# Azure Data Factory

## Overview

Azure Data Factory (ADF) is a cloud-based data integration service that allows you to create data-driven workflows (pipelines) for orchestrating and automating data movement and data transformation. It's the ETL/ELT service of choice in Azure for enterprise-scale data integration scenarios.

Data Factory enables you to ingest data from various sources, transform it at scale, and load it into data stores for analytics, reporting, and machine learning.

## Core Concepts

```mermaid
graph TB
    subgraph "Azure Data Factory"
        PIPELINE[Pipeline]
        ACTIVITY[Activities]
        DATASET[Datasets]
        LS[Linked Services]
        TRIGGER[Triggers]
        IR[Integration Runtime]
    end
    
    TRIGGER --> PIPELINE
    PIPELINE --> ACTIVITY
    ACTIVITY --> DATASET
    DATASET --> LS
    LS --> IR
    
    style PIPELINE fill:#0078D4,color:#fff
```

### Component Relationships

```mermaid
flowchart TB
    subgraph "Data Factory Components"
        subgraph "Compute"
            IR1[Azure IR]
            IR2[Self-Hosted IR]
            IR3[Azure-SSIS IR]
        end
        
        subgraph "Connection"
            LS1[Source Linked Service]
            LS2[Sink Linked Service]
        end
        
        subgraph "Data"
            DS1[Source Dataset]
            DS2[Sink Dataset]
        end
        
        subgraph "Logic"
            PIPE[Pipeline]
            COPY[Copy Activity]
            DF[Data Flow]
        end
    end
    
    IR1 & IR2 --> LS1 & LS2
    LS1 --> DS1
    LS2 --> DS2
    DS1 --> COPY
    COPY --> DS2
    PIPE --> COPY & DF
    
    style PIPE fill:#0078D4,color:#fff
```

## Pipelines and Activities

### Pipeline Structure

```mermaid
flowchart LR
    subgraph "Pipeline"
        A1[Activity 1] --> A2[Activity 2]
        A2 --> A3[Activity 3]
        A3 --> A4[Activity 4]
    end
    
    PARAM[Parameters] --> A1
    VAR[Variables] --> A2
    
    style A1 fill:#0078D4,color:#fff
    style A2 fill:#0078D4,color:#fff
    style A3 fill:#0078D4,color:#fff
    style A4 fill:#0078D4,color:#fff
```

### Activity Types

| Category | Activities | Description |
|----------|------------|-------------|
| **Data Movement** | Copy | Move data between stores |
| **Data Transformation** | Data Flow, HDInsight, Databricks | Transform data at scale |
| **Control Flow** | If, ForEach, Switch, Until | Pipeline logic |
| **External** | Web, Azure Function, Logic App | Call external services |
| **Lookup** | Lookup, Get Metadata | Query data/metadata |
| **Iteration** | ForEach, Until | Loop constructs |

### Activity Dependencies

```mermaid
flowchart TB
    A1[Extract Data] -->|On Success| A2[Transform]
    A1 -->|On Failure| ERR[Log Error]
    A2 -->|On Success| A3[Load]
    A2 -->|On Completion| NOTIFY[Send Notification]
    
    style A1 fill:#0078D4,color:#fff
    style A2 fill:#0078D4,color:#fff
    style A3 fill:#0078D4,color:#fff
    style ERR fill:#FF6B6B,color:#fff
```

**Dependency Conditions:**
- **On Success** - Runs if previous succeeded
- **On Failure** - Runs if previous failed
- **On Completion** - Always runs
- **On Skipped** - Runs if previous was skipped

## Data Integration Patterns

### Pattern 1: ETL Pipeline

```mermaid
flowchart LR
    subgraph "Extract"
        SRC1[(SQL Server)]
        SRC2[(Oracle)]
        SRC3[Files]
    end
    
    subgraph "Transform"
        DF[Data Flow]
    end
    
    subgraph "Load"
        DW[(Synapse Analytics)]
    end
    
    SRC1 & SRC2 & SRC3 --> DF --> DW
    
    style DF fill:#0078D4,color:#fff
```

### Pattern 2: ELT Pipeline

```mermaid
flowchart LR
    subgraph "Extract & Load"
        COPY[Copy Activity]
    end
    
    subgraph "Source"
        SRC[(Source DB)]
    end
    
    subgraph "Staging"
        LAKE[(Data Lake)]
    end
    
    subgraph "Transform"
        DB[Databricks]
        SYN[Synapse]
    end
    
    SRC --> COPY --> LAKE
    LAKE --> DB --> DW[(Data Warehouse)]
    LAKE --> SYN --> DW
    
    style COPY fill:#0078D4,color:#fff
```

### Pattern 3: Incremental Load

```mermaid
flowchart TB
    GET[Get Watermark] --> LOOKUP[Lookup New Data]
    LOOKUP --> COPY[Copy New Rows]
    COPY --> UPDATE[Update Watermark]
    
    style GET fill:#50E6FF
    style COPY fill:#0078D4,color:#fff
```

## Copy Activity

### Architecture

```mermaid
flowchart LR
    subgraph "Source"
        SRC[Source Dataset]
        SRC_LS[Linked Service]
    end
    
    subgraph "Copy Activity"
        DIR[DIU Allocation]
        PARALLEL[Parallel Copy]
        STAGING[Staging if needed]
    end
    
    subgraph "Sink"
        SINK[Sink Dataset]
        SINK_LS[Linked Service]
    end
    
    SRC --> SRC_LS --> DIR
    DIR --> PARALLEL --> STAGING --> SINK_LS --> SINK
    
    style DIR fill:#0078D4,color:#fff
```

### Performance Tuning

| Setting | Description | Recommendation |
|---------|-------------|----------------|
| **Data Integration Units** | Parallel compute | Auto or 2-256 DIU |
| **Parallel Copy** | Concurrent operations | Based on source |
| **Staging** | Use blob for PolyBase | Enable for Synapse |
| **Source Partitions** | Physical partitions | Match source |

### Supported Connectors

```mermaid
graph TB
    subgraph "Cloud Sources"
        AZURE[Azure Services]
        AWS[Amazon Services]
        GCP[Google Cloud]
        SAAS[SaaS Apps]
    end
    
    subgraph "On-Premises"
        SQL[SQL Server]
        ORACLE[Oracle]
        SAP[SAP]
        FILES[File Systems]
    end
    
    ADF[Data Factory] --> AZURE & AWS & GCP & SAAS
    ADF --> SQL & ORACLE & SAP & FILES
    
    style ADF fill:#0078D4,color:#fff
```

**90+ Connectors Including:**
- Azure (Blob, ADLS, SQL, Cosmos DB, Synapse)
- Databases (SQL Server, Oracle, MySQL, PostgreSQL)
- SaaS (Salesforce, Dynamics, SAP, ServiceNow)
- Files (Parquet, JSON, CSV, Avro, ORC)
- Big Data (Hadoop, Spark, Databricks)

## Data Flows

### Mapping Data Flows

Visual data transformation at scale using Spark.

```mermaid
flowchart LR
    SRC[Source] --> DERIVE[Derived Column]
    DERIVE --> FILTER[Filter]
    FILTER --> AGG[Aggregate]
    AGG --> JOIN[Join]
    JOIN --> SINK[Sink]
    
    style SRC fill:#90EE90
    style SINK fill:#FF6B6B,color:#fff
```

### Transformation Types

| Category | Transformations |
|----------|-----------------|
| **Schema Modifier** | Derived Column, Select, Cast |
| **Row Modifier** | Filter, Sort, Alter Row, Assert |
| **Multiple Inputs** | Join, Union, Lookup, Exists |
| **Multiple Outputs** | Conditional Split, New Branch |
| **Aggregators** | Aggregate, Window, Pivot, Unpivot |
| **Formatters** | Flatten, Parse, Stringify |

### Data Flow Example

```mermaid
flowchart TB
    subgraph "Data Flow: Order Processing"
        S1[Source: Orders]
        S2[Source: Customers]
        
        D1[Derive: Calculate Total]
        F1[Filter: Active Orders]
        J1[Join: Customer Info]
        A1[Aggregate: By Region]
        
        SK1[Sink: Summary Table]
        SK2[Sink: Detail Table]
    end
    
    S1 --> D1 --> F1
    S2 --> J1
    F1 --> J1 --> A1
    A1 --> SK1
    J1 --> SK2
    
    style S1 fill:#90EE90
    style S2 fill:#90EE90
    style SK1 fill:#FF6B6B,color:#fff
    style SK2 fill:#FF6B6B,color:#fff
```

### Data Flow Performance

| Feature | Description |
|---------|-------------|
| **Core Count** | 8-256 cores per cluster |
| **TTL** | Cluster reuse time |
| **Partitioning** | Round robin, hash, dynamic |
| **Debug Cluster** | Interactive development |

## Integration Runtime

### Runtime Types

```mermaid
graph TB
    subgraph "Integration Runtime Types"
        AZURE[Azure IR]
        SELF[Self-Hosted IR]
        SSIS[Azure-SSIS IR]
    end
    
    AZURE --> CLOUD[Cloud Data Sources]
    SELF --> ONPREM[On-Premises Sources]
    SELF --> PRIVATE[Private Network]
    SSIS --> SSIS_PKG[SSIS Packages]
    
    style AZURE fill:#0078D4,color:#fff
    style SELF fill:#50E6FF
    style SSIS fill:#7719AA,color:#fff
```

### Integration Runtime Comparison

| Feature | Azure IR | Self-Hosted IR | Azure-SSIS IR |
|---------|----------|----------------|---------------|
| **Location** | Azure regions | On-premises | Azure |
| **Use case** | Cloud to cloud | Hybrid | SSIS lift & shift |
| **Managed** | Fully managed | Customer managed | Managed |
| **Network** | Public | Private/hybrid | VNet |
| **Data Flow** | ✓ | ✗ | ✗ |

### Self-Hosted IR Architecture

```mermaid
flowchart TB
    subgraph "Azure"
        ADF[Data Factory]
    end
    
    subgraph "On-Premises Network"
        SHIR[Self-Hosted IR]
        SQL[(SQL Server)]
        FILES[File Server]
        ORACLE[(Oracle)]
    end
    
    ADF <-->|HTTPS| SHIR
    SHIR --> SQL & FILES & ORACLE
    
    style ADF fill:#0078D4,color:#fff
    style SHIR fill:#50E6FF
```

## Triggers

### Trigger Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Schedule** | Time-based, cron | Regular ETL jobs |
| **Tumbling Window** | Fixed intervals | Time-series data |
| **Event** | Storage events | File-based processing |
| **Custom Event** | Event Grid | Event-driven |

### Schedule Trigger

```json
{
    "name": "DailyTrigger",
    "type": "ScheduleTrigger",
    "properties": {
        "recurrence": {
            "frequency": "Day",
            "interval": 1,
            "startTime": "2024-01-01T00:00:00Z",
            "timeZone": "UTC"
        },
        "pipelines": [
            {
                "pipelineReference": {
                    "type": "PipelineReference",
                    "referenceName": "DailyETL"
                }
            }
        ]
    }
}
```

### Tumbling Window Trigger

```mermaid
flowchart LR
    W1[Window 1<br/>00:00-01:00] --> W2[Window 2<br/>01:00-02:00]
    W2 --> W3[Window 3<br/>02:00-03:00]
    W3 --> W4[Window N]
    
    W1 --> P1[Pipeline Run 1]
    W2 --> P2[Pipeline Run 2]
    W3 --> P3[Pipeline Run 3]
    
    style W1 fill:#0078D4,color:#fff
    style W2 fill:#0078D4,color:#fff
    style W3 fill:#0078D4,color:#fff
```

**Tumbling Window Features:**
- Retry on failure
- Dependency management
- Backfill historical data
- Concurrent execution control

### Event Trigger

```mermaid
flowchart LR
    BLOB[Blob Created] --> EG[Event Grid]
    EG --> ADF[Data Factory Trigger]
    ADF --> PIPE[Pipeline]
    
    style ADF fill:#0078D4,color:#fff
    style EG fill:#50E6FF
```

## Control Flow

### ForEach Loop

```mermaid
flowchart TB
    LOOKUP[Lookup Files] --> FOREACH{ForEach File}
    FOREACH --> COPY[Copy Activity]
    FOREACH --> COPY
    FOREACH --> COPY
    COPY --> COMPLETE[Complete]
    
    style FOREACH fill:#FFE66D
    style COPY fill:#0078D4,color:#fff
```

### If Condition

```mermaid
flowchart TB
    CHECK[Check Condition] --> IF{If Condition}
    IF -->|True| TRUE_BRANCH[Execute True Branch]
    IF -->|False| FALSE_BRANCH[Execute False Branch]
    
    style IF fill:#FFE66D
```

### Until Loop

```mermaid
flowchart TB
    START[Start] --> CHECK{Until Condition}
    CHECK -->|False| EXECUTE[Execute Activities]
    EXECUTE --> CHECK
    CHECK -->|True| END[End]
    
    style CHECK fill:#FFE66D
```

## Monitoring

### Monitor Dashboard

```mermaid
flowchart LR
    ADF[Data Factory] --> MON[Monitor]
    MON --> RUNS[Pipeline Runs]
    MON --> TRIGGERS[Trigger Runs]
    MON --> IR[Integration Runtime]
    MON --> ALERTS[Alerts]
    
    style MON fill:#0078D4,color:#fff
```

### Key Metrics

| Metric | Description |
|--------|-------------|
| **Pipeline runs** | Success/failure count |
| **Activity runs** | Per-activity status |
| **Trigger runs** | Trigger execution history |
| **IR utilization** | Resource usage |
| **Data flow metrics** | Spark execution details |

### Alerting

```json
{
    "alerts": [
        {
            "name": "PipelineFailure",
            "condition": "Pipeline run failed",
            "action": "Send email, Logic App, Function"
        },
        {
            "name": "LongRunning",
            "condition": "Duration > threshold",
            "action": "Send notification"
        }
    ]
}
```

## Security

### Authentication Methods

| Method | Use Case |
|--------|----------|
| **Managed Identity** | Azure resources |
| **Service Principal** | Cross-tenant |
| **SQL Authentication** | Database access |
| **Key Vault** | Secret management |
| **User-assigned MI** | Shared identity |

### Network Security

```mermaid
graph TB
    subgraph "Network Options"
        VNET[VNet Integration]
        PE[Private Endpoints]
        MG[Managed VNet]
        SHIR[Self-Hosted IR]
    end
    
    ADF[Data Factory] --> VNET
    ADF --> PE
    ADF --> MG
    ADF --> SHIR
    
    style ADF fill:#0078D4,color:#fff
```

### Managed Virtual Network

```mermaid
flowchart TB
    subgraph "Managed VNet"
        AIR[Azure IR]
        PE1[Private Endpoint - SQL]
        PE2[Private Endpoint - Storage]
        PE3[Private Endpoint - Key Vault]
    end
    
    subgraph "Azure Resources"
        SQL[(Azure SQL)]
        BLOB[Blob Storage]
        KV[Key Vault]
    end
    
    AIR --> PE1 --> SQL
    AIR --> PE2 --> BLOB
    AIR --> PE3 --> KV
    
    style AIR fill:#0078D4,color:#fff
```

## CI/CD and DevOps

### Git Integration

```mermaid
flowchart LR
    DEV[Development] --> GIT[Git Repository]
    GIT --> COLLAB[Collaboration Branch]
    COLLAB --> PR[Pull Request]
    PR --> MAIN[Main Branch]
    MAIN --> RELEASE[Release Pipeline]
    RELEASE --> PROD[Production]
    
    style GIT fill:#0078D4,color:#fff
```

### Deployment Workflow

```mermaid
flowchart TB
    subgraph "Development"
        DEV_ADF[Dev Data Factory]
        DEV_GIT[Feature Branch]
    end
    
    subgraph "Build"
        ARM[ARM Templates]
        PARAM[Parameter Files]
    end
    
    subgraph "Release"
        TEST[Test Environment]
        PROD[Production]
    end
    
    DEV_ADF --> DEV_GIT --> ARM
    ARM --> TEST --> PROD
    PARAM --> TEST & PROD
    
    style ARM fill:#50E6FF
```

## Best Practices

### Design

| Practice | Description |
|----------|-------------|
| **Modular pipelines** | Reusable components |
| **Use parameters** | Environment flexibility |
| **Incremental loads** | Efficient processing |
| **Error handling** | Proper failure paths |
| **Naming conventions** | Consistent naming |

### Performance

| Practice | Description |
|----------|-------------|
| **Right-size DIUs** | Balance cost/performance |
| **Use staging** | For Synapse/PolyBase |
| **Partition data** | Parallel processing |
| **Optimize queries** | Source optimization |
| **Cache lookup results** | Reduce round trips |

### Operations

| Practice | Description |
|----------|-------------|
| **Enable Git** | Version control |
| **Monitor pipelines** | Proactive alerting |
| **Document pipelines** | Annotations |
| **Test thoroughly** | Debug before publish |

## Architecture Patterns

### Pattern: Data Lake Architecture

```mermaid
flowchart TB
    subgraph "Sources"
        SRC1[(Operational DB)]
        SRC2[Files]
        SRC3[APIs]
    end
    
    subgraph "Data Factory"
        ADF[Orchestration]
    end
    
    subgraph "Data Lake"
        RAW[Raw Zone]
        CURATED[Curated Zone]
        CONSUME[Consumption Zone]
    end
    
    subgraph "Analytics"
        SYN[Synapse]
        PBI[Power BI]
    end
    
    SRC1 & SRC2 & SRC3 --> ADF
    ADF --> RAW --> CURATED --> CONSUME
    CONSUME --> SYN --> PBI
    
    style ADF fill:#0078D4,color:#fff
```

### Pattern: Hybrid Integration

```mermaid
flowchart LR
    subgraph "On-Premises"
        ONPREM[(On-Prem SQL)]
        SHIR[Self-Hosted IR]
    end
    
    subgraph "Azure"
        ADF[Data Factory]
        ADLS[(Data Lake)]
        SYN[(Synapse)]
    end
    
    ONPREM --> SHIR --> ADF
    ADF --> ADLS --> SYN
    
    style ADF fill:#0078D4,color:#fff
    style SHIR fill:#50E6FF
```

## Pricing Considerations

### Cost Components

| Component | Billing |
|-----------|---------|
| **Pipeline Orchestration** | Per activity run |
| **Data Movement** | Per DIU-hour |
| **Data Flow** | Per cluster hour |
| **Integration Runtime** | Per hour (Self-hosted free) |
| **SSIS IR** | Per node hour |

### Cost Optimization Tips

1. **Right-size compute** - Match DIUs to data volume
2. **Use TTL for clusters** - Reuse data flow clusters
3. **Schedule off-peak** - Lower costs during off-hours
4. **Monitor usage** - Identify optimization opportunities

## Hands-On Lab Ideas

1. **Build ETL pipeline** - Copy and transform data
2. **Incremental load** - Watermark-based loading
3. **Data flow development** - Visual transformations
4. **Event-driven pipeline** - Blob trigger processing
5. **Hybrid integration** - Self-hosted IR setup

---

## References

- [Azure Data Factory Documentation](https://learn.microsoft.com/en-us/azure/data-factory/)
- [Data Factory Overview](https://learn.microsoft.com/en-us/azure/data-factory/introduction)
- [Pipelines and Activities](https://learn.microsoft.com/en-us/azure/data-factory/concepts-pipelines-activities)
- [Mapping Data Flows](https://learn.microsoft.com/en-us/azure/data-factory/concepts-data-flow-overview)
- [Integration Runtime](https://learn.microsoft.com/en-us/azure/data-factory/concepts-integration-runtime)
- [Copy Activity Performance](https://learn.microsoft.com/en-us/azure/data-factory/copy-activity-performance)
- [CI/CD in Data Factory](https://learn.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery)
- [Data Factory Pricing](https://azure.microsoft.com/en-us/pricing/details/data-factory/)
