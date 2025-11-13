Title: Observability and Turn Trace Analytics — Overview
Status: Draft v1
Scope: Logging, metrics, tracing, and analytics spanning API, workers, LLM calls, and frontends.

Observability Stack
- Tracing: OpenTelemetry SDKs (Python/Node) → CloudWatch/X-Ray; propagate trace ids across services.
- Logs: Structured logs (JSON) with correlation ids; API Gateway/CloudFront access logs enabled.
- Metrics: Business and SLO metrics derived from traces (TTFB, token rate, costs, error rates).

Turn Trace Analytics
- Write-only JSONL to S3 during turns; partitioned by dt=YYYY/MM/DD/HH.
- Glue catalog + Athena for querying; dashboards for volume, latency, error classes, model usage, cost.
- Near-real-time: EventBridge → Lambda aggregator → CloudWatch dashboards/alarms.

Dashboards & Alerts
- SLOs: web/mobile TTFB, turn completion rate, error rates per provider, DLQ backlog, budget breaches.
- Cost: per-model, per-org, per-daemon token/cost; monthly forecast.
- Infra: Lambda concurrency, API errors, RDS connections, SQS age, Pinecone/Neo4j health.

Retention & Cost Control
- S3 lifecycle to Glacier after 30 days; CloudWatch log retention tuned per env.
- Athena partition pruning and compaction jobs to reduce scan costs.

Security & Compliance
- Pseudonymized identifiers in analytics; access limited via IAM; audit access to sensitive datasets.

