# Technology Stack

This document defines the complete technology stack for the Crimson Pangolin (Si) system.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│ FRONTEND (Vercel)                                               │
│ └─ Next.js 14 + TypeScript + Edge Functions                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ SUPABASE (Source of Truth)                                      │
│ ├─ PostgreSQL 15+ with pgvector extension                      │
│ ├─ Auth (JWT-based user authentication)                        │
│ ├─ Realtime (WebSocket for streaming responses)                │
│ └─ Storage (audio files, turn trace archives)                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ AWS CORE                                                         │
│ ├─ Lambda (turn executors, daemon orchestration)               │
│ ├─ SQS FIFO (turn processing queue, preserves order)           │
│ ├─ ElastiCache Redis (working memory cache, session state)     │
│ ├─ S3 (turn trace archives, audio storage)                     │
│ ├─ CloudWatch (monitoring, logs, metrics, alarms)              │
│ └─ Secrets Manager (API keys, credentials)                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌───────────────────────────┬─────────────────────────────────────┐
│ PINECONE SERVERLESS       │ NEO4J AURA                          │
│                           │                                     │
│ Vector Similarity Search  │ Graph Relationships & Traversal     │
│ • Semantic memory facts   │ • Memory interconnections           │
│ • Hybrid dense+sparse     │ • Turn trace causality chains       │
│ • 10-30ms p95 latency     │ • Tool dependency graphs            │
│                           │ • 20-60ms p95 latency               │
└───────────────────────────┴─────────────────────────────────────┘
```

## Core Technology Decisions

### Frontend Layer

**Vercel + Next.js 14**
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript 5+
- **Styling**: Tailwind CSS 3+
- **UI Components**: Radix UI (WCAG 2.1 Level AA compliant)
- **State Management**: Zustand
- **WebSocket**: Supabase Realtime client
- **Audio**: Web Audio API + MediaRecorder API
- **Real-time**: Server-Sent Events (SSE) for streaming responses

**Edge Functions** (Vercel Edge Runtime):
- Router daemon (quick decisions < 50ms)
- Consent validation
- Audio upload handling

**Rationale**:
- Edge functions reduce latency for simple queries
- Next.js streaming SSR perfect for real-time responses
- Vercel auto-scaling handles traffic spikes
- Free tier sufficient for MVP

---

### Database Layer (Source of Truth)

**Supabase (PostgreSQL 15+)**

**Extensions Enabled**:
```sql
CREATE EXTENSION IF NOT EXISTS vector;      -- pgvector for embeddings
CREATE EXTENSION IF NOT EXISTS pg_trgm;     -- Trigram matching
CREATE EXTENSION IF NOT EXISTS btree_gin;   -- Multi-column indexes
```

**Schema Organization**:
- `public` schema: Core tables (users, sessions, turns)
- `memory` schema: All memory types (episodic, semantic, conversation)
- `trace` schema: Turn trace events, observability

**Key Features Used**:
- **Row Level Security (RLS)**: User data isolation
- **Realtime**: WebSocket subscriptions for streaming
- **Auth**: Email/password, OAuth (Google, GitHub)
- **Storage**: Audio files, user uploads
- **Functions**: Postgres functions for complex queries

**Rationale**:
- Single vendor for DB + Auth + Realtime + Storage
- pgvector sufficient for episodic memory (less critical than semantic)
- Free tier for MVP ($0), Pro tier ($25) for production
- Can self-host later if needed (Docker available)

---

### Vector Search

**Pinecone Serverless**

**Index Configuration**:
- **Dimensions**: 1536 (OpenAI ada-002) or 1024 (Anthropic embeddings)
- **Metric**: Cosine similarity
- **Pods**: Serverless (auto-scaling)
- **Hybrid Search**: Sparse + Dense vectors

**Index Structure**:
```python
{
    "id": "mem_<ulid>",
    "values": [0.1, 0.2, ...],  # Dense embedding (1536-dim)
    "sparse_values": {
        "indices": [45, 123, 789],  # Keyword indices
        "values": [0.8, 0.6, 0.4]   # TF-IDF scores
    },
    "metadata": {
        "user_id": "user_<ulid>",
        "fact": "User prefers dark mode",
        "confidence": 0.95,
        "created_at": "2025-11-08T12:00:00Z",
        "version": 2
    }
}
```

**Indexes Created**:
- `semantic-memory-prod`: Production semantic memories
- `semantic-memory-dev`: Development/testing

**Rationale**:
- 3-5x faster than pgvector for vector search (10-30ms vs 50-200ms)
- Hybrid search critical for semantic memory accuracy
- Serverless = pay per query, no idle costs
- Free tier (100k vectors) for MVP

---

### Graph Database

**Neo4j Aura (Managed Cloud)**

**Graph Schema**:

**Node Types**:
- `Memory`: Semantic memory facts
- `Turn`: Turn execution records
- `Daemon`: Daemon invocations
- `Tool`: Tool executions
- `User`: User entities

**Relationship Types**:
- `SUPPORTS`: One memory supports another (strength: 0.0-1.0)
- `CONTRADICTS`: Conflicting memories (strength: 0.0-1.0)
- `RELATES_TO`: General association (strength: 0.0-1.0)
- `INVOKED`: Turn/daemon invoked another (order, timestamp)
- `DEPENDS_ON`: Tool/action dependencies
- `SPAWNED`: Turn spawned sub-turn

**Example Cypher Queries**:
```cypher
// Find connected memories (graph expansion)
MATCH (seed:Memory)-[r:SUPPORTS|RELATES_TO*1..2]-(related:Memory)
WHERE seed.id IN $seed_ids
  AND r.strength > 0.5
  AND related.user_id = $user_id
