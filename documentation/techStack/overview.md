# Si Tech Stack — AWS‑first with Pinecone, Neo4j, and Multi‑LLM Providers

Status: v1.0 (Initial)  
Scope: Concrete technology selections and build/run guidance aligned to Si specs (Turn, WM, FC, First‑Mile, Turn Trace)

## 1) Goals and constraints
- AWS‑first; minimize third parties except where explicitly chosen
- Start with Pinecone (vector) and Neo4j (graph)
- Typed Python backend; Next.js on Vercel (web); React Native (mobile)
- Multi‑provider LLM: AWS Bedrock + direct OpenAI, Google Gemini, Anthropic Claude
- Streaming UX for web/mobile; SMS tolerates seconds
- Strong audit via Turn Trace; background work continues after initial response

## 2) High‑level architecture
- Frontends
  - Web: Next.js on Vercel (TypeScript, RSC/Edge streaming; NextAuth + Cognito)
  - Mobile: React Native (TypeScript, Expo recommended); SSE for token streams
- Backend compute
  - API Gateway + Lambda (Python 3.11, FastAPI via Mangum) for APIs/webhooks
  - SQS + Lambda for async/background (summarization, compaction, analyzers)
  - EventBridge for domain events; Step Functions only for long batch flows
  - Optional: ECS Fargate for long‑lived streaming or if Lambda limits become binding
- Storage and messaging
  - Relational: Amazon RDS Postgres (FC system of record, auth/session, CH/WM metadata)
  - Vector: Pinecone (primary) — upsert/query embeddings for WM hybrid retrieval
  - Graph: Neo4j (primary) — skills, goals, actions, relationships, reasoning paths
  - Object: S3 (+ CloudFront) — transcripts, audio, Turn Trace, exports
  - Queue/Bus: SQS (Std/FIFO per ordering needs), EventBridge (fan‑out)
  - Cache/kv: DynamoDB (sessions, rate limits, idempotency keys)
- LLM providers and embeddings
  - Bedrock (Claude, Llama, Mistral) with streaming
  - Direct: OpenAI, Google Gemini, Anthropic (Claude) APIs
  - Embeddings: Bedrock embeddings or OpenAI text‑embedding‑3‑small; batch where possible
- Observability & audit
  - Turn Trace: write‑only during turn → S3 JSONL; Athena/Glue for analysis; CW Logs tee for live debug
  - OpenTelemetry tracing (Python + Node) → CloudWatch/X‑Ray; derived business metrics from trace

## 3) Component mapping to specifications
- Turn/Parallelism/Response‑Timing → Lambda + SQS/EventBridge; async/await in FastAPI; background continuations
- Daemon Coordination Protocol (timeouts, ordering) → SQS FIFO where required + app‑level policies
- FC Persistence (optimistic locking, ACL, audit) → Postgres with version columns + mutation tables
- Working Memory (hybrid retrieval, budgets) → Pinecone + Postgres metadata; budget enforced in LLM gateway
- Turn Trace (write‑only; analysis later) → S3 (+ Athena) with cheap analytics

## 4) Service choices and initial sizing
- RDS Postgres: db.t4g.medium, Multi‑AZ; pg_partman for large audit tables; GIN indexes on JSONB
- Pinecone: 768/1024 dims (match embedding model), cosine; 1 small pod per env; enable pod autoscaling
- Neo4j: AuraDB Professional (or self‑managed on AWS EC2) with 2‑3 vCPU equivalent; APOC enabled
- S3: lifecycle to Glacier for Turn Trace after 30 days; default SSE‑S3, bucket‑key enabled
- API Gateway/Lambda: HTTP API; 1024–2048 MB memory (tune for CPU/network); provisioned concurrency only if needed
- SQS: Standard queues by default; FIFO for ordered daemon interactions (declare explicitly)
- DynamoDB: On‑demand; TTL for sessions and rate‑limit counters
- CloudFront: front S3 for media; signed URLs

## 5) LLM gateway design (multi‑provider)
- Single typed Python service (SDK or service boundary) that implements:
  - Providers: Bedrock, OpenAI, Google Gemini, Anthropic
  - Capabilities: chat/completions, streaming, embeddings
  - Budget enforcement: global + daemon‑specific budgets; prompt simplification hooks per daemon
  - Safety: per‑provider safety params; content filters; configurable blocklists
  - Retries/backoff: provider‑specific transient error handling; circuit‑breaker per model
  - Rate limiting: token bucket per org/user/provider; leverage DynamoDB counters
  - Telemetry: emit ToolInvocation + Budget events to Turn Trace; OTel spans
- Routing policy examples
  - Quick routing: Router/Executor small prompts → Claude via Bedrock (low‑latency/stream)
  - Planning: FC → Claude Sonnet (Bedrock or direct Anthropc), fallback to GPT‑4o or Gemini 1.5 Pro
  - Code/structured: GPT‑4o‑mini for tool schema generation; Gemini for vision if needed
  - Embeddings: “lowest cost acceptable” first, fallback sequentially

## 6) Data model and flows
- Postgres holds canonical entities (FC Goals, PendingActions, Questions, Stories), auth, sessions, WM/CH metadata
- Pinecone holds dense vectors with external_id → Postgres row IDs; metadata mirrors key fields
- Neo4j holds relationships for skills/goals/actions, path queries, lineage; periodically reconciled from Postgres
- Turn Trace events stream to S3; near‑real‑time aggregates published to CloudWatch via Lambda consumer

