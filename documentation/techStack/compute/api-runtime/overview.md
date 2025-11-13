Title: API Runtime (API Gateway + Lambda + FastAPI) — Overview
Status: Draft v1
Scope: Primary API surface for channels and internal services. SSE-capable responses.

Purpose
- Serve channel webhooks and user API requests with low ops and auto-scaling.
- Provide SSE streaming endpoints for web/mobile.

Key Decisions
- API Gateway HTTP API fronting Lambda; FastAPI app via Mangum adapter.
- Pydantic v2 for request/response typing; strict validation.
- RDS Proxy for Postgres connections; boto3 for AWS integrations.
- Idempotency via DynamoDB keys for sensitive operations.

Endpoints (examples by category; non-exhaustive)
- Turn initiation; tool/action execution; streaming token endpoint.
- Admin/health: provider health, budget status, dependencies healthchecks.

Operational Concerns
- Timeouts: keep within Lambda/APIGW limits; ensure server-sent events flush frequently.
- Logging: structured logs with correlation ids; emit Turn Trace write events.
- Security: Cognito authorizers or custom JWT validation; WAF on API Gateway.

Risks
- Cold starts: mitigate via provisioned concurrency for hot paths if necessary.
- Long-lived streams: move to ECS service if connection limits bind.

