# Implementation plan

Only Phase 0 belongs in this setup. Verification below describes future acceptance criteria unless stated otherwise. Add tests with each feature; Phase 6 expands integration coverage rather than postponing all testing until then.

| Phase | Deliverable | Verification |
| --- | --- | --- |
| 0: Structure and documentation | Package placeholders, configs, contracts, architecture, CI skeleton | Parse TOML, compile/import modules, inspect files, verify public repository and commits; no pipeline tests claimed |
| 1: Synthetic input generation and ingestion | Seeded generator, fixed clock, event/control snapshots, input manifests | Same seed produces same checksums; ingestion preserves counts and provenance; malformed fixtures stay identifiable |
| 2: Schema, freshness and deduplication | Explicit contracts, rejection reasons, deterministic duplicate handling | Test nulls, types, unknown columns, timestamp boundaries and conflicting duplicates across repeat batches |
| 3: Consent, jurisdiction and filtering | Purpose-specific decisions, interval resolution, output allowlists | Denied and unverified cases never publish; test revocation, overlapping intervals, stale snapshots and restricted field leakage |
| 4: Retention, deletion and replay | Tombstone ledger, lineage, propagation state and controlled backfills | Delete across raw/staging/approved/reject stores; replay cannot resurrect records; partial propagation blocks affected outputs |
| 5: PySpark, SQL and approved outputs | Distributed transforms, Parquet contracts, analytics SQL, gated manifest publication | Compare expected rows across partition counts; inspect actual output schemas; interrupted writes cannot expose partial datasets |
| 6: Tests, CI, audit and failure cases | Integrated pytest suite, Spark CI setup, durable audit and recovery scenarios | Inject audit/policy/storage failures, crash around manifest commit, verify idempotency and safe logs; CI runs actual tests |
| 7: Performance and operations | Measured partition tuning, run metrics, deletion lag, recovery notes | Report hardware, dataset size, reproducible commands and measured timings; exercise skew and recovery without inventing benchmarks |

## Next task

Agree on the draft fields and synthetic policy values, then implement a seeded generator and local ingestion manifest. Include a valid event, malformed event, stale event, duplicate, revoked consent, unknown jurisdiction and deletion request. Keep generation separate from validation so a deliberately bad input is not silently corrected by the generator.

## Deferred decisions

Choose compatible Spark, Java and Python versions when Spark is introduced; do not install Spark just for empty modules. Add a YAML parser when configurations are consumed. Choose retention and freshness values explicitly for the synthetic scenario. Determine whether versioned replacement events are necessary before implementing corrections. S3, orchestration and distributed publication locking remain optional later extensions.
