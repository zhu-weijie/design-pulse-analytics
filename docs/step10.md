#### **1. Logical View: C4 Component Diagram**

The logical diagram is updated to show the `Data Quality Gate` as a new, explicit step in the process, making it a first-class citizen of the architecture.

```mermaid
C4Component
  title Component Diagram with Sequential Data Quality Gate

  System_Boundary(data_platform, "Pulse Analytics Platform") {
    Component(raw_data_store, "Raw Data Store", "S3 Data Lake", "Stores raw, immutable data")
    Component(quality_gate, "Data Quality Gate", "Great Expectations", "Reads and validates raw data against a suite of expectations")
    Component(transformation, "Containerized Transformation", "Docker with PySpark", "Applies business logic and transformations to validated data")
    Component(data_warehouse, "Data Warehouse", "Amazon Redshift", "Stores final, high-quality, analysis-ready data")
  }

  Rel(raw_data_store, quality_gate, "1. Reads raw data from")
  Rel(quality_gate, transformation, "2. Passes valid data to")
  Rel(transformation, data_warehouse, "3. Writes transformed data to")
```

#### **2. Physical View: Mapping to AWS Resources**

The new logical component is implemented as a library within our existing container.

| C4 Component                 | AWS Resource / Tool                              | Rationale for Selection                                                                                                                              |
| ---------------------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| ... (Previous Components) ... | ... (Previous Resources) ...                     | ... (No change) ...                                                                                                                                  |
| **Data Quality Gate**        | Great Expectations (Python Library)              | The leading open-source tool for data validation. It allows for expressive, declarative data tests that can be version-controlled alongside our Spark code. It integrates natively with Spark DataFrames. |

#### **3. Physical View: AWS Deployment Diagram**

The physical diagram is updated to show the new validation step within the Spark job's execution flow.

```mermaid
graph TD
    subgraph "AWS Cloud"
        %% Control Plane
        subgraph "Containerized Control Plane (ECS)"
            Worker["Container: Airflow Worker"]
            SparkTask["Container: Spark Job"]
        end

        %% Data Plane
        subgraph "Data & Ingestion Services"
            S3["S3 Bucket (Raw Data Lake)"]
            Redshift["Amazon Redshift (Warehouse)"]
        end
        
        %% Observability
        subgraph "Observability Layer"
            CloudWatch["Amazon CloudWatch"]
        end
    end
    
    %% Control Flow
    Worker -. "Launches" .-> SparkTask

    %% Detailed Spark Job Flow
    subgraph SparkTask
        direction TB
        ReadData["Read Raw Data"]
        ValidateData["Run Data Quality Checks<br>(Great Expectations)"]
        TransformData["Run Transformations"]
        
        ReadData --> ValidateData
        ValidateData -- "On Success" --> TransformData
    end
    
    %% Failure Path
    ValidateData -- "On Failure, sends alert via" --x CloudWatch

    %% Data Flow
    S3 -- "Reads from" --> ReadData
    TransformData -- "Writes TRANSFORMED to" --> Redshift
```
