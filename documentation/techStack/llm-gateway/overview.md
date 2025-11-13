Title: LLM Gateway (Multi-Provider) — Overview
Status: Draft v1
Scope: Provider-agnostic gateway with concrete adapters for AWS Bedrock, OpenAI, Google Gemini, Anthropic Claude. Streaming-first; embeddings; budget enforcement; routing; safety; telemetry.

Purpose
- Provide a single interface for all LLM interactions (chat/completions, streaming, embeddings) used by Router, Executor, Evaluator, and Working Memory.
- Minimize provider lock-in and cost via policy-driven routing and fallbacks.
- Enforce token/latency budgets and safety policies before and during calls.

Responsibilities
- Normalize request/response semantics across providers (messages, tools, function calling).
- Streaming delivery with minimal latency to SSE/Web clients.
- Embedding generation with consistent dimensionality and metadata.
- Budget enforcement (global + daemon-level) and model routing policies.
- Provider-specific retry/backoff and error classification; circuit breaking.
- Security (secrets management, PII redaction hooks) and audit via Turn Trace events.
- Telemetry via OpenTelemetry and business events for cost/usage tracking.

Provider Matrix (initial)
- Bedrock: Claude 3 family, Llama, Mistral; embeddings. Regional routing via IAM role.
- OpenAI: gpt-4o/mini, text-embedding-3, responses API for streaming.
- Google Gemini: 1.5 Flash/Pro; multimodal; embeddings.
- Anthropic (direct): Claude 3 family; streaming.

Interfaces (conceptual; natural language)
- Chat/Completions: input messages, optional tool schema, temperature, max tokens, safety options, budget.
- Streaming: incremental token deltas with heartbeats, usage counters, finalization event.
- Embeddings: list of texts, model, batching policy; returns vectors and dimensions.
- Routing: policy inputs (latency target, budget tier, modality, org/user limits) → provider+model selection.
- Telemetry: emit start/partial/finish/error events with model, tokens, cost estimate, latency.

Routing Policy (defaults)
- Planning/long context: Claude Sonnet (Bedrock preferred) → fallback OpenAI GPT‑4o → Gemini Pro.
- Quick interactions: Claude Haiku (Bedrock) → GPT‑4o‑mini → Gemini Flash.
- Code/JSON-heavy: GPT‑4o‑mini primary; fallback to Claude Haiku.
- Embeddings: lowest-cost acceptable with target dimension set; prefer Bedrock where available, else OpenAI small.

Budgets and Safety
- Budgets: preflight token estimate; refuse or auto‑shrink prompts if exceeding daemon budget; log decision.
- Safety: provider-native safety controls plus project blocklists; redact PII in Working Memory write-path.

Operational Considerations
- Secrets in AWS Secrets Manager; no plaintext env keys in code.
- Rate limiting at org/user/provider level with DynamoDB token buckets.
- Timeouts aligned to First‑Mile SLOs and Turn budget (<50s end-to-end).
- Cost visibility: per-call and per-turn cost in Turn Trace.

Dependencies
- Turn Trace, LLM Budget System, Working Memory (for RAG), First‑Mile streaming layer.

Risks and Mitigations
- Provider outages → multi-provider fallback and circuit breakers.
- Cost overruns → strict budgets and default to cheaper models.
- Latency variance → adaptive routing and prewarming (where applicable).

