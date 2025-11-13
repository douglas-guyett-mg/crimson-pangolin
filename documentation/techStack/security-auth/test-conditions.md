Title: Security and Auth — Test Conditions
Status: Draft v1

Authentication
- Web: NextAuth ↔ Cognito login succeeds; refresh/rotation works; logout invalidates sessions.
- Mobile: Amplify/Auth handles login/refresh; tokens stored securely (Keychain/Keystore); logout clears tokens.

Authorization
- JWT claims drive RBAC; unauthorized endpoints return 403; admin scope required for admin APIs.
- Postgres write attempts without proper role are rejected and audited with actor id.

Secrets & Keys
- All provider keys retrieved from Secrets Manager at runtime via IAM; no plaintext in env/logs.
- Key rotation test: update Secrets Manager → new calls use rotated key without deploy.

Network & Perimeter
- WAF rules block common attacks (SQLi/XSS patterns) and abusive IPs; logs visible in CloudWatch.
- VPC endpoints used for S3/SQS/Secrets; outbound to third-party APIs restricted to required domains.

Data Protection
- S3 objects encrypted; signed URL access requires valid signature and expires as configured.
- RDS requires TLS; connection without TLS is rejected.

Auditability
- Turn Trace records actor and correlation ids for write operations; sensitive fields redacted per policy.

Rate Limiting & Quotas
- Per-user/org/provider token quotas enforced; exceeding results in 429 with retry-after.

