#### **1. Logical View: C4 Component Diagram**

The logical diagram is updated to include the new Data Warehouse component, which becomes the primary serving layer for future analytical tools.

```mermaid
C4Component
  title Component Diagram with Data Warehouse

  System_Boundary(third_party, "Third-Party Systems") {
    System(external_sources, "External Systems", "Google Analytics, Support Vendor")
  }

  System_Boundary(app_systems, "Application Systems") {
    Component(internal_dbs, "Internal Databases", "PostgreSQL, MySQL")
  }

  System_Boundary(data_platform, "Pulse Analytics Platform") {
    Component(ingestion_services, "Ingestion Services", "DMS, AppFlow, Lambda", "Handles all data ingestion from internal and external sources")
    Component(raw_data_store, "Raw Data Store", "Data Lake", "Stores all raw, immutable data")
    Component(data_warehouse, "Data Warehouse", "Amazon Redshift", "Stores structured, transformed data for high-performance analytics")
  }

  Rel(external_sources, ingestion_services, "Ingests data from")
  Rel(internal_dbs, ingestion_services, "Ingests data from")
  Rel(ingestion_services, raw_data_store, "Writes raw data to")
  Rel(raw_data_store, data_warehouse, "Is loaded into", "ETL/ELT Process")
```

#### **2. Physical View: Mapping to AWS Resources**

We introduce Amazon Redshift to our stack.

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

#### **3. Physical View: AWS Deployment Diagram**

This diagram adds the Amazon Redshift cluster to our Analytics VPC and shows the new critical data path from our S3 data lake into the warehouse.

```mermaid
graph TD
    subgraph "External Systems & Internal DBs"
        direction LR
        External_APIs["External APIs"]
        Internal_DBs["Internal RDS Databases"]
    end

    subgraph "AWS Cloud"
        subgraph "Analytics VPC"
            subgraph "Ingestion Layer"
                Ingestion_Services["AWS DMS, AppFlow, Lambda"]
            end

            subgraph "Storage & Analytics Layer"
                S3["S3 Bucket<br>(Raw Data Store)"]
                Redshift["Amazon Redshift<br>Cluster"]
            end
        end
    end

    External_APIs --> Ingestion_Services
    Internal_DBs --> Ingestion_Services
    Ingestion_Services -- "Raw Data" --> S3
    S3 -- "Load Process<br>(e.g., COPY command)" --> Redshift
```
