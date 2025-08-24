#### **1. Logical View: C4 Component Diagram**

The logical diagram now explicitly shows the `Data Transformation` component, which was only alluded to in the previous issue. This component is the "brain" of our pipeline.

```mermaid
C4Component
  title Component Diagram with Data Transformation

  System_Boundary(data_platform, "Pulse Analytics Platform") {
    Component(ingestion_services, "Ingestion Services", "DMS, AppFlow, Lambda", "Ingests all raw data")
    Component(raw_data_store, "Raw Data Store", "Data Lake", "Stores all raw, immutable data")
    Component(data_transformation, "Data Transformation", "AWS Glue ETL", "Cleans, joins, aggregates, and applies business logic to raw data")
    Component(data_warehouse, "Data Warehouse", "Amazon Redshift", "Stores structured, analysis-ready data")
  }

  Rel(ingestion_services, raw_data_store, "Writes to")
  Rel(raw_data_store, data_transformation, "Reads raw data from")
  Rel(data_transformation, data_warehouse, "Writes transformed data to")
```

#### **2. Physical View: Mapping to AWS Resources**

We introduce AWS Glue as the primary service for implementing the transformation logic.

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

#### **3. Physical View: AWS Deployment Diagram**

This diagram introduces the AWS Glue components, showing how they fit between S3 and Redshift to orchestrate the ETL process.

```mermaid
graph TD
    subgraph "AWS Cloud"
        subgraph "Analytics VPC"
            subgraph "Ingestion & Raw Storage"
                direction TB
                Ingestion_Services["AWS DMS, AppFlow, Lambda"] --> S3["S3 Bucket<br>(Raw Data Store)"]
            end

            subgraph "Transformation Layer"
                Glue_Catalog["AWS Glue Data Catalog<br>(Metadata Store)"]
                Glue_Crawler["AWS Glue Crawler"]
                Glue_Job["AWS Glue ETL Job<br>(PySpark Script)"]
            end

            subgraph "Analytics Layer"
                Redshift["Amazon Redshift<br>Cluster"]
            end

            %% Data Flow for ETL
            S3 -- "Scanned by" --> Glue_Crawler
            Glue_Crawler -- "Populates metadata in" --> Glue_Catalog
            Glue_Job -- "Reads raw data via Catalog" --> S3
            Glue_Job -- "Writes transformed data to" --> Redshift
        end
    end
```