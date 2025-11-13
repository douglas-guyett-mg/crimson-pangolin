Title: Web Frontend (Next.js on Vercel) — Overview
Status: Draft v1
Scope: Next.js app delivering streaming UX, Cognito auth via NextAuth, and typed SDK for API.

Architecture
- Next.js (App Router) with server actions/routes for streaming SSE from AWS backend.
- NextAuth configured for Cognito OIDC; session strategy aligned with backend JWT claims.
- Generated TypeScript SDK from FastAPI OpenAPI; shared type safety.

Streaming
- SSE proxy route disables buffering and flushes tokens promptly; backpressure aware.
- UI components display incremental tokens; handle heartbeats and finalization.

Security
- Env secrets via Vercel project env; no secrets in client bundle.
- CSRF protection on auth routes; strict CSP; sanitize user-provided content.

Performance & UX
- Edge runtime for lightweight routes; server runtime for SSE proxy.
- Progressive rendering; error boundaries for tool/LLM failures.

Operational
- Observability: OTel web instrumentation for page/api metrics; correlate with backend via headers.
- Multi-env configs (dev/stage/prod) with separate Cognito apps and API endpoints.

