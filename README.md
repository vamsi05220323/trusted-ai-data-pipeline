# trusted-ai-data-pipeline

Started this project to work through a common data problem. Data arrives from different systems, but its presence in storage does not tell us whether it is correct, recent enough, or allowed for a particular use. Analytics and model training may also have different permissions for the same record.

The plan is to check these conditions before publishing a dataset. I want to understand how the checks fit into a pipeline, how failures are handled, and how deletion requests affect data that has already been processed. All inputs will be synthetic.

## Data and processing

Inputs will include customer and product interaction events, consent and eligibility records, jurisdiction rules, deletion requests, and historical records for replay.

The pipeline will ingest raw records, validate schemas and freshness, handle duplicates, resolve permissions for each intended use, remove restricted fields, and apply retention and deletion rules. A final check will run against the actual output before anything is published.

Expected outputs are curated analytics data, approved data for simulated model training, restricted rejection records, and execution metadata. These outputs will have separate contracts. A record approved for analytics is not automatically approved for training.

If a required check fails or cannot be verified, the affected data stays unpublished. Missing policy data, stale consent, and an unavailable deletion ledger must not quietly become approvals. Rejected records will have reason codes, without copying private payloads into logs.

## Components and tools

Python modules will separate ingestion, validation, privacy, deletion, publishing, and audit responsibilities. YAML will hold explicit policy and pipeline settings. The planned processing stack is PySpark, SQL, and Parquet, with pytest and GitHub Actions for verification. Development starts locally; S3 and orchestration can come later.

## Current status

Phase 0 only: repository structure, proposed contracts, design notes, configuration placeholders, and a CI skeleton. There is no working pipeline, generated dataset, or pipeline test suite yet. CI checks syntax and imports only.

Next is deterministic synthetic input generation and ingestion, including bad records and repeatable source metadata. See [the implementation plan](docs/implementation_plan.md) and [architecture notes](docs/architecture.md).