RETURN DISTINCT related
ORDER BY related.confidence DESC
LIMIT 30

// Find turn bottlenecks
MATCH (t:Turn)-[:INVOKED]->(d:Daemon)
WHERE t.created_at > datetime() - duration('P7D')
RETURN d.name, avg(d.execution_time_ms) as avg_time
ORDER BY avg_time DESC
LIMIT 10

// Find tool usage patterns
MATCH (u:User)-[:EXECUTED]->(t1:Tool)-[:FOLLOWED_BY]->(t2:Tool)
WHERE u.id = $user_id
RETURN t1.name, t2.name, count(*) as frequency
ORDER BY frequency DESC
```

**Indexes**:
```cypher
CREATE CONSTRAINT memory_id IF NOT EXISTS FOR (m:Memory) REQUIRE m.id IS UNIQUE;
CREATE CONSTRAINT user_id IF NOT EXISTS FOR (u:User) REQUIRE u.id IS UNIQUE;
CREATE INDEX memory_user IF NOT EXISTS FOR (m:Memory) ON (m.user_id);
CREATE INDEX turn_timestamp IF NOT EXISTS FOR (t:Turn) ON (t.created_at);
```

**Rationale**:
- Best-in-class for graph traversal (20-60ms for 2-3 hop queries)
- Cypher query language intuitive for complex relationships
- Critical for memory interconnections and turn trace analysis
- Free tier (200k nodes) for MVP, Pro ($65) for production

---

### Compute Layer

**AWS Lambda (Python 3.11+)**

**Lambda Functions**:

| Function | Memory | Timeout | Trigger | Purpose |
|----------|--------|---------|---------|---------|
| `turn-executor` | 1024MB | 60s | SQS | Main turn orchestrator |
| `router` | 256MB | 10s | Sync invoke | Routing decisions |
| `clarifier` | 256MB | 10s | Sync invoke | Ambiguity detection |
| `frontal-cortex` | 1024MB | 30s | Sync invoke | Planning daemon |
| `responder` | 512MB | 15s | Sync invoke | Response generation |
| `evaluator` | 256MB | 15s | Sync invoke | Turn evaluation |
| `reflector` | 512MB | 20s | EventBridge | Learning (async) |
| `sync-service` | 512MB | 30s | DB trigger | 3-way data sync |
| `memory-compaction` | 1024MB | 300s | EventBridge | Nightly compaction |

**Runtime**: Python 3.11 with AWS Lambda Powertools
**Architecture**: ARM64 (Graviton2 - 20% cheaper)
**VPC**: Lambda functions in VPC for ElastiCache access
**Layers**: Shared dependencies (boto3, anthropic SDK, etc.)

**AWS ECS Fargate** (for long-running tasks):
- Tool execution (> 60s timeout)
- Large model inference
- Batch turn trace analysis

**Rationale**:
- Serverless = pay per use, auto-scaling
- Python best for LLM integration, async orchestration
- Lambda Powertools for observability
- ECS Fargate for tasks > 60s

---

**AWS SQS (Queue)**

**Queues**:
- `turn-processing.fifo`: Main turn queue (FIFO per user)
- `turn-processing-dlq.fifo`: Dead letter queue
- `sub-turn-queue.fifo`: Sub-turn spawning
- `reflection-queue`: Async learning (standard queue)

**Configuration**:
- Visibility timeout: 90s
- Message retention: 14 days
- Max receives: 3 (then DLQ)
- Content-based deduplication: Enabled

**Rationale**:
- FIFO preserves turn order per user
- Decouples turn triggering from execution
- Handles retry logic automatically

---

**AWS ElastiCache (Redis 7+)**

**Node Type**: `cache.t4g.micro` (MVP) → `cache.t4g.medium` (production)
**Cluster Mode**: Disabled (single shard)
**Replication**: 1 replica for high availability

**Use Cases**:
- Working memory cache (assembled context, 5 min TTL)
- Session state (user sessions, 24 hour TTL)
- LLM response cache (identical prompts, 1 hour TTL)
- Rate limiting (token budgets per user)

**Data Structures**:
```python
# Working memory cache
"working_memory:{turn_id}" -> JSON (5 min TTL)

