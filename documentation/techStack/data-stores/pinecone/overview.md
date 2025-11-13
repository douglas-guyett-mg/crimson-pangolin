Title: Pinecone — Overview
Status: Draft v1
Scope: Primary dense vector store for Working Memory hybrid retrieval (dense + sparse re-ranking in app).

Purpose
- Store and query embeddings for conversational context, episodic memory, and knowledge items.
- Enable fast top‑k semantic search with metadata filtering and stable IDs linking to Postgres.

Key Decisions
- Index dimension aligned to chosen embedding model; cosine similarity metric.
- One index per environment; namespace per tenant/domain as needed.
- Metadata mirrors essential fields (type, org, created_at, permissions hash) for filterable queries.

Write & Update Policy
- Idempotent upserts keyed by external_id (Postgres row id); partial updates allowed.
- Deletions mirrored from Postgres tombstones; periodic vacuum of stale vectors.

Query Policy
- Top‑k tuned to cost/latency; rerank in app if needed; hybrid retrieval uses BM25/FTS from Postgres.

Operations
- Autoscaling enabled; region aligned with AWS region; Secrets Manager for API key.
- Re-embedding jobs scheduled on model/version changes; drift monitored via recall on canary sets.

Risks
- Dimension mismatch; ensure gateway validates before upsert.
- Cost growth; compact/summarize and TTL policies for low-value items.

