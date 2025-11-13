Title: Security and Auth — Overview
Status: Draft v1
Scope: Authentication, authorization, secrets, network boundaries, and data protection for Si across web, mobile, and backend.

Goals
- Minimize blast radius and third-party exposure while enabling multi-provider LLM access.
- Enforce least privilege, strong encryption, and auditable actions tied to Turn Trace.

Authentication
- Users: Amazon Cognito (OIDC) as the identity provider.
  - Web: NextAuth with Cognito provider on Vercel.
  - Mobile: Amplify/Auth library; secure token storage (Keychain/Keystore).
- Service-to-service: IAM roles for AWS resources; short-lived credentials.

Authorization
- API: Cognito JWT authorizers on API Gateway; scopes/claims drive RBAC.
- Data: Postgres row-level access enforced at service layer; audit all writes with actor id.
- Admin: Separate admin app/client with constrained scopes; break-glass process documented.

Secrets & Keys
- AWS Secrets Manager for: OpenAI, Gemini, Anthropic, Pinecone, Neo4j, NextAuth, DB credentials.
- KMS-managed encryption for secrets at rest; rotation policy for all third-party keys.
- No plaintext secrets in env or logs; load via IAM at runtime.

Network & Perimeter
- VPC for RDS/EC2 workloads; VPC endpoints for S3, SQS, Secrets Manager.
- NAT for controlled egress to third-party APIs; restrict outbound with security groups.
- WAF on CloudFront and API Gateway; basic bot/abuse protections and IP throttling.

Data Protection
- Encryption in transit (TLS 1.2+) to all services; at rest (S3 SSE, RDS, Pinecone/Neo4j managed enc).
- PII redaction gates in Working Memory write path; configurable allowlist/denylist.
- Signed URLs for S3 media; short TTL; least-privilege bucket policies.

Compliance & Auditability
- Turn Trace: include actor, request ids, and causality; redact sensitive content per policy.
- Access logs for API Gateway/CloudFront; OTel tracing across services for forensics.

Threats & Mitigations
- Credential leakage → IAM + Secrets Manager + zero plaintext policy + CI checks.
- Abuse/spam → WAF rules + rate limits + org/user quotas in DynamoDB.
- SSRF/egr. data exfiltration → VPC endpoints, egress controls, domain allow-lists.

