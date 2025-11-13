Title: Mobile Frontend — Test Conditions
Status: Draft v1

iOS & Android
- Login, refresh, and logout flows succeed; tokens stored securely (Keychain/Keystore).
- SSE streams deliver first token ≤ 2s; survive intermittent connectivity with automatic reconnect.

Resilience
- App background/foreground transitions do not leak connections; streams resume or fail cleanly.
- Error states display helpful messages; retries back off and cap.

Security & Telemetry
- No secrets in logs; crash reports scrub PII.
- Telemetry events for stream starts/stops/errors visible in backend dashboards.

