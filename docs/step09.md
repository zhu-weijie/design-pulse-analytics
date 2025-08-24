#### **1. Logical View: C4 Component Diagram**

Monitoring is a cross-cutting concern. This diagram shows a `Monitoring System` that observes our platform and notifies the operations team.

```mermaid
C4Component
  title Component Diagram with Monitoring and Alerting

  Person(operator, "Pipeline Operator", "Monitors pipeline health and responds to alerts")

  System_Boundary(data_platform, "Pulse Analytics Platform") {
    Component(orchestrator, "Pipeline Orchestrator", "Apache Airflow on ECS")
    Component(transformation, "Containerized Transformation", "Docker Container with PySpark")
    Component(monitoring_system, "Monitoring & Alerting System", "Amazon CloudWatch & SNS", "Collects logs/metrics, detects anomalies, and sends alerts")
  }

  Rel(orchestrator, monitoring_system, "Sends logs & metrics to")
  Rel(transformation, monitoring_system, "Sends logs & metrics to")
  Rel(monitoring_system, operator, "Sends alerts to")
```

#### **2. Physical View: Mapping to AWS Resources**

We are adding the core AWS observability services to our stack.

| C4 Component | AWS Resource | Rationale for Selection |
| :--- | :--- | :--- |
| **Internal Databases** | Amazon RDS | Managed, reliable, and secure database service, likely already in use by the source application systems. |
| **Third-Party Systems** | (External SaaS APIs) | External systems providing data via APIs (e.g., Google Analytics). Not an AWS resource, but a key data source. |
| **Ingestion Services** | AWS DMS, AppFlow, Lambda | A "right tool for the job" approach: DMS for managed database replication (CDC), AppFlow for no-code SaaS integration, and Lambda for flexible, custom API polling. |
| **Raw Data Store** | Amazon S3 Bucket | The de facto standard for data lakes due to its virtually unlimited scalability, high durability, low cost, and deep ecosystem integration. |
| **Scheduler** | Amazon EventBridge | A serverless, reliable scheduler for triggering the pipeline via a cron-like expression. Natively integrated with the AWS ecosystem. |
| **Pipeline Orchestrator** | Apache Airflow on Amazon ECS (Fargate) | **(Refactored)** A portable, open-source solution to avoid vendor lock-in. Airflow is the industry standard for data orchestration. ECS with Fargate runs the containers without server management. |
| **Containerized Transformation** | Docker Container (PySpark) on Amazon ECS Task (Fargate) | **(Refactored)** Docker provides a portable, version-controlled environment. PySpark is the industry standard for large-scale data transformation. Running as an ephemeral ECS task is cost-effective. |
| **Data Warehouse** | Amazon Redshift | A fully managed, petabyte-scale columnar data warehouse optimized for high-performance business intelligence and analytical queries. |
| **Business Intelligence Platform** | Amazon QuickSight | A serverless, cloud-native BI service that integrates seamlessly and securely with AWS data sources like Redshift and offers a pay-per-session pricing model. |
| **Monitoring & Alerting System** | Amazon CloudWatch (Logs, Metrics, Alarms, Container Insights) | The native AWS observability suite. Provides the tightest integration with ECS, Fargate, and other AWS services, making it the most direct path to comprehensive monitoring. |
| **(Notification Channel)** | Amazon SNS | A fully managed pub/sub messaging service that decouples our alarm producers (CloudWatch) from our notification consumers (email, Slack, etc.). |
| **Business User / Analyst** | N/A (Human Actor) | The end-consumer of the data platform's output, who interacts with the system primarily through the BI Platform. |
| **Pipeline Operator** | N/A (Human Actor) | The person or team responsible for the operational health of the data pipeline, who interacts with the Monitoring & Alerting System. |

#### **3. Physical View: AWS Deployment Diagram**

This diagram overlays the monitoring data flow (metrics and logs) and the alerting path on top of our existing hybrid architecture.

```mermaid
graph TD
    subgraph "Operations Team"
        Operator["Pipeline Operator"]
    end

    subgraph "AWS Cloud"
        %% Existing Hybrid Architecture
        subgraph "Containerized Control Plane (ECS)"
            Airflow["Airflow Containers"]
            SparkTask["Spark Job Container"]
        end
        
        subgraph "Managed Data Services"
            Ingestion["Data & Ingestion Services"]
        end

        %% New Monitoring & Alerting Layer
        subgraph "Observability Layer"
            CloudWatch["Amazon CloudWatch<br>(Logs, Metrics, Alarms)"]
            SNS["Amazon SNS Topic"]
        end
    end

    %% Monitoring Data Flow (dotted lines)
    Airflow -- "Streams Logs & Metrics" --> CloudWatch
    SparkTask -- "Streams Logs & Metrics" --> CloudWatch
    Ingestion -- "Streams Logs & Metrics" --> CloudWatch

    %% Alerting Flow (dashed lines)
    CloudWatch -. "Triggers Alarm" .-> SNS
    SNS -. "Sends Notification" .-> Operator
```
