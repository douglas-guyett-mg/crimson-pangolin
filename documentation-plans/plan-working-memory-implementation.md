# Working Memory Implementation Plan

**Status**: Draft  
**Last Updated**: 2025-10-30  
**Based on**: arxiv.org/pdf/2309.02427 (Multi-Store Memory Architecture)  
**Key Principle**: Pluggable/extensible memory types (registry-driven, not hard-coded)

---

## Overview

This plan outlines the implementation of a working memory system that supports multiple memory types (Conversation History, Episodic, Semantic, and custom plugins) through a registry-driven architecture. The system will follow the paper's multi-store approach while maintaining technology-agnosticism and modularity.

---

## Implementation Phases

### Phase 1: Core Infrastructure (Foundation)
- [ ] **1.1 Define MemoryType Registry**
  - Create registry interface for registering/discovering memory types
  - Implement factory pattern for instantiating memory types
  - Support dynamic registration without code changes
  - Test: Registry can add/remove types; factory creates correct instances

- [ ] **1.2 Implement MemoryPlugin Interface**
  - Define abstract base with: `capabilities()`, `read()`, `write()`, `search()`, `summarize()`
  - Create base implementation with common logic (budgeting, redaction, indexing)
  - Test: All methods callable; return types match spec

- [ ] **1.3 Build Memory Item Data Model**
  - Implement core MemoryItem with: id, type, text, tokens_estimate, timestamp, agent_id, tags, metadata
  - Add store-specific fields (WM: position/step_index; EM: parent_id/chunk_index; SM: version/lineage)
  - Support serialization/deserialization
  - Test: Items serialize/deserialize correctly; all fields preserved

### Phase 2: Core Memory Stores (Pluggable Implementations)
- [ ] **2.1 Implement Working Memory Store**
  - Bounded queue (max_items=64, max_tokens=4k)
  - FIFO eviction with importance guard
  - TTL by step-count and wall time
  - Test: Respects size/token limits; eviction preserves high-importance items

- [ ] **2.2 Implement Episodic Memory Store**
  - Append-only log with chunking for large entries
  - Chronological range queries and episode grouping
  - Soft-delete support
  - Test: Queries return correct chronological order; chunking preserves semantics

- [ ] **2.3 Implement Semantic Memory Store**
  - Upsert/soft-delete with versioning
  - LRU+importance retention
  - Hybrid search support (dense + sparse)
  - Test: Upserts work correctly; LRU eviction respects importance

- [ ] **2.4 Implement Conversation History Plugin**
  - Last N utterances + rolling summary
  - Swappable backend (in-memory ring buffer or durable store)
  - Test: Retrieves correct utterances; summary updates correctly

### Phase 3: Orchestrator & Routing
- [ ] **3.1 Implement Memory Orchestrator**
  - `assemble_context()`: queries all registered memory types, respects budgets
  - `track_tool_invocation()`: logs tool calls with outcomes
  - `commit()`: exports EM/SM/FC writes
  - Test: Context assembly respects token budgets; tool tracking is complete

- [ ] **3.2 Implement Budget Enforcement**
  - Per-component token/event/time budgets
  - Sampling and redaction policies
  - Budget snapshots and status reporting
  - Test: Budgets enforced; snapshots accurate

- [ ] **3.3 Implement Write Gating Algorithm**
  - Redaction → Scoring → Routing → Eviction cascade
  - Importance/novelty/recency scoring
  - Route to WM/EM/SM based on item type
  - Test: Items routed correctly; gating respects policies

### Phase 4: Indexing & Retrieval
- [ ] **4.1 Implement Hybrid Indexing**
  - Dense embeddings (vector search)
  - Sparse terms (keyword search)
  - Metadata filtering
  - Test: Hybrid search returns 90%+ ground-truth relevant items

- [ ] **4.2 Implement Re-ranking**
  - Combine dense + sparse scores
  - Apply recency/importance weights
  - Test: Re-ranked results improve relevance

### Phase 5: Summarization & Compression
- [ ] **5.1 Implement Multi-Resolution Summarizer**
  - Micro (1–3 sentences for WM injection)
  - Meso (paragraph-length for episodes)
  - Macro (distilled patterns for skills/goals)
  - Test: Summaries retain answerability; token targets met

- [ ] **5.2 Implement Skill Extraction**
  - Extract reusable procedures from episodic traces
  - Link to source events
  - Test: Extracted skills are generalizable

### Phase 6: Integration & Testing
- [ ] **6.1 Integrate with Frontal Cortex**
  - Read FC goals/pending actions at turn start
  - Write goal progress deltas at commit
  - Test: FC reads/writes work correctly

- [ ] **6.2 Integrate with Turn Trace**
  - Log all orchestrator decisions to Turn Trace
  - Include mandate checks and budget snapshots
  - Test: Turn Trace captures all events

- [ ] **6.3 End-to-End Testing**
  - Multi-turn scenarios with tool invocations
  - Budget enforcement under load
  - Memory type addition/removal without breaking
  - Test: System handles realistic workflows

---

## Key Design Decisions

1. **Registry-Driven**: Memory types registered at startup; new types added via plugin interface
2. **Pluggable Backends**: Each memory type can use different storage (in-memory, database, vector DB)
3. **Unified Interface**: All memory types implement MemoryPlugin; orchestrator treats them uniformly
4. **Budget-First**: All operations respect token/event/time budgets
5. **Deterministic**: Fixed seeds produce identical context assembly and summaries

---

## Success Criteria

- [ ] All memory types can be added/removed without code changes
- [ ] Context assembly respects all budgets
- [ ] Hybrid search achieves 90%+ recall on synthetic tasks
- [ ] Summarization preserves answerability under token limits
- [ ] Tool tracking is complete and auditable
- [ ] System handles 1000+ memory items without degradation

