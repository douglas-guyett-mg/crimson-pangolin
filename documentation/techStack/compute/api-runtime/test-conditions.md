Title: API Runtime — Test Conditions
Status: Draft v1

Latency & Throughput
- P95 latency for common endpoints ≤ 200ms (no streaming). Cold-start P95 measured and documented.
- SSE endpoint time-to-first-byte meets First‑Mile SLOs (web ≤ 800ms; mobile ≤ 2s) under baseline load.

Correctness
- Request validation rejects malformed requests with clear errors; contracts round-trip in client SDK generation.

Resilience
- Idempotency keys prevent duplicate processing under retries.
- Graceful degradation when dependencies (LLM, DB) fail; appropriate HTTP status and trace entries.

Security
- AuthZ enforced via Cognito/JWT; unauthorized requests rejected; audit trail records actor.

Observability
- Structured logs with correlation ids; OTel spans covering entire request including streaming period.

