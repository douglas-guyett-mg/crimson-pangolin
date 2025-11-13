Title: Observability and Turn Trace Analytics — Test Conditions
Status: Draft v1

Tracing & Logs
- 100% API requests produce OTel traces with correlation ids; child spans for LLM calls and DB.
- Structured logs include trace id and actor id where applicable; search yields a full request timeline.

Metrics & Dashboards
- Dashboards display: TTFB, error rates per provider, DLQ backlog, Lambda concurrency, RDS connections.
- Alarms fire on threshold breach (e.g., TTFB p95 > target, DLQ > 0, error rate > 2%).

Athena Analytics
- Athena query returns last 24h Turn Trace stats under 30s; partition pruning verified.
- Event taxonomy consistent; no missing required fields (model, tokens, cost, latency).

Retention & Security
- S3 lifecycle transitions verified after TTL; CloudWatch log retention matches policy.
- IAM policies restrict Athena/S3 access to analytics role; access is audited.

