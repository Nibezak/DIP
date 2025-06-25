# 📊 DIP: Data Ingestion Pipeline

**DIP** (Data Ingestion Pipeline) is a modular, scalable system designed to collect, process, and deliver social‑media analytics to downstream machine‑learning services. Although its first use case targets TikTok and Instagram metrics for abaya‑seller content optimization, its architecture is adaptable to any streaming or batch data source.

---

## 🔍 Overview

DIP automates the end‑to‑end flow:

1. **Data Connectors** fetch raw metrics from APIs.
2. **Data Lake** persists raw, unmodified payloads for audit and replay.
3. **Processing & Feature Store** normalizes, enriches, and stores cleaned data.
4. **Orchestration & Scheduling** ensures reliability, retries, and lineage.
5. **Monitoring & Alerting** tracks health, latency, and data quality.

This workflow guarantees fresh, accurate inputs for any ML or analytics consumer.

---

## 🏗 Architecture

The DIP architecture is designed for high throughput, low latency, and strong reliability, with clear separation of concerns:

```mermaid
flowchart TB
  subgraph Connectors
    A[API Clients]
    A -->|OAuth2 / API Key| B[Ingestion Gateway]
  end

  subgraph Ingestion
    B --> C[Message Broker (Kafka / PubSub)]
    C --> D[Raw Ingest Topic]
  end

  subgraph Storage
    D --> E[Raw Zone (Object Storage: S3/GCS)]
    E -->|Retention Policies| F[Archive Zone]
  end

  subgraph Processing
    C --> G[Streaming ETL (Flink / Spark Structured Streaming)]
    G --> H[Bronze Zone (Parquet)]
    H --> I[Delta Lake / Iceberg]
    I --> J[Silver Zone (Cleaned & Typed Tables)]
    J --> K[Feature Store (Redis / Feast)]
  end

  subgraph Orchestration & Governance
    L[Airflow / Prefect]
    L --> A
    L --> G
    L --> M[Schema Registry & Data Catalog]
    M --> G
  end

  subgraph Monitoring & Observability
    N[Prometheus Metrics]
    O[OpenTelemetry Traces]
    P[Grafana / Kibana Dashboards]
    A --> N
    G --> N
    G --> O
  end

  subgraph Security & Compliance
    Q[Vault / KMS for Secrets]
    R[IAM & RBAC]
    B --> Q
    A --> R
    E --> R
    I --> R
  end

  subgraph Downstream Consumers
    K --> S[ML Training Jobs (Kubeflow / MLflow)]
    K --> T[Real-time API (FastAPI)]
    S --> U[Model Registry]
    T --> V[BI Dashboard / Alerts]
  end
```

Key aspects:

* **Decoupled Ingestion Gateway** for consistent auth, rate-limit handling, and backpressure signaling.
* **Kafka / PubSub** ensures fault-tolerant, replayable ingestion streams with configurable retention.
* **Multi-zone Storage** (Raw ▶ Bronze ▶ Silver) enforces schema evolution, data validation, idempotent writes, and partition pruning for performance.
* **Streaming ETL** provides near real-time feature availability, implements watermarking for late-arriving data, and enforces exactly-once semantics when supported.
* **Feature Store** (e.g. Feast) separates online (low-latency Redis) and offline (BigQuery/Snowflake) serving layers.
* **Orchestration & Governance** via Airflow/Prefect connects ingestion, ETL, and training, while leveraging a Schema Registry for compatibility checks.
* **Observability** with Prometheus and OpenTelemetry for end-to-end metrics, traces, and logs; Grafana/Kibana for dashboards and alerting.
* **Security** through centralized secret management, fine-grained IAM/RBAC, encryption-at-rest, and audit logs.

---

## ⚙️ Workflow Steps

1. **Connect**
   • Authenticate and fetch metrics (e.g. post views, likes, comments, audience demographics) via TikTok/Instagram APIs.
2. **Persist**
   • Write raw payloads to object storage, partitioned by date and source.
3. **Process**
   • Schedule ETL jobs to normalize timestamps, extract hashtags, compute engagement ratios, and handle missing fields.
   • Store enriched records in a low‑latency feature store for consumption.
4. **Serve & Consume**
   • Expose feature endpoints via microservice for both batch re‑training and real‑time scoring.
5. **Monitor**
   • Track connector success rates, ETL durations, and data freshness.
   • Trigger alerts on failures or SLA breaches.

---

## 📁 Components

* **Connectors**: Python clients for each social API, with pagination and rate‑limit handling.
* **Data Lake**: Immutable raw store for audit and replay scenarios.
* **ETL Workers**: Kafka‑driven or batch Spark jobs transforming raw to structured.
* **Feature Store**: Redis for real‑time, Apache Hudi for batch serving.
* **Orchestrator**: Airflow or Prefect DAGs managing dependencies and retries.
* **Monitoring**: Prometheus exporters and Grafana dashboards.
* **Microservices**: FastAPI endpoints delivering features to ML and BI layers.

---

## 📈 Adaptability & Scaling

* **Pluggable Connectors**: Add new APIs by implementing a standard interface.
* **Storage Agnostic**: Swap S3 for GCS or HDFS with minimal config.
* **Auto‑Scaling**: Containerized workers scale on message queue depth or batch backlog.

---

