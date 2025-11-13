Title: Web Frontend — Test Conditions
Status: Draft v1

Auth
- NextAuth login with Cognito succeeds; session reflects JWT claims; logout clears session.

Streaming
- SSE proxy delivers first token ≤ 800ms on baseline network; no buffering by Vercel for stream routes.
- UI renders token stream smoothly; handles heartbeats and error events.

SDK & Types
- Generated SDK compiles; request/response types align with FastAPI OpenAPI; CI verifies no breaking changes.

Security
- CSP blocks inline scripts; secrets not present in client code; auth routes protected against CSRF.

Observability
- Frontend traces correlate to backend via headers; dashboards show page/api latency and error rates.

