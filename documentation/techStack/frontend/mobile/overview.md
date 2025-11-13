Title: Mobile Frontend (React Native) — Overview
Status: Draft v1
Scope: RN app with Cognito auth, SSE streaming, and resilient network handling.

Architecture
- React Native (TypeScript; Expo recommended) with Amplify/Auth for Cognito.
- SSE client with background reconnect and lifecycle-aware listeners.

Auth & Security
- Secure token storage: iOS Keychain / Android Keystore; no secrets in logs.
- Deep-link OIDC callback handling; refresh tokens managed by Amplify.Auth.

Streaming
- SSE handler processes token, heartbeat, error, done events; app shows incremental tokens.
- Backoff and jitter on reconnect; prevents leaks on app state changes.

Operational
- Feature flags for model/provider selection; per-org quotas enforced via headers.
- Basic telemetry: latency, errors, stream drops; sent to backend or third-party as allowed.

