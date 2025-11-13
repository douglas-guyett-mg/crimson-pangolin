Title: Streaming Transport (SSE-first) — Overview
Status: Draft v1
Scope: Transport for token streaming to web (Next.js) and mobile (React Native); fallback to WebSockets if needed.

Decision
- Default to Server-Sent Events for simplicity, cost, and one-way streaming fit.
- Consider WebSockets for bidirectional control or extreme session concurrency needs.

SSE Contract (conceptual)
- Message types: token, heartbeat, error, done.
- Heartbeat cadence to keep intermediaries alive and enable client-side health.
- Reconnect strategy with exponential backoff; resume not guaranteed.

Client Guidance
- Web: Next.js route proxies to AWS SSE; ensure no buffering at Vercel edge for streaming routes.
- Mobile: RN SSE client with background reconnect support; treat app lifecycle events carefully.

Operational Considerations
- API Gateway/Lambda tuning to avoid idle timeouts; chunked responses.
- Backpressure: gateway flush frequency and chunk sizes tuned to meet SLOs.

Risks
- Intermediary buffering; test across browsers and networks.
- Mobile network variability; implement robust reconnection logic.

