#### **1. Logical View: C4 Component Diagram**

The logical diagram introduces an `Orchestrator` that acts as the "brain," coordinating the pipeline components.

```mermaid
C4Component
  title Component Diagram with Pipeline Orchestration

  System_Boundary(aws_services, "AWS Services") {
    Component(scheduler, "Scheduler", "Amazon EventBridge", "Triggers the pipeline on a time-based schedule")
  }

  System_Boundary(data_platform, "Pulse Analytics Platform") {
    Component(orchestrator, "Pipeline Orchestrator", "AWS Step Functions", "Manages the sequence, dependencies, and error handling of pipeline tasks")
    Component(ingestion_services, "Ingestion Services", "DMS, AppFlow, Lambda", "Ingests all raw data")
    Component(data_transformation, "Data Transformation", "AWS Glue ETL", "Transforms raw data")
  }

  Rel(scheduler, orchestrator, "Triggers")
  Rel(orchestrator, ingestion_services, "Starts and monitors")
  Rel(orchestrator, data_transformation, "Starts and monitors")
```

#### **2. Physical View: Mapping to AWS Resources**

We introduce Amazon EventBridge for scheduling and AWS Step Functions for orchestration.

| C4 Component                 | AWS Resource                                     | Rationale for Selection                                                                                                                              |
| ---------------------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Analytics**           | (External SaaS)                                  | A key third-party data source.                                                                                                   |
| **Support Vendor**             | (External SaaS)                                  | Another critical third-party source.                                                                                             |
| **Internal Databases**         | Amazon RDS                                       | Existing managed databases.                                                                                                      |
| **Data Replication Service**   | AWS Database Migration Service (DMS)             | The existing managed service for internal database replication.                                                                  |
| **SaaS Ingestion Service**     | AWS AppFlow                                      | A fully managed integration service with pre-built connectors for SaaS applications, eliminating the need for custom code for GA. |
| **Custom Ingestion Service**   | AWS Lambda                                       | A serverless compute service perfect for running custom, event-driven code to interact with any vendor API without managing servers. |
| **Raw Data Store**             | Amazon S3 Bucket                                 | The central, scalable, and cost-effective data lake.                                                                             |
| **Data Warehouse**             | Amazon Redshift                                  | A fully managed, petabyte-scale cloud data warehouse. Its columnar storage and massive parallel processing (MPP) architecture are ideal for BI and analytics. It integrates seamlessly with S3. |
| **Data Transformation**        | AWS Glue (ETL Jobs, Crawler, Data Catalog)       | A fully managed, serverless ETL service that natively integrates with S3 and Redshift. It uses Apache Spark, enabling large-scale data processing without managing clusters. |
| **Business Intelligence Platform** | Amazon QuickSight                              | A serverless, cloud-native BI service that integrates seamlessly and securely with AWS data sources like Redshift. Its managed nature removes operational overhead. |
| **Business User**              | (Human Actor)                                    | The end consumer of the data platform's output.                                                                                                      |
| **Scheduler**                | Amazon EventBridge (Scheduler)                   | A serverless event bus that makes it easy to trigger workflows from a wide variety of sources, including a simple time-based cron schedule.           |
| **Pipeline Orchestrator**    | AWS Step Functions                               | A serverless function orchestrator that excels at coordinating multiple AWS services (like Lambda and Glue) into a resilient, manageable workflow.    |

#### **3. Physical View: AWS Deployment Diagram**

This diagram shows the control flow (dotted lines) from the new orchestration layer to the existing data processing components.

```mermaid
graph TD
    subgraph "AWS Cloud"
        subgraph "Orchestration & Control Layer"
            Scheduler["Amazon EventBridge<br>(Scheduled Rule)"]
            StateMachine["AWS Step Functions<br>(State Machine)"]
        end

        subgraph "Execution Layer (Analytics VPC)"
            Ingestion["Ingestion Services<br>(Lambda, AppFlow, etc.)"]
            Glue_Job["AWS Glue ETL Job"]
            Redshift["Amazon Redshift Cluster"]
        end

        %% Control Flow
        Scheduler -- "Triggers daily" --> StateMachine
        StateMachine -. "Starts Ingestion Tasks" .-> Ingestion
        StateMachine -. "Starts Transformation after Ingestion" .-> Glue_Job
        
        %% Data Flow (for context)
        Ingestion -- "Raw Data" --> S3_Bucket((S3 Bucket))
        Glue_Job -- "Transformed Data" --> Redshift
    end
```
