# Planned verification

There are no behavioral tests yet. Phase 0 CI checks syntax, TOML parsing, and imports only.
Add pytest cases alongside each implemented phase; Phase 6 expands the failure matrix.

Required cases: malformed schema, unexpected fields, stale and future timestamps,
conflicting duplicates, missing or revoked consent, purpose mismatch, unknown jurisdiction,
restricted output fields, expired records, pending deletion, stale control snapshots,
failed audit writes, interrupted publication, repeat runs, and replay after deletion.

Use a fixed clock and seeded synthetic inputs. Assert exact output schemas and counts,
reason codes, absence of restricted values in logs, and unchanged published manifests on failure.
Compare local Spark results across partition counts once distributed processing is introduced.
Never turn pytest's no-tests-collected exit into a passing pipeline test result.
