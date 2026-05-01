# GEMINI.md - Project Context

## Project Overview
This project is a Site Reliability Engineering (SRE) implementation for a data engineering pipeline. The goal is to process approximately 100,000 orders from the Olist marketplace, transitioning data from CSV files to a Postgres analytical database. The project emphasizes building a reliable, idempotent, and observable ETL process that can be monitored via Grafana dashboards.

### Key Goals:
- **Idempotence:** Ensuring that re-running the ETL doesn't duplicate data.
- **Observability:** Monitoring the pipeline's health and SLA via Grafana.
- **Resilience:** Handling partial failures (e.g., corrupted files, database downtime, EC2 interruptions).
- **Data Integrity:** Preventing silent failures and data loss.

## Main Technologies
- **Language:** Likely Python or SQL (to be confirmed as code is added).
- **Database:** Postgres (Analytical).
- **Visualization:** Grafana.
- **Infrastructure:** AWS (EC2).
- **Data Source:** CSV files.

## Project Structure
- `specs/`: Contains project requirements and problem definitions.
  - `00problem.md`: Detailed description of the business problem, stakeholders, and failure modes.

## Building and Running
*Current status: Initialization phase. Code and configuration to be implemented.*

### TODO:
- [ ] Implement CSV to Postgres ingestion script.
- [ ] Set up Postgres database schema.
- [ ] Configure Grafana dashboards for monitoring.
- [ ] Define observability metrics and SLA monitoring.

## Development Conventions
- **Reliability First:** Every component should account for the "Modos de Falha" (Failure Modes) defined in `specs/00problem.md`.
- **Idempotent Loads:** Use `UPSERT` or staging table patterns to ensure data consistency.
- **Observability:** Implement logging and metrics for every stage of the pipeline.
- **Documentation:** Keep the `specs/` directory updated with architectural decisions.
