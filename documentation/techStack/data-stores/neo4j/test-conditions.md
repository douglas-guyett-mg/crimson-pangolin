Title: Neo4j — Test Conditions
Status: Draft v1

Correctness
- Projection fidelity: nodes and relationships in Neo4j match Postgres-derived source for a sample set (define size).
- Referential integrity: all nodes with Postgres ids resolve; no orphan relationships.

Performance
- P95 path queries complete within defined thresholds (set target, e.g., ≤ 200ms) for baseline graph size.

Synchronization
- Event-driven updates apply within SLA (e.g., ≤ 1m); periodic reconciliation fixes drifts and logs discrepancies.

Security
- Credentials in Secrets Manager; TLS enforced; limited network access (VPC peering or IP allow-list as applicable).