# Session state
"session:{session_id}" -> JSON (24 hour TTL)

# LLM cache
"llm:{hash(prompt)}" -> JSON (1 hour TTL)

# Token budget
"budget:{user_id}:{month}" -> Counter (1 month TTL)
```

**Rationale**:
- Sub-50ms cache hits for working memory
- Reduces Supabase query load
- LLM cache saves ~30-40% on repeated queries

---

**AWS S3**

**Buckets**:
- `si-turn-traces-prod`: Turn trace archives (Glacier after 30 days)
- `si-audio-recordings-prod`: User audio (Standard-IA after 30 days)
- `si-episodic-memory-archive-prod`: Old episodic memory (Glacier Deep Archive after 1 year)

**Lifecycle Policies**:
- Turn traces: Standard (30 days) → Glacier Instant Retrieval → Glacier Deep Archive (1 year)
- Audio: Standard (30 days) → Standard-IA → Glacier (90 days)

**Rationale**:
- Cost optimization (Glacier 90% cheaper than Standard)
- Turn traces queryable for 30 days, then archival
- S3 Intelligent-Tiering for uncertain access patterns

---

**AWS CloudWatch**

**Metrics**:
- `TurnLatency` (p50, p95, p99)
- `DaemonExecutionTime` (per daemon type)
- `LLMTokenUsage` (per user, per daemon)
- `ErrorRate` (per function, per error type)
- `CostPerTurn` (estimated, per user)

**Alarms**:
- Turn latency > 50s (P95)
- Error rate > 5%
- Lambda throttling > 10/min
- ElastiCache CPU > 80%
- Monthly budget exceeded

**Dashboards**:
- Turn execution overview (latency, throughput, errors)
- Daemon performance (execution time, invocation count)
- Cost tracking (LLM tokens, AWS services)

**Rationale**:
- Native AWS integration
- Turn-level observability
- Budget monitoring critical for LLM costs

---

### LLM & AI Services

**Primary LLM**: Anthropic Claude
- **Haiku**: Router, Clarifier, Gut ($0.25/1M input tokens)
- **Sonnet 3.5**: Executor, Frontal Cortex, Reflector ($3/1M input tokens)
- **Opus**: Complex reasoning only ($15/1M input tokens)

**Backup LLM**: OpenAI GPT
- **GPT-3.5 Turbo**: Responder fallback ($0.50/1M tokens)
- **GPT-4**: Frontal Cortex fallback ($10/1M tokens)

**Embeddings**:
- **Primary**: OpenAI ada-002 ($0.10/1M tokens, 1536-dim)
- **Alternative**: Anthropic embeddings (when available)

**Speech-to-Text**: Deepgram
- Nova-2 model ($0.0043/minute)
- Real-time streaming support
- Punctuation, diarization

**Text-to-Speech**: AWS Polly
- Neural voices ($16/1M characters)
- SSML support for emphasis
- Low latency (<500ms)

**Rationale**:
- Claude best for reasoning, long context
- Tiered model approach saves 40-60% on LLM costs
- Deepgram cheapest, best real-time STT
- Polly cheapest TTS, AWS integration

---

### Communication Channels

**Twilio** (SMS, WhatsApp, Voice):
- Conversations API for thread management
- Voice escalation for complex queries
- Phone number: 1 per region

**WebSocket** (Web channel):
- Supabase Realtime for browser clients
- Fallback: Vercel WebSocket functions

**Push Notifications** (Mobile):
- Firebase Cloud Messaging (FCM)
- APNs for iOS

**Rationale**:
- Twilio industry standard for SMS/WhatsApp
- Supabase Realtime included, no extra cost
- FCM/APNs for mobile push

---

### Monitoring & Observability

**Application Monitoring**: Sentry
- Exception tracking
- Performance monitoring
- Release tracking
- User context (user_id, session_id)

**Distributed Tracing**: OpenTelemetry + AWS X-Ray
- Trace turn execution across services
- Visualize daemon call chains
- Identify bottlenecks

**Log Aggregation**: CloudWatch Logs Insights
- Centralized logging
- Query language for analysis
- Retention: 30 days

**Custom Observability**: Turn Trace
- Write-only during execution
- Read-only after completion
- Millisecond-precision timestamps
- Causality tracking (DAG)

**Rationale**:
- Turn Trace is primary observability mechanism
- Sentry for user-impacting errors
- OpenTelemetry for distributed tracing
- CloudWatch for AWS-native integration

---

## Data Sync Architecture

### Three-Way Sync (Supabase ↔ Pinecone ↔ Neo4j)

**Write Flow**:
```python
async def create_semantic_memory(user_id, fact, embedding):
    memory_id = generate_ulid()

    # Parallel writes using asyncio.gather
    await asyncio.gather(
        # 1. Supabase (source of truth)
        supabase.table('semantic_memory').insert({
            'id': memory_id,
            'user_id': user_id,
            'fact': fact,
            'embedding': embedding,
            'confidence': calculate_confidence(fact),
            'created_at': now()
        }),

        # 2. Pinecone (vector index)
        pinecone.upsert(vectors=[{
            'id': memory_id,
            'values': embedding,
            'sparse_values': generate_sparse_vector(fact),
            'metadata': {
                'user_id': user_id,
                'fact': fact,
                'confidence': calculate_confidence(fact)
            }
        }]),

        # 3. Neo4j (graph node)
        neo4j.execute_query("""
            CREATE (m:Memory {
                id: $id,
                user_id: $user_id,
                fact: $fact,
                confidence: $confidence,
                created_at: datetime($timestamp)
            })
        """, id=memory_id, user_id=user_id, fact=fact, ...)
    )

    return memory_id
