Title: Neo4j — Overview
Status: Draft v1
Scope: Primary graph for skills, goals, actions, relationships, and reasoning paths; lineage back to Postgres.

Purpose
- Enable deep path queries and traversal-based reasoning not well-suited to relational schemas.
- Provide visibility into action dependencies and story/goal linkage across turns.

Key Decisions
- Neo4j Aura (managed) or self-managed on AWS EC2 with APOC; official Python driver.
- Each node references its canonical Postgres id and Turn Trace lineage id where applicable.
- Indices on node labels and key properties; constraints to ensure uniqueness and referential integrity.

Synchronization
- Postgres is the system of record; change events flow to Neo4j for projection.
- Periodic reconciliation job validates counts and spot-checks property parity.

Query Patterns
- Shortest path between skills and goals; dependency expansion for execution planning; community detection if needed.

Risks
- Divergence from Postgres; mitigated by reconciliation and compensating updates.
- Cost at scale; partitioning or tier upgrades may be required.

