#### **1. Logical View: C4 Component Diagram**

The logical diagram is expanded to show the new external systems and the specialized ingestion components designed to handle them.

```mermaid
C4Component
  title Component Diagram for Third-Party Data Integration

  System_Boundary(third_party, "Third-Party Systems") {
    System(ga, "Google Analytics", "SaaS")
    System(support, "Support Vendor", "Provides customer support logs via API")
  }

  System_Boundary(app_systems, "Application Systems") {
    Component(internal_dbs, "Internal Databases", "PostgreSQL, MySQL", "Orders, Logistics, Promotions, Payments")
  }

  System_Boundary(data_platform, "Pulse Analytics Platform") {
    Component(replication_service, "Data Replication Service", "AWS DMS", "Replicates internal database data")
    Component(saas_ingestion, "SaaS Ingestion Service", "AWS AppFlow", "Pulls data from SaaS platforms like Google Analytics")
    Component(custom_ingestion, "Custom Ingestion Service", "AWS Lambda", "Fetches data from custom vendor APIs")
    Component(raw_data_store, "Raw Data Store", "Data Lake", "Stores raw, immutable data from all sources")
  }

  Rel(internal_dbs, replication_service, "Replicates data from")
  Rel(replication_service, raw_data_store, "Writes to")

  Rel(ga, saas_ingestion, "Pulls data from", "API")
  Rel(saas_ingestion, raw_data_store, "Writes to")

  Rel(support, custom_ingestion, "Fetches logs from", "API")
  Rel(custom_ingestion, raw_data_store, "Writes to")
```

#### **2. Physical View: Mapping to AWS Resources**

We add AWS AppFlow and AWS Lambda to our set of services, mapping them to the new logical components.

| C4 Component                 | AWS Resource                                     | Rationale for Selection                                                                                                          |
| ---------------------------- | ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| **Google Analytics**           | (External SaaS)                                  | A key third-party data source.                                                                                                   |
| **Support Vendor**             | (External SaaS)                                  | Another critical third-party source.                                                                                             |
| **Internal Databases**         | Amazon RDS                                       | Existing managed databases.                                                                                                      |
| **Data Replication Service**   | AWS Database Migration Service (DMS)             | The existing managed service for internal database replication.                                                                  |
| **SaaS Ingestion Service**     | AWS AppFlow                                      | A fully managed integration service with pre-built connectors for SaaS applications, eliminating the need for custom code for GA. |
| **Custom Ingestion Service**   | AWS Lambda                                       | A serverless compute service perfect for running custom, event-driven code to interact with any vendor API without managing servers. |
| **Raw Data Store**             | Amazon S3 Bucket                                 | The central, scalable, and cost-effective data lake.                                                                             |

#### **3. Physical View: AWS Deployment Diagram**

This diagram adds the new ingestion paths for third-party data, showing how they run in parallel with the existing internal data ingestion pipeline.

```mermaid
graph TD
    subgraph "Third-Party Vendors"
        GA["Google Analytics API"]
        Support["Support Vendor API"]
    end

    subgraph "AWS Cloud"
        subgraph "Application VPC"
            RDS_Services["Amazon RDS<br>(Internal DBs)"]
        end

        subgraph "Analytics VPC"
            DMS["AWS DMS<br>Replication Instance"]
            AppFlow["AWS AppFlow"]
            Lambda["AWS Lambda<br>Function"]
            S3["S3 Bucket<br>(Raw Data Store)"]
        end

        %% Internal Data Flow
        RDS_Services --> DMS
        DMS -- "Internal Data" --> S3

        %% Third-Party Data Flows
        GA -- "API Call" --> AppFlow
        AppFlow -- "GA Data" --> S3

        Support -- "API Call" --> Lambda
        Lambda -- "Support Logs" --> S3
    end
```
