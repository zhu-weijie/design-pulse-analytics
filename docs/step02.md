#### **1. Logical View: C4 Component Diagram**

The logical diagram is updated to include the new data sources. The `Ingestion Service` is now evolved into a more capable `Data Replication Service` that handles all internal databases.

```mermaid
C4Component
  title Component Diagram for Internal Data Source Integration

  System_Boundary(app_systems, "Application Systems") {
    Component(orders_db, "Orders DB", "PostgreSQL")
    Component(logistics_db, "Logistics DB", "MySQL")
    Component(promotions_db, "Promotions DB", "PostgreSQL")
    Component(payments_db, "Payments DB", "MySQL")
  }

  System_Boundary(data_platform, "Pulse Analytics Platform") {
    Component(replication_service, "Data Replication Service", "AWS DMS", "Performs full load and ongoing replication (CDC) from all internal databases")
    Component(raw_data_store, "Raw Data Store", "Data Lake", "Stores raw, immutable data from all sources as files")
  }

  Rel(orders_db, replication_service, "Replicates data from")
  Rel(logistics_db, replication_service, "Replicates data from")
  Rel(promotions_db, replication_service, "Replicates data from")
  Rel(payments_db, replication_service, "Replicates data from")
  Rel(replication_service, raw_data_store, "Writes raw data to")
```

#### **2. Physical View: Mapping to AWS Resources**

The EC2 script is now replaced with AWS DMS, a managed service designed for this exact purpose.

| C4 Component                 | AWS Resource                                     | Rationale for Selection                                                                                                   |
| ---------------------------- | ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| **Orders DB**                  | Amazon RDS for PostgreSQL                        | Managed relational database.                                                                                              |
| **Logistics DB**               | Amazon RDS for MySQL                             | Managed relational database.                                                                                              |
| **Promotions DB**              | Amazon RDS for PostgreSQL                        | Managed relational database.                                                                                              |
| **Payments DB**                | Amazon RDS for MySQL                             | Managed relational database.                                                                                              |
| **Data Replication Service**   | AWS Database Migration Service (DMS)             | A managed service that simplifies database migration and replication, reducing operational overhead and improving reliability. |
| **Raw Data Store**             | Amazon S3 Bucket                                 | Highly durable, scalable, and cost-effective object storage for the data lake.                                            |

#### **3. Physical View: AWS Deployment Diagram**

This diagram shows the replacement of the single EC2 instance with a scalable AWS DMS service that connects to all internal application databases.

```mermaid
graph TD
    subgraph "AWS Cloud"
        subgraph "Application VPC"
            RDS_Orders["Amazon RDS<br>(Orders DB)"]
            RDS_Logistics["Amazon RDS<br>(Logistics DB)"]
            RDS_Promotions["Amazon RDS<br>(Promotions DB)"]
            RDS_Payments["Amazon RDS<br>(Payments DB)"]
        end

        subgraph "Analytics VPC"
            DMS["AWS DMS<br>Replication Instance"]
            S3["S3 Bucket<br>(Raw Data Store)"]
        end

        RDS_Orders --> DMS
        RDS_Logistics --> DMS
        RDS_Promotions --> DMS
        RDS_Payments --> DMS

        DMS -- "Writes all source data<br>(Parquet/CSV)" --> S3
    end
```
