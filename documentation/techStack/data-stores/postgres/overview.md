Title: Postgres (RDS) — Overview
Status: Draft v1
Scope: System of record for Frontal Cortex, sessions, auth linkage, Working Memory metadata, and audit adjunct tables.

Purpose
- Provide transactional integrity, optimistic locking, and durable storage for core entities (Goals, PendingActions, Questions, Stories).
- Support JSONB for flexible attributes; support hybrid retrieval metadata and indexes.

Key Decisions
- Amazon RDS Postgres (Multi‑AZ) with RDS Proxy for Lambda connection management.
- SQLAlchemy 2.x with Pydantic models as typed schema boundary.
- pgvector present for backup/analysis but Pinecone is primary vector store.
- GIN indexes on JSONB; FTS where needed; partition large audit tables with pg_partman.

Concurrency & Audit
- Optimistic locking with version columns; conflict detection mapped to FC rules.
- Write-ahead audit tables capture mutations with actor, timestamp, causality id (Turn Trace linkage).

Security & Access
- VPC-only; IAM auth or standard user/pass via Secrets Manager.
- Role separation for read/write/analytics; least privilege for Lambdas.

Operations
- Daily snapshots and PITR; migration via Alembic with forward/backward compatibility.
- Observability via pg_stat_statements; slow query logging and query plans tracked.

Risks
- Lambda connection storms → use RDS Proxy; pool size tuned.
- Large tables → partitioning and archiving strategy; S3 export for cold data.

