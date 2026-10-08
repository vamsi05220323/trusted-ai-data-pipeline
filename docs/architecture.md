# Proposed architecture

Status: design only. None of these controls is implemented.

## Flow and boundaries

Synthetic sources -> restricted raw landing -> schema/freshness/deduplication ->
purpose-specific consent and jurisdiction checks -> field projection -> retention and
deletion filtering -> restricted staging -> output validation -> publication manifest.
Every stage also produces safe execution metadata. Consumers resolve committed manifests;
they must not enumerate staging files or treat a successful Spark write as publication.

## Components

| Module | Responsibility |
| --- | --- |
| ingestion | Record source snapshots, checksums, schema versions and ingestion time |
| validation | Validate shape, timestamps, uniqueness and output contracts |
| privacy | Resolve use-specific eligibility and project allowed fields |
| deletion | Apply retention, tombstones, propagation and replay exclusion |
| publishing | Validate evidence and expose complete approved snapshots |
| audit | Persist run state, reason counts, policy versions and lineage |

Initial sources are event batches, consent histories, eligibility records, fictional
jurisdiction policies, deletion requests, and historical event snapshots. Targets are
analytics Parquet, simulated-training Parquet, restricted rejection storage, and audit
metadata. Input histories and deletion ledgers are restricted even with synthetic data.
There is no chatbot, retrieval index, model training job, or agent.

## Control decisions

Consent is evaluated separately for analytics and simulated training. Require permission
at event time and current permission at publication. Explicit denial rejects a record;
missing, stale or contradictory control evidence blocks its dataset-run-purpose publication.
A run pins input manifests, policy version, code revision, clock, and control snapshots.
Immediately before commit, recheck deletion and consent watermarks. A changed watermark
requires reevaluation rather than publishing with an obsolete decision.

Known invalid records are excluded with reason codes. Other records may publish only if
the complete remaining dataset passes all required controls. An unreadable policy or
unverifiable source completeness blocks the whole affected dataset-run-purpose. Never
fall back to an old approved snapshot silently after a failed new run.

Use explicit output field allowlists; reject unexpected input fields and free text in the
initial contract. Do not describe hashed identifiers as anonymized. Analytics subject keys
are pseudonymous. Training removes direct subject keys but remains controlled because
linkable event IDs and internal lineage exist. General anonymization is deferred.

## Publication and recovery

Write to a unique run staging directory. Validate the actual Parquet schema and values,
counts, eligibility evidence, retention cutoff and deletion watermark. Persist a durable
PREPARED audit record before atomically replacing a local manifest pointer. Record COMMITTED
afterward. Reconcile a crash between the pointer change and final audit from the manifest;
never expose a pointer without durable preparation evidence. Stage output is not readable
through the consumer interface. Local first implementation uses a single writer lock.
Multiwriter fencing and object-store transactions are later work, not implied by rename.

A stable run identity plus input and policy digests supports retries. Same record identity
and same payload deduplicate; conflicting payloads are quarantined rather than resolved
by arbitrary partition order. Restricted rejects store minimal metadata, with raw payloads
kept only in bounded-retention raw storage. Logs contain no payloads or direct identifiers.

## Retention, deletion and replay

Retention is measured from event time with a fixed UTC cutoff; control and audit retention
are separate. Deletion creates a durable tombstone before rewriting affected raw, rejected,
staged and approved datasets. A restricted subject-to-output lineage index locates records
even when the public projection omits subject keys. Invalidate affected manifests while
rewriting and recompute derived aggregates. Track each destination's acknowledgement;
partial propagation remains pending and prevents affected republishing.

Replay and backfill reapply current consent, retention and tombstones. Never restore a
deleted record from a historical input. Tombstones must outlive replayable history; their
own minimization and expiry need an explicit policy. Physical Parquet rewrites, old files,
cached copies and backups require separate verification. A deletion request is not complete
merely because the next query filters it out. Already trained model deletion is out of scope.

## Distributed execution and operations

Phase 1 ingestion establishes local contracts; Phase 5 introduces Spark transformations.
Use built-in Spark expressions and joins, avoid collecting datasets to the driver, and
broadcast only demonstrably small policy tables. Start with event-date partitions and
measure skew and file sizes before tuning. No scale or throughput claims exist yet.
Track source/output/reject counts, control age, deletion lag and publication state. AWS S3,
orchestration, IAM and multiwriter operation are future extensions. Local filesystem
separation demonstrates boundaries but is not a production access-control system.
