# GoldenRecord MDM

GoldenRecord MDM is a comprehensive Master Data Management (MDM) pipeline designed to ingest, process, match, and resolve customer identities from multiple sources into a single "Golden Record". The project features a robust 10-phase data pipeline orchestrated via Bash, with data generation, ingestion, matching, historical tracking, and governance.

## Technologies Used
* **Python**: Used for data generation (producers), ingestion, staging, matching, and governance scripts.
* **PostgreSQL**: Serving as the data warehouse and relational storage engine (containerized).
* **MinIO**: S3-compatible object storage for intermediate/raw file ingestion (containerized).
* **Docker & Docker Compose**: Infrastructure orchestration.
* **Bash**: Pipeline orchestration script.

## Pipeline Architecture

The end-to-end pipeline is orchestrated by `run_mdm_pipeline.sh`. It consists of the following phases:

* **Phase 0 — Infrastructure & Base Schema**: Starts up Postgres and MinIO using Docker Compose and applies the initial SQL schemas.
* **Phase 1 — Running Producers**: Generates mock data for Sales, Support, and Marketing systems.
* **Phase 2 — Ingestion (MinIO → Raw)**: Loads the generated data into the raw schemas in Postgres.
* **Phase 3 — Staging Normalization**: Cleans and normalizes data into a standard staging format.
* **Phase 4 — Identity Inputs**: Prepares the data for matching.
* **Phase 4.5 — Blocking Candidates**: Generates blocked match candidates (reducing the search space for the matching engine).
* **Phase 5 — Identity Matching Engine**: Executes matching algorithms to find duplicates and identical customer entries.
* **Phase 6 — Global Customer ID Resolution**: Assigns a global unique identifier to resolved customer entities.
* **Phase 7 — Golden Record Construction**: Builds the definitive "Golden Record" (`dim_customers`) by surviving fields based on rules.
* **Phase 8 — Golden Record History (SCD2)**: Tracks slowly changing dimensions (Type 2) to maintain historical versions of the golden records.
* **Phase 9A — Golden Change Events (CDC)**: Publishes change data capture events based on modifications.
* **Phase 9B — Steward Overrides**: Applies manual data steward overrides over the automated match results.
* **Phase 10 — Data Quality & Governance**: Detects conflicts, calculates match confidence metrics, and manages the steward review queue.

## Getting Started

1. **Prerequisites**: Ensure you have Docker, Docker Compose, and Python installed on your system.
2. **Environment Variable**: Ensure the `.env` file is properly configured with your desired credentials (`DB_USER`, `MINIO_ROOT_USER`, etc.).
3. **Run Pipeline**:
   The entire pipeline can be run from start to finish using the provided bash script:
   ```bash
   ./run_mdm_pipeline.sh
   ```

## Directory Structure
* `db/`: SQL scripts for schema initialization and table creation.
* `docker/`: Docker configurations.
* `producers/`: Python scripts to generate synthetic data for different source domains.
* `ingestion/`: MinIO to Postgres data ingestion scripts.
* `staging/`: Normalization scripts.
* `matching/`: Data matching scripts and engines.
* `identity/`: Identification and resolution logic.
* `gold/`: Golden record creation, CDC, overrides, and governance scripts.
