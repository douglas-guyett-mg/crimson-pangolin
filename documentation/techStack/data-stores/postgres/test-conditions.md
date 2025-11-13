Title: Postgres (RDS) — Test Conditions
Status: Draft v1

Correctness & Concurrency
- Concurrent updates on same FC entity trigger version conflict; last writer wins only via explicit merge.
- Transactional integrity preserved across multi-entity updates; rollback on failure.

Performance
- P95 query latency < 30ms for typical OLTP reads; < 200ms for complex JSONB/FTS queries at baseline scale.
- Connection handling via RDS Proxy under Lambda concurrency N (set target) without errors.

Audit & Trace
- Audit tables receive entries for 100% writes with actor, version, and Turn Trace correlation id.

Migrations
- Zero-downtime migrations verified: apply up/down on staging and restore previous version without data loss.

Security
- All DB access uses VPC and Secrets Manager; credentials never logged; TLS in-transit enforced.

Resilience
- PITR restore tested; data validated against pre-restore snapshot for last 24h window.

