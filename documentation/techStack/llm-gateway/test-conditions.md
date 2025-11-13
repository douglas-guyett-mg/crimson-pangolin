Title: LLM Gateway — Test Conditions
Status: Draft v1

Functional
- Chat streaming works for Bedrock, OpenAI, Gemini, Anthropic with token deltas, heartbeats, and finalization event.
- Embeddings: vectors generated for all providers; dimensions match configured indexes (e.g., Pinecone dim).
- Routing: policy selects default providers per category; override per daemon is honored.

Budget & Safety
- Requests exceeding daemon budget are denied or auto‑reduced; decision logged in Turn Trace.
- Safety: provider safety settings applied; blocklisted terms are rejected with appropriate status.

Reliability
- Retries/backoff applied on transient errors; non‑retryable errors surface correctly.
- Circuit breaker trips on repeated failures and recovers after cool‑down.
- Rate limits enforced per org/user/provider; excess returns 429 with retry-after.

Performance
- P95 time-to-first-token: web ≤ 800ms, mobile ≤ 2s (under nominal network conditions).
- P95 end-to-end completion latency within Turn budget (<50s) including RAG when enabled.

Telemetry & Audit
- Start/partial/finish/error events recorded to Turn Trace with model, tokens, cost estimate, latency.
- OpenTelemetry spans emitted for provider calls with error attributes on failure.

Security
- Secrets resolved from Secrets Manager via IAM; no plaintext keys in environment or logs.
- All provider calls over TLS; headers sanitized in logs; PII redacted when configured.

Compatibility
- Embedding dimension mismatch is detected and flagged before upsert; Pinecone upserts succeed after correction.
- SSE token stream conforms to streaming layer contract (message types, heartbeats, termination).