```

**Eventual Consistency**:
- If any write fails → retry with exponential backoff
- If all retries fail → write to SQS DLQ for manual reconciliation
- Supabase is source of truth, can rebuild Pinecone/Neo4j

**Sync Service**:
- Lambda function triggered by Supabase DB changes (via webhook)
- Reconciles Pinecone and Neo4j with Supabase
- Runs nightly full sync check (compare counts, checksums)

---

### Query Flow (Hybrid Vector + Graph)

```python
async def query_semantic_memory(user_id: str, query: str, limit: int = 20):
    """
    Two-stage hybrid query
    """
    # Stage 1: Vector search in Pinecone (10-30ms)
    embedding = await get_embedding(query)
    sparse_vector = generate_sparse_vector(query)

    vector_results = await pinecone.query(
        vector=embedding,
        sparse_vector=sparse_vector,
        top_k=50,
        filter={'user_id': user_id},
        include_metadata=True
    )

    memory_ids = [match.id for match in vector_results.matches]

    # Stage 2: Graph expansion in Neo4j (20-60ms)
    graph_results = await neo4j.execute_query("""
        MATCH (seed:Memory)
        WHERE seed.id IN $memory_ids

        MATCH (seed)-[r:SUPPORTS|RELATES_TO*1..2]-(related:Memory)
        WHERE r.strength > 0.5
          AND related.user_id = $user_id

        RETURN DISTINCT
          related.id as id,
          related.fact as fact,
          related.confidence as confidence,
          [rel in r | rel.strength] as path_strengths,
          length(r) as hops
        ORDER BY related.confidence DESC, hops ASC
        LIMIT 30
    """, memory_ids=memory_ids, user_id=user_id)

    # Stage 3: Merge and re-rank
    combined = merge_vector_graph_results(
        vector_results.matches,
        graph_results.records
    )

    # Stage 4: Cache in Redis (5 min TTL)
    await redis.setex(
        f"semantic_query:{user_id}:{hash(query)}",
        300,
        json.dumps(combined[:limit])
    )

    return combined[:limit]
