#### **1. Logical View: C4 Component Diagram**

The logical diagram introduces the `Data Catalog` as a central system that users interact with to *learn about* the data in our storage layers.

```mermaid
C4Component
  title Component Diagram with Data Catalog

  Person(analyst, "Data Analyst / Business User", "Searches for relevant datasets, understands their meaning, and trusts their quality")

  System_Boundary(data_platform, "Pulse Analytics Platform") {
    Component(data_catalog, "Data Catalog", "Amundsen", "A user-facing platform for data discovery, governance, and understanding")
    Component(raw_data_store, "Raw Data Store", "S3 Data Lake")
    Component(data_warehouse, "Data Warehouse", "Amazon Redshift")
  }
  
  Rel(analyst, data_catalog, "Uses to discover and understand data")
  
  %% Corrected Relationships: Catalog pulls from BOTH data stores
  Rel(data_catalog, raw_data_store, "Pulls metadata from")
  Rel(data_catalog, data_warehouse, "Pulls metadata from")
```

#### **2. Physical View: Mapping to AWS Resources**

The new logical component is implemented as a suite of containerized services running on our existing ECS cluster.

| C4 Component                 | AWS Resource / Tool                              | Rationale for Selection                                                                                                                              |
| ---------------------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| ... (Previous Components) ... | ... (Previous Resources) ...                     | ... (No change) ...                                                                                                                                  |
| **Data Catalog**             | Amundsen on Amazon ECS (Fargate)                 | A leading open-source data catalog with a strong focus on user experience and data discovery. Deploying it on our existing ECS cluster leverages our container-based strategy. |

#### **3. Physical View: AWS Deployment Diagram**

The physical diagram shows the new Amundsen containers on the ECS cluster and, crucially, the flow of *metadata* (not business data) from our data stores into the catalog.

```mermaid
graph TD
    subgraph "Users"
        Analyst["Data Analyst"]
    end

    subgraph "AWS Cloud"
        %% Existing Control Plane
        subgraph "Containerized Control Plane (ECS)"
            Airflow["Airflow Containers"]
            
            %% New Data Catalog Services
            subgraph "Amundsen Services"
                Amundsen_Web["Container: Amundsen Web UI"]
                Amundsen_Meta["Container: Amundsen Metadata Service"]
                Amundsen_Search["Container: Amundsen Search Service"]
                Neo4j["Container: Neo4j (Graph DB)"]
                Elasticsearch["Container: Elasticsearch"]
            end
        end

        %% Existing Data Plane
        subgraph "Data Plane"
            GlueCatalog["AWS Glue Data Catalog"]
            Redshift["Amazon Redshift"]
        end
    end
    
    %% User Interaction
    Analyst -- "Accesses UI" --> Amundsen_Web
    
    %% Metadata Ingestion Flow
    Airflow -. "Runs metadata ingestion task against" .-> GlueCatalog
    Airflow -. " " .-> Redshift
    
    GlueCatalog -- "Pushes Schemas to" --> Amundsen_Meta
    Redshift -- "Pushes Schemas, Stats to" --> Amundsen_Meta

    %% Amundsen Internal Arch
    Amundsen_Meta -- "Stores metadata in" --> Neo4j
    Amundsen_Meta -- "Indexes metadata in" --> Elasticsearch
    Amundsen_Web -- "Queries for display" --> Amundsen_Meta
    Amundsen_Web -- "Queries for search" --> Amundsen_Search
    Amundsen_Search -- "Searches" --> Elasticsearch
```