## 7) Security and networking
- Secrets: AWS Secrets Manager for Pinecone, Neo4j Aura, OpenAI, Gemini, Anthropic keys; rotation policy defined
- VPC: RDS/EC2 Neo4j within VPC; NAT for egress to third‑party APIs; VPC endpoints for S3, SQS, Secrets
- Egress minimization: co‑locate Pinecone region with AWS region; choose Neo4j region closest to AWS
- AuthZ: Cognito (users) + IAM (workload); API Gateway authorizers; signed URLs for S3/CloudFront
- Data protection: SSE on S3; RDS encryption; TLS to Pinecone/Neo4j; redaction gates in WM write path
- WAF: Attach to CloudFront/API Gateway; bot/abuse throttling

## 8) Frontend integration
- Web (Vercel)
  - NextAuth + Cognito OIDC; edge/serverless routes for streaming from AWS backend (SSE)
  - Generate TypeScript SDK from FastAPI OpenAPI
- Mobile (React Native)
  - Cognito via Amplify/Auth library; SSE client for token streams; background reconnect
- Realtime choice
  - Default SSE for simplicity and cost; promote to WebSockets (API Gateway or Fargate) if session semantics demand

## 9) Libraries (Python; typed)
- API: FastAPI, Pydantic v2, Uvicorn, Mangum
- AWS: boto3/botocore, aws‑lambda‑powertools
- Postgres: SQLAlchemy 2.x, asyncpg / psycopg
- Pinecone: pinecone‑client
- Neo4j: neo4j (official Python driver)
- LLMs: boto3‑bedrock‑runtime, openai, google‑generativeai, anthropic
- Async/HTTP: httpx, anyio
- Observability: opentelemetry‑sdk, opentelemetry‑instrumentation‑fastapi, structlog/loguru
- Tooling: mypy (strict), ruff, pytest, pre‑commit

## 10) Environment variables (names only)
- PINECONE_API_KEY, PINECONE_ENV, PINECONE_INDEX
- NEO4J_URI, NEO4J_USER, NEO4J_PASSWORD
- BEDROCK_REGION (plus IAM role on runtime)
- OPENAI_API_KEY, GEMINI_API_KEY, ANTHROPIC_API_KEY
- DATABASE_URL (RDS), AWS_REGION, S3_BUCKET_TURN_TRACE, S3_BUCKET_MEDIA
- COGNITO_USER_POOL_ID, COGNITO_CLIENT_ID, NEXTAUTH_SECRET

## 11) Setup steps (minimum viable)
1. Provision RDS Postgres (Multi‑AZ); apply schemas for FC + audit + WM metadata
2. Create S3 buckets: media, turn‑trace; attach lifecycle policies; set bucket policies for VPC endpoints
3. Configure Pinecone project + index (dims, metric); set API key in Secrets Manager; allow‑list IP if applicable
4. Create Neo4j Aura DB (or EC2 self‑managed); store creds in Secrets Manager; enable APOC
5. Deploy API Gateway + Lambda stacks (FastAPI via Mangum); attach IAM role with Secrets, SQS, S3, RDS access
6. Create SQS queues (standard; FIFO where ordering is required); add DLQs
7. Configure EventBridge rules for turn lifecycle and trace aggregation
8. Set Cognito user pool + app client; NextAuth Cognito provider on Vercel
9. Enable OTel exporters and CloudWatch dashboards; wire Turn Trace writer to S3; Athena/Glue catalog for queries
10. Configure LLM gateway secrets and model routing; verify streaming for all providers

## 12) Cost controls
- Batch embeddings; shard Pinecone indexes by tenant or domain; set HNSW/IVF parameters conservatively
- TTL and compaction in WM; summarization frequency tuned by budget
- Use lower‑cost models by default (Claude Haiku, GPT‑4o‑mini, Gemini 1.5 Flash) with policy‑based overrides
- DynamoDB on‑demand with TTL; scale RDS vertically only under load tests; autoscaling on Lambda concurrency
- S3 lifecycle to Glacier; Athena partitioning by date to cut scan costs

## 13) Verification checklist
- Web/mobile streaming P95 meets SLOs (web ≤ 800ms token start; mobile ≤ 1–2s)
- Turn Trace written for 100% turns; Athena query returns last 24h traces
- FC optimistic locking: concurrent write tests pass per spec
- WM hybrid retrieval returns relevant context; Pinecone queries < 100ms at baseline scale
- LLM gateway: streaming, budget enforcement, retry policies verified for Bedrock, OpenAI, Gemini, Anthropic
- Security: secrets fetched via IAM; no plaintext keys in env; TLS to Pinecone/Neo4j verified

## 14) Upgrade & alternatives
- Compute: Promote key workloads to ECS Fargate if Lambda concurrency/cold‑start/connection limits bind
- Realtime: Switch to WebSockets if bidirectional control required; consider dedicated ECS service
- Search: Add OpenSearch for logs/analytics if Athena latency is insufficient
- Graph at scale: Partition Neo4j or migrate to larger Aura tiers; evaluate Neptune (opencypher) if staying fully AWS

## 15) Alignment with Si core tenants
- Documentation‑as‑code: prescriptive, testable selections
- Tests‑first: checklist mapped to SLOs/specs; unit/integration tests for FC/WM/LLM
- Modularity: pluggable LLM providers, independent stores (RDS, Pinecone, Neo4j)
- Technology‑agnosticism: clear interfaces; swaps possible with defined triggers