```

**Total Latency**: 30-90ms (well within 2s context assembly budget)

---

## Development Stack

**Languages**:
- **Backend**: Python 3.11+ (Lambda functions, sync service)
- **Frontend**: TypeScript 5+
- **Infrastructure**: Python (AWS CDK)
- **Queries**: SQL (Postgres), Cypher (Neo4j)

**Frameworks**:
- **Backend**: FastAPI (local dev), AWS Lambda Powertools
- **Frontend**: Next.js 14, React 18
- **Testing**: pytest, Playwright, Testcontainers

**Package Management**:
- **Python**: Poetry
- **Node**: npm/pnpm

**Code Quality**:
- **Linting**: ruff (Python), ESLint (TypeScript)
- **Formatting**: black (Python), Prettier (TypeScript)
- **Type Checking**: mypy (Python), TypeScript compiler
- **Testing**: pytest + pytest-asyncio, Jest + React Testing Library

**CI/CD**:
- **Platform**: GitHub Actions
- **Deployment**: AWS CDK (infrastructure), Vercel (frontend)
- **Testing**: Automated on PR, required to merge
- **Security**: Dependabot, Snyk scanning

---

## Infrastructure as Code

**AWS CDK (Python)**:
```python
from aws_cdk import (
    Stack,
    aws_lambda as lambda_,
    aws_sqs as sqs,
    aws_elasticache as elasticache,
)

class CrimsonPangolinStack(Stack):
    def __init__(self, scope, id, **kwargs):
        super().__init__(scope, id, **kwargs)

        # SQS Queue
        turn_queue = sqs.Queue(
            self, "TurnQueue",
            fifo=True,
            content_based_deduplication=True,
            visibility_timeout=Duration.seconds(90)
        )

        # Lambda: Turn Executor
        executor = lambda_.Function(
            self, "TurnExecutor",
            runtime=lambda_.Runtime.PYTHON_3_11,
            handler="executor.handler",
            code=lambda_.Code.from_asset("../lambda/turn-executor"),
            memory_size=1024,
            timeout=Duration.seconds(60),
            environment={
                "SUPABASE_URL": supabase_url,
                "PINECONE_API_KEY_SECRET": pinecone_secret.secret_arn,
                "NEO4J_URI": neo4j_uri,
            }
        )

        # ... more resources
```

**Supabase Migrations**:
```sql
-- migrations/001_initial_schema.sql
CREATE SCHEMA memory;

