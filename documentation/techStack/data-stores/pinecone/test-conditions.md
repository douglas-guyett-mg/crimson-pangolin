Title: Pinecone — Test Conditions
Status: Draft v1

Correctness
- Upsert idempotency: repeated upserts for same external_id do not create duplicates; metadata merged as defined.
- Delete propagation: tombstoned items in Postgres are removed from Pinecone within SLA (define target, e.g., ≤ 5m).

Performance
- P95 query latency ≤ 100ms for top‑k=10 at baseline scale; p99 ≤ 200ms.
- Upsert throughput meets ingestion targets without throttling; backoff and retry applied when rate-limited.

Relevance
- Hybrid retrieval returns relevant items on canary query set; minimum recall/precision thresholds defined.

Safety & Security
- API key stored in Secrets Manager; calls over TLS; no keys logged.
- Embedding dimension mismatches detected and rejected by gateway before upsert.

Operations
- Re-embedding pipeline validates new vectors on sample before full rollout; rollback plan documented.

