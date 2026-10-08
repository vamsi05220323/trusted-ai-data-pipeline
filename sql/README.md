# SQL models

Phase 5 will add models over approved Parquet datasets using Spark SQL.
SQL must read committed approved snapshots, never raw or staging paths.
Proposed first models: daily product interaction counts and purpose-specific training projections.
Aggregates remain subject to deletion and retention through source lineage and recomputation.