CREATE TABLE memory.semantic_memory (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id),
  fact TEXT NOT NULL,
  confidence FLOAT NOT NULL CHECK (confidence >= 0 AND confidence <= 1),
  embedding vector(1536),
  metadata JSONB DEFAULT '{}'::jsonb,
  version INTEGER DEFAULT 1,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE memory.semantic_memory ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can only access their own memories"
  ON memory.semantic_memory
  FOR ALL
  USING (auth.uid() = user_id);
```

---

## Cost Optimization Strategies

### 1. LLM Cost Reduction (Biggest Lever)
- **Tiered models**: Haiku for simple tasks, Sonnet for complex
- **Prompt caching**: Cache identical prompts (saves 40%)
- **Token budgets**: Enforce per-user limits
- **Batch non-urgent**: Reflector runs nightly, not per-turn

### 2. Infrastructure Optimization
- **Lambda ARM64**: 20% cheaper than x86
- **Spot Fargate**: 70% savings for batch jobs
- **S3 Lifecycle**: Glacier 90% cheaper for archives
- **ElastiCache right-sizing**: Start small, scale up

### 3. Query Optimization
- **Redis caching**: 30-40% cache hit rate on queries
- **Pinecone batching**: Batch upserts (cheaper than individual)
- **Neo4j query tuning**: Indexed properties, limit traversal depth

### 4. Data Retention
- Turn traces: 30 days hot → archive to S3
- Episodic memory: Compress after 90 days
- Audio files: Delete after 1 year (with user consent)

**Estimated Savings**: 40-60% reduction vs naive implementation

---

## Security & Compliance

### Authentication & Authorization
- **Supabase Auth**: JWT tokens, refresh tokens
- **Row Level Security**: Postgres-level data isolation
- **API Key Rotation**: Every 90 days (automated)
- **MFA**: Optional for users, required for admins

### Data Encryption
- **At Rest**: All databases encrypted (AES-256)
- **In Transit**: TLS 1.3 for all connections
- **Secrets**: AWS Secrets Manager with automatic rotation

### Privacy
- **GDPR Compliance**: Right to deletion, data export
- **Data Retention**: Configurable per user
- **Consent Management**: Explicit consent for audio recording
- **Anonymization**: PII removed from Turn Trace analytics

### Network Security
- **VPC**: Lambda functions in private subnets
- **Security Groups**: Least privilege access
- **API Gateway**: Rate limiting, DDoS protection
- **WAF**: Cloudflare for web application firewall

---

## Disaster Recovery

### Backup Strategy
- **Supabase**: Daily automated backups, 7-day retention
- **Neo4j**: Daily snapshots, 7-day retention
- **S3**: Versioning enabled, cross-region replication
- **Pinecone**: No backups (can rebuild from Supabase)

### Recovery Procedures
1. **Database corruption**: Restore from Supabase backup (< 1 hour RTO)
2. **Pinecone data loss**: Rebuild from Supabase (< 6 hours)
3. **Neo4j data loss**: Restore from snapshot (< 1 hour)
4. **AWS region outage**: Failover to secondary region (manual, < 4 hours)

### Recovery Time Objectives (RTO)
- **Critical (Turn execution)**: 1 hour
- **Important (Memory queries)**: 4 hours
- **Non-critical (Analytics)**: 24 hours

---

## Performance Targets

### Latency Budgets (P95)
- **Input processing**: < 100ms
- **Vector search (Pinecone)**: < 30ms
- **Graph traversal (Neo4j)**: < 60ms
- **Working memory assembly**: < 2s
- **Turn execution (total)**: < 50s
- **Response streaming (first byte)**: < 1s

### Throughput
- **Turns/second**: 10-100 (scales horizontally)
- **Concurrent users**: 1,000-10,000
- **Vector queries/sec**: 100+ (Pinecone auto-scales)
- **Graph queries/sec**: 50+ (Neo4j can scale)

### Availability
- **Web streaming channel**: 99.7% uptime (< 22 hours downtime/year)
- **Turn execution**: 99.5% success rate
- **Data durability**: 99.999999999% (S3 SLA)

---

## Vendor Summary

| Vendor | Services Used | Monthly Cost (Prod) | Free Tier |
|--------|---------------|---------------------|-----------|
| **AWS** | Lambda, SQS, ElastiCache, S3, CloudWatch | $150-250 | ✅ Generous |
| **Supabase** | Postgres, Auth, Realtime, Storage | $25 | ✅ 500MB DB |
| **Vercel** | Frontend hosting, Edge functions | $20 | ✅ Hobby tier |
| **Pinecone** | Vector search | $70 | ✅ 100k vectors |
| **Neo4j** | Graph database | $65 | ✅ 200k nodes |
| **Anthropic** | Claude LLM | $1000-3000 | ❌ Pay per use |
| **Twilio** | SMS, WhatsApp | $50-200 | ❌ Pay per use |
| **Deepgram** | Speech-to-text | $50-150 | ❌ Pay per use |
| **Sentry** | Error tracking | $26 | ✅ 5k events/month |
| **Total** | | **$1,456-3,980** | |

**Primary Vendors**: 4 (AWS, Supabase, Pinecone, Neo4j)
**Secondary Vendors**: 5 (Vercel, Anthropic, Twilio, Deepgram, Sentry)

---

## Migration Path

### Phase 1: MVP (Months 0-3)
- ✅ Vercel + Next.js
- ✅ Supabase (free tier)
- ✅ AWS Lambda + SQS (free tier)
- ✅ Pinecone (free tier)
- ✅ Neo4j (free tier)
- **Cost**: ~$100-300/month (mostly LLM)

### Phase 2: Production (Months 3-6)
- Upgrade Supabase to Pro ($25)
- Add ElastiCache Redis ($12)
- Upgrade Pinecone to Serverless ($70)
- Upgrade Neo4j to Professional ($65)
- **Cost**: ~$400-700/month

### Phase 3: Scale (Months 6-12)
- Multi-region AWS deployment
- Increase ElastiCache to medium ($50)
- Optimize LLM usage (caching, batching)
- Add ECS Fargate for long-running tasks
- **Cost**: ~$800-1500/month

### Phase 4: Enterprise (Months 12+)
- Consider self-hosting Postgres (cost reduction)
- Evaluate open-source LLMs (cost reduction)
- Multi-region Neo4j (high availability)
- Dedicated Pinecone pods (predictable costs)
- **Cost**: ~$2000-5000/month

---

## Decision Log

### Why Pinecone over pgvector alone?
- **Performance**: 3-5x faster (10-30ms vs 50-200ms)
- **Hybrid search**: Native sparse+dense support
- **Scale**: Handles 1M+ vectors effortlessly
- **Trade-off**: $70/month cost, extra vendor

### Why Neo4j over recursive CTEs?
- **Performance**: 5-10x faster for graph traversal
- **Algorithms**: PageRank, community detection built-in
- **Complexity**: Handles multi-hop queries efficiently
- **Trade-off**: $65/month cost, data sync complexity

### Why Supabase over self-hosted Postgres?
- **All-in-one**: DB + Auth + Realtime + Storage
- **Ops burden**: Zero database administration
- **Cost**: $25/month vs $50+ for RDS + other services
- **Trade-off**: Less control, potential vendor lock-in

### Why AWS Lambda over ECS everywhere?
- **Cost**: Pay per invocation, no idle costs
- **Scale**: Auto-scales instantly
- **Simplicity**: No container orchestration
- **Trade-off**: 60s timeout limit (use Fargate for longer)

### Why multiple LLM providers?
- **Reliability**: Fallback if Claude has outage
- **Cost optimization**: Different models for different tasks
- **Quality**: Claude for reasoning, GPT for specific tasks
- **Trade-off**: More integration code, more vendors

---

## References

- [Supabase Documentation](https://supabase.com/docs)
- [Pinecone Documentation](https://docs.pinecone.io)
- [Neo4j Aura Documentation](https://neo4j.com/docs/aura)
- [AWS Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [Anthropic Claude API](https://docs.anthropic.com)
- [Next.js Documentation](https://nextjs.org/docs)

---

**Last Updated**: 2025-11-08
**Version**: 1.0
**Maintained By**: Architecture Team
