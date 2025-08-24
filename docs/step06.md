#### **1. Logical View: C4 Component Diagram**

The logical diagram is completed by adding the final `Business Intelligence` component, which consumes data from the warehouse and delivers it to end-users.

```mermaid
C4Component
  title Component Diagram with Business Intelligence Layer

  Person(user, "Business User / Analyst", "Views dashboards and explores data")

  System_Boundary(data_platform, "Pulse Analytics Platform") {
    Component(data_warehouse, "Data Warehouse", "Amazon Redshift", "Stores structured, analysis-ready data")
    Component(bi_platform, "Business Intelligence Platform", "Amazon QuickSight", "Provides dashboards, reports, and self-service analytics")
  }

  Rel(data_warehouse, bi_platform, "Is the data source for")
  Rel(bi_platform, user, "Delivers insights to")
```

#### **2. Physical View: Mapping to AWS Resources**

We introduce Amazon QuickSight, the final piece of our AWS-native analytics stack.

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

#### **3. Physical View: AWS Deployment Diagram**

This diagram adds Amazon QuickSight to the architecture, showing it as the user-facing layer that connects to our data warehouse.

```mermaid
graph TD
    subgraph "Users"
        User["Business Users & Analysts"]
    end

    subgraph "AWS Cloud"
        subgraph "Analytics VPC"
            %% Data flow from previous steps is assumed
            Redshift["Amazon Redshift<br>Cluster"]
        end

        subgraph "BI Service"
             %% QuickSight is a managed service outside the VPC but connects to it
            QuickSight["Amazon QuickSight"]
        end
    end

    Redshift -- "Provides data via<br>secure connection" --> QuickSight
    QuickSight -- "Serves dashboards & reports via HTTPS" --> User
```