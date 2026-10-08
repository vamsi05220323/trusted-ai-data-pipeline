# Proposed data contracts

Draft version 1. All examples and policies will be synthetic. These are requirements for
future code, not schemas currently enforced by the repository.

## Shared rules

Input JSONL initially; approved outputs Parquet later. UTF-8 strings, UTC timestamps with
an explicit offset, nonempty identifiers, and exact types. Do not silently cast malformed
values. Required fields cannot be null. Reject extra fields unless a new contract version
explicitly permits them. Unknown schema versions block ingestion publication.

| Source | Proposed required fields | Constraints |
| --- | --- | --- |
| Events | schema_version, event_id, subject_key, event_time, event_type, product_id, quantity, jurisdiction, source_system | quantity integer >= 0; event_type in view/click/purchase; subject_key synthetic pseudonym |
| Consent history | schema_version, consent_id, subject_key, purpose, decision, effective_from, recorded_at, source_version | purpose analytics/simulated_training; decision granted/revoked; positive source_version |
| Eligibility | schema_version, eligibility_id, subject_key, purpose, eligible, effective_from, recorded_at, source_version | eligible boolean; no implicit eligibility from event presence |
| Deletion requests | schema_version, request_id, subject_key, requested_at, scope | initial scope all_subject_data; request_id unique |
| Historical inputs | original event fields plus source_snapshot_id | original event_time retained; replay must not reset retention age |

Consent and eligibility may have nullable effective_to for an open interval. Intervals are
[from, to), with to greater than from. Overlap or contradictory records are ambiguous and
block affected publication; do not use arrival order as a tie breaker. Current and event-time
permission must both be verified. Source snapshots must declare completeness and as_of time.
Absence of a deletion request is trusted only within a fresh, complete deletion snapshot.

Optional synthetic event fields email, phone, device_id, and free_text may be null or strings.
They exist only to exercise filtering and never enter approved outputs or diagnostic logs.
Fictional jurisdiction configuration maps jurisdiction to allowed purposes and retention_days.
An empty policy, missing duration, or unknown jurisdiction is unresolved, never permissive.

## Ingestion metadata

Attach ingestion_id, ingested_at, batch_id, source_snapshot_id, source_checksum,
source_record_reference and schema_version. Keep provenance separate from input payloads.
Snapshot manifests carry record_count, as_of, completeness, and source_version.
Freshness compares event_time to the fixed run evaluation_time; future-time tolerance and
maximum age must be explicitly configured. Late replay needs a separate approved backfill
window; it still cannot bypass retention or current permission checks.

Deduplication key is (source_system, event_id). Identical repeated payloads collapse to one
record. Conflicting payloads for that key reject every conflicting variant. Corrections and
cross-batch conflicts require an explicit versioned replacement contract before support.

## Output contracts

| Dataset | Fields / contents |
| --- | --- |
| Analytics | event_id, subject_key, event_time, event_type, product_id, quantity |
| Simulated training | event_id, event_time, event_type, product_id, quantity |
| Restricted rejections | rejection_id, run_id, protected source reference, stage, reason_code, detected_at |
| Audit | run_id, stage, state, reason counts, input/output counts, evaluation_time, recorded_at |
| Restricted lineage | subject_key, source identity, output dataset, output snapshot, event_id |

Output rows additionally carry run_id, output_schema_version and policy_version. Manifests
carry purpose, input digests, code revision, control snapshot versions and watermarks,
output file checksums, row counts and validation evidence. Metadata is not permission to
publish: the actual output schema and contents must pass verification too.

## Invalid records and failures

Proposed codes: SCHEMA_INVALID, STALE_EVENT, FUTURE_EVENT, DUPLICATE_CONFLICT,
CONSENT_DENIED, CONSENT_UNVERIFIED, PURPOSE_DENIED, POLICY_UNVERIFIED,
RETENTION_EXPIRED, DELETION_PENDING, CONTROL_SNAPSHOT_STALE, OUTPUT_INVALID,
AUDIT_UNAVAILABLE. Exact duplicate counts are recorded separately from rejects.
Malformed records get safe source references without including their payloads in logs.
Restricted diagnostics have their own retention policy. A failed batch may keep restricted
staging for investigation only within that retention window; consumers cannot read it.

Schema evolution is explicit and versioned. Adding an input field does not add it to an
output. Renames, type changes and changes to permission semantics require a contract update,
fixtures, compatibility checks and a replay decision before release.
