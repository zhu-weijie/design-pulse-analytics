# Pulse Analytics

This is requirements of system design for Data Pipelines

### In a food delivery platform, management would like dashboards summarising key user metrics such as conversion funnels from marketing campaigns, orders, and key logistics metrics such as delivery time end to end

- From separate application systems: orders, logistics, promotions, payments
- Third party vendor data such as customer support logs, Google Analytics, photos

### Performance targets

- Throughput: data volume of x TB raw data daily, to be completed in 6 hours from start to finish
- Latency
- Error rate
- Alerts and pipeline monitoring
- Budget of $xk monthly

### Issue:

- Decentralised analytics: everyone pulls their own data and does their own transformation and
- visualisation
- No central knowledge base / definitions of key metrics

### **Complete Logical View: C4 Component Diagram**

This diagram consolidates all the logical components and their interactions that we designed throughout the process. It illustrates the four primary information flows within the system:

1.  **Primary Data Flow:** The main path for business data, from sources to the data warehouse.
2.  **Control Flow:** How the orchestrator manages and triggers the data processing tasks.
3.  **Metadata Flow:** How the data catalog discovers and indexes information about the data.
4.  **Monitoring Flow:** How the platform is observed and how operators are alerted to problems.

```mermaid
C4Component
  title Complete Logical View: Pulse Analytics Platform

  %% Define Actors (Users)
  Person(biz_user, "Business User / Manager", "Views dashboards to make strategic decisions.")
  Person(analyst, "Data Analyst", "Discovers datasets, understands lineage, and performs ad-hoc queries.")
  Person(operator, "Pipeline Operator", "Monitors pipeline health and responds to alerts.")

  %% Define External Source Systems
  System_Boundary(sources, "Upstream Systems") {
    System(internal_dbs, "Internal Application Databases", "e.g., Orders, Logistics, Payments")
    System(third_party, "Third-Party Systems", "e.g., Google Analytics, Support Logs")
  }

  %% Define the Core Data Platform
  System_Boundary(platform, "Pulse Analytics Platform") {
    %% Data Flow Components (Arranged Left-to-Right)
    Component(ingestion, "Ingestion Services", "DMS, AppFlow, Lambda", "Handles ingestion from all internal and external sources.")
    Component(raw_store, "Raw Data Store (Data Lake)", "Amazon S3", "Stores raw, immutable data from all sources.")
    Component(processing, "Data Validation & Transformation Engine", "Docker with PySpark & Great Expectations", "Validates, cleans, transforms, and applies business logic.")
    Component(warehouse, "Data Warehouse", "Amazon Redshift", "Stores structured, validated, and analysis-ready data.")

    %% User-Facing Application Components
    Component(bi_platform, "Business Intelligence Platform", "Amazon QuickSight", "Provides dashboards and self-service analytics.")
    Component(catalog, "Data Catalog", "Amundsen on ECS", "Provides a central, searchable knowledge base for all data assets.")

    %% Cross-Cutting / Control Plane Components
    Component(orchestrator, "Pipeline Orchestrator", "Apache Airflow on ECS", "Schedules, orchestrates, and monitors the entire data workflow.")
    Component(monitoring, "Monitoring & Alerting System", "Amazon CloudWatch & SNS", "Collects telemetry (logs, metrics) and sends alerts on failures.")
  }

  %% --- Define Relationships ---

  %% Primary Data Flow
  Rel(internal_dbs, ingestion, "Provides data to")
  Rel(third_party, ingestion, "Provides data to")
  Rel(ingestion, raw_store, "Writes raw data to")
  Rel(raw_store, processing, "Reads raw data from")
  Rel(processing, warehouse, "Writes validated, transformed data to")
  Rel(warehouse, bi_platform, "Serves data to")
  Rel(bi_platform, biz_user, "Delivers dashboards and insights to")

  %% Control Flow (Orchestration)
  Rel(orchestrator, ingestion, "Triggers and monitors")
  Rel(orchestrator, processing, "Triggers and monitors")

  %% Metadata Flow (Data Discovery)
  Rel(catalog, raw_store, "Pulls metadata from")
  Rel(catalog, warehouse, "Pulls metadata from")
  Rel(analyst, catalog, "Uses to discover and understand data")

  %% Monitoring & Alerting Flow
  Rel(orchestrator, monitoring, "Sends logs & metrics to")
  Rel(processing, monitoring, "Sends logs & metrics to")
  Rel(monitoring, operator, "Sends alerts to")
```

#### **Complete Physical View: AWS Deployment Diagram**

#### **View 1: The Core Data Flow (For Data Engineers)**

This view focuses exclusively on the main data pipeline, showing how business data moves from source to analytics.

```mermaid
graph TD
    %% Actors
    subgraph "Users"
        biz_user["Business User"]
    end

    %% Cloud
    subgraph "AWS Cloud"
        %% Trigger
        EventBridge["Amazon EventBridge"]

        %% Control & Execution
        subgraph "Control & Execution Plane"
            Airflow_Worker["Container: Airflow Worker"]
            SparkTask["Container: Spark Job<br>with Great Expectations"]
        end

        %% Data & BI
        subgraph "Data & BI Plane"
            Ingestion["Ingestion Services"]
            S3["S3 Bucket (Raw Data Lake)"]
            Redshift["Amazon Redshift (Warehouse)"]
            QuickSight["Amazon QuickSight"]
        end
    end

    %% Relationships
    EventBridge -- "Triggers" --> Airflow_Worker
    Airflow_Worker -. "Launches" .-> SparkTask
    Airflow_Worker -. "Triggers" .-> Ingestion

    Ingestion -- "Writes Raw Data" --> S3
    S3 -- "Reads Raw Data" --> SparkTask
    SparkTask -- "Writes Transformed Data" --> Redshift
    Redshift -- "Serves Data" --> QuickSight
    QuickSight -- "Delivers Dashboards" --> biz_user
```

#### **View 2: The Data Discovery Flow (For Data Analysts)**

This view focuses on how an analyst uses the platform, completely hiding the complex data pipeline.

```mermaid
graph TD
    %% Actor
    subgraph "Users"
        analyst["Data Analyst"]
    end

    %% Cloud
    subgraph "AWS Cloud"
        %% Catalog Services
        subgraph "Amundsen Data Catalog (on ECS)"
            Amundsen_Web["Container: Amundsen Web UI"]
            Amundsen_Meta["Container: Amundsen Metadata"]
            Neo4j["Container: Neo4j (Graph DB)"]
        end

        %% Data Sources for Metadata
        subgraph "Metadata Sources"
            GlueCatalog["AWS Glue Data Catalog"]
            Redshift["Amazon Redshift"]
        end
    end

    %% Relationships
    analyst -- "Discovers data via" --> Amundsen_Web
    Amundsen_Web -- "Queries" --> Amundsen_Meta
    Amundsen_Meta -- "Pulls metadata from" --> GlueCatalog
    Amundsen_Meta -- "Pulls metadata from" --> Redshift
    Amundsen_Meta -- "Stores graph in" --> Neo4j
```
