# 📊 DIP: Data Ingestion Pipeline

**DIP** (Data Ingestion Pipeline) is a modular, scalable system designed to collect, process, and deliver social media analytics to downstream machine-learning services. Although its first use case targets TikTok and Instagram metrics for abaya-seller content optimization, its architecture is adaptable to any streaming or batch data source.

---

## 🔍 Overview

DIP automates the end-to-end flow:

1. **Data Connectors** fetch raw metrics from APIs
2. **Data Lake** persists raw, unmodified payloads for audit and replay
3. **Processing & Feature Store** normalizes, enriches, and stores cleaned data
4. **Orchestration & Scheduling** ensures reliability, retries, and lineage
5. **Monitoring & Alerting** tracks health, latency, and data quality

This workflow guarantees fresh, accurate inputs for any ML or analytics consumer.

---

## 🏗 Architecture

```mermaid
flowchart TB
  subgraph Connectors
    A[API Clients]
    A -- OAuth2/API Key --> B[Ingestion Gateway]
  end

  subgraph Ingestion
    B --> C[Message Broker: Kafka/PubSub]
    C --> D[Raw Topic]
    C --> Z[Dead-Letter Queue]
  end

  subgraph Storage
    D --> E[Raw Zone: S3/GCS]
    E -->|Tiered Retention & Encryption| F[Archive Zone]
  end

  subgraph Processing
    C --> G[Streaming ETL: Flink/Spark]
    G --> H[Bronze Zone: Parquet]
    H --> I[Delta Lake/Iceberg]
    I --> J[Silver Zone: Clean Tables]
    J --> K[Feature Store: Feast/Redis]
  end

  subgraph Orchestration
    L[Airflow/Prefect]
    L --> A
    L --> G
    L --> M[Schema Registry]
    M --> G
  end

  subgraph Observability
    N[Prometheus Metrics]
    O[OpenTelemetry Traces]
    P[Grafana/Kibana]
    A --> N
    G --> N
    G --> O
    N --> P
    O --> P
  end

  subgraph Security
    Q[Vault/KMS]
    R[IAM & RBAC]
    B --> Q
    E --> R
    K --> R
  end

  subgraph Downstream
    K --> S[ML Training: Kubeflow/MLflow]
    K --> T[Realtime API: FastAPI]
    T --> U[BI Dashboard]
  end
