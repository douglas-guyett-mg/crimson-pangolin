# Working Memory System

**Status:** v0.2 (Draft)
**Last Updated:** 2025-11-07
**Alignment:** Si Core Engineering Tenants (documentation-as-code, tests-first, modularity, technology-agnosticism)

## Overview

The Working Memory System is the central hub for managing all forms of memory in Si. It coordinates the storage, retrieval, and lifecycle of memories across multiple specialized memory types (episodic, semantic, working, conversation history, and others). The system is designed to be modular, extensible, and storage-aware, enabling Si to maintain rich context while respecting storage constraints.

**Note**: This system manages memory *storage*. For LLM call budgets and per-user cost tracking, see `documentation/llm-budget-system/`.

This specification is detailed and prescriptive enough for a coding agent to implement in any language and runtime, without vendor lock-in or technology-specific constraints.

## Purpose

The Working Memory System serves to:

1. **Coordinate Memory Operations**: Provide a unified interface for all memory-related operations across multiple memory types
2. **Assemble Context**: Query all memory types and combine results into coherent context for prompt injection
3. **Manage Storage**: Enforce storage policies and eviction strategies across all memory stores
4. **Track Tool Invocations**: Log tool calls and outcomes for learning and skill extraction
5. **Route Memory Items**: Determine which memory types should receive each item based on content and type
6. **Support Extensibility**: Enable new memory types to be added without modifying core orchestration logic
7. **Maintain Audit Trail**: Log all operations for debugging, compliance, and learning

## Scope

**In Scope**: Memory models, data schemas, read/write/eviction policies, retrieval and compaction algorithms, testing requirements, APIs, and behavioral specifications.

**Out of Scope**: Choosing specific vendors (DBs, vector engines, LLM models), product UI, or deployment details.

## Background and Inspiration

The Working Memory System is inspired by cognitive science research on multi-store memory architectures. Key concepts adopted:

- **Multi-store memory**: Working (short-term scratchpad for ongoing reasoning), episodic (chronological events), semantic (distilled facts/skills)
- **Iterative loop**: Each thought-action-observation step can read from memory and selectively write back distilled updates
- **Gating and policies**: Importance scoring, recency, and novelty control writes; compaction keeps memory within budget
- **Hybrid retrieval**: Dense + sparse search with re-ranking; context windows are dynamically budgeted
- **Summarization and reflection**: Compress past traces into abstractions and "skills," maintaining utility under tight token limits

## Terminology

- **MemoryItem**: The atomic unit stored in any memory store
- **Working Memory (WM)**: Bounded in-context scratchpad for the current turn/task
- **Episodic Memory (EM)**: Time-ordered log of observations, interactions, and events
- **Semantic Memory (SM)**: Distilled, generalizable knowledge, facts, and skills
- **Conversation History (CH)**: Recent dialogue with rolling summaries
- **Memory Orchestrator (MO)**: Central coordinator that routes reads/writes/evictions across stores
- **Memory Interface**: Pluggable memory type implementing a standard interface
- **Write Gating**: Algorithm that routes items to appropriate memory types based on content and importance
- **Context Assembly**: Process of querying all memory types and combining results for prompt injection
- **Eviction Policy**: Strategy for managing storage capacity in each memory store (e.g., FIFO, LRU, importance-based)

## Core Architecture

The Working Memory System consists of several interconnected components:

### Memory Types (Pluggable)
- **Episodic Memory Store**: Append-only log of experiences and events in chronological order
- **Semantic Memory Store**: Fact and pattern repository with versioning and hybrid search
- **Working Memory Store**: Short-term buffer of recent, high-importance items
- **Conversation History Plugin**: Rolling buffer of recent conversation turns
- Additional types can be registered via the Memory Type Registry

### Central Coordination
- **Memory Orchestrator**: Central coordinator that manages all memory operations
- **Memory Interface**: Contract that all memory types must implement
- **Memory Type Registry**: Registry of available memory types and their capabilities

### Supporting Systems
- **Memory Item Data Model**: Standardized format for all memory items
- **Hybrid Indexing**: Multi-index search across memory types
- **Re-ranking**: Combines results from multiple memory types into ranked lists
- **Multi-Resolution Summarizer**: Generates summaries at different levels of detail
- **Skill Extraction**: Analyzes episodic memory to extract reusable patterns
- **Turn Trace Integration**: Logs all memory operations for debugging and analysis
- **Frontal Cortex Integration**: Reads goals and writes progress deltas
- **End-to-End Testing**: Comprehensive test suite for the entire system

## Key Concepts

### Memory Item
A standardized unit of memory containing:
- **Content**: The actual memory (text, structured data, etc.)
- **Type**: Classification (observation, fact, skill, pattern, etc.)
- **Metadata**: Tags, source, importance, timestamps, etc.
- **Lifecycle**: Creation, access, modification, deletion timestamps

### Write Gating
The write gating algorithm determines which memory types should receive each item based on:
- Item type (observation, fact, skill, pattern, etc.)
- Item content and importance
- Current budget status
- Configured routing rules

### Context Assembly
The process of querying all memory types and combining results into coherent context:
1. Query each memory type with search parameters
2. Rank and combine results
3. Format for prompt injection
4. Return formatted context with token count

**Note**: The LLM call budget (max tokens in prompt) is enforced by the LLM Budget System, not by memory stores. See `documentation/llm-budget-system/overview.md`.

## Operations

### assemble_context(prompt, budget, filters)
Assembles context from all memory types based on a prompt.

**Returns**: Formatted context string, token count, source breakdown

### track_tool_invocation(tool_name, parameters, result, error)
Tracks a tool invocation for learning and skill extraction.

**Returns**: Confirmation, item ID in episodic memory

### memorize(item, metadata)
Memorizes an item to appropriate memory types based on write gating.

**Returns**: Confirmation, routing information, eviction information

### commit()
Finalizes memory state for the turn and exports updates.

**Returns**: Confirmation, summary of writes, exports for episodic/semantic memory and frontal cortex

### get_budget_status()
Returns current budget status.

**Returns**: Token usage, remaining budget, event count, time budget

## Design Principles

1. **Modular Architecture**: Each memory type is independent and can be swapped or extended
2. **Storage-First**: All operations respect storage policies and eviction strategies
3. **Pluggable Memory Types**: New memory types can be added by implementing the Memory Interface
4. **Intent-Based Interface**: Memory types interpret operations according to their own semantics
5. **Transparent Routing**: Items are routed based on type and content, not explicit configuration
6. **Auditable**: All operations are logged for debugging and compliance
7. **Deterministic**: Same inputs produce same outputs

## Interaction with Other Systems

- **Consciousness**: Provides templates and mandates that influence memory routing and context assembly
- **Frontal Cortex**: Reads goals, writes progress deltas, marks important memories
- **Turn Trace**: Logs all memory operations for debugging and analysis
- **Skill Extraction**: Analyzes episodic memory to extract reusable patterns
- **First-Mile Communications**: Writes canonical transcripts to episodic memory

## Configuration

The Working Memory System can be configured with:

- **default_filters**: Default filters for context assembly
- **enable_tool_tracking**: Whether to track tool invocations (default: true)
- **enable_skill_extraction**: Whether to enable skill extraction (default: true)
- **memory_types**: List of registered memory types and their configurations

## Data Model

### MemoryItem (Common Fields)
All memory items share a standardized structure:
- **id**: Unique identifier (ULID)
- **type**: Classification (user_message, tool_result, thought, action, plan, skill, summary, fact, metric, observation)
- **text**: The actual content
- **tokens_estimate**: Estimated token count
- **timestamp**: ISO-8601 creation time
- **agent_id**: Which agent created this item
- **task_id**: Associated task
- **episode_id**: Associated episode (optional)
- **importance**: Float 0–1, user-assigned or computed
- **novelty**: Float 0–1, computed vs existing memory
- **recency**: Float 0–1, computed based on age
- **source**: Origin (user, tool, system, model)
- **tags**: String array for categorization
- **metadata**: Arbitrary key-value pairs
- **retention**: Policy ID for lifecycle management
- **ttl**: Time-to-live duration (optional)
- **embedding**: Dense vector representation (optional)
- **sparse_terms**: Keyword terms for sparse search (optional)
- **redaction_status**: none, partial, or full
- **checksum**: For integrity verification

### Store-Specific Fields
- **WM**: position (int), step_index (int)
- **EM**: parent_id (string), chunk_index (int)
- **SM**: version (int), lineage_ids (string[]), validity (time range)



## Read Path (Retrieval) Algorithm

Given a ReadContext (query_text, task_id, agent_id, top_k, budget_tokens, include: [WM|EM|SM]):

1. **Build composite query**: Current user prompt + agent goal + last plan + recent WM
2. **Candidate pools**:
   - WM: Recent N items (cheap, recency-biased)
   - EM: Hybrid search (dense + BM25/keywords) with time decay; retrieve k1 candidates
   - SM: Hybrid search + metadata filters (task/agent); retrieve k2 candidates
3. **Re-rank all candidates**:
   - **Scoring formula**: `score = α×similarity + β×recency + γ×importance − δ×duplication`
   - **Default coefficients** (tunable via System Change Proposals):
     - α = 0.40 (similarity weight: how relevant is this to the query?)
     - β = 0.25 (recency weight: how recent is this?)
     - γ = 0.25 (importance weight: how important is this?)
     - δ = 0.10 (duplication penalty: how much does this duplicate existing WM?)
   - **Rationale**: Similarity is weighted highest because relevance matters most; recency and importance are balanced; duplication is penalized to avoid redundancy
4. **Novelty filter**: Remove near-duplicates to WM
5. **Budgeting**: Greedily pack highest-scoring items until budget_tokens exhausted
6. **Return**: MemoryBundle with selected items and metadata

**Coefficient Tuning:**
- Initial values are defaults based on cognitive science research
- Over time, System Change Proposals can suggest coefficient adjustments based on:
  - Which items were actually useful in past turns
  - Which items were included but unused
  - Patterns in task success/failure
- Changes are tracked in Turn Trace for auditability and learning

**Test Requirements**:
- Deterministic packing under fixed seeds
- Hybrid search returns ≥90% of ground-truth relevant items in synthetic tasks
- Budgeting respects token limits and ordering stability
- Scoring formula produces consistent rankings across multiple runs
- Coefficient changes measurably improve context quality (measured via downstream task success)

## Write Path (Gating) Algorithm

On WriteEvent (item, candidate_store: WM|EM|SM|auto):

1. **Redact**: Apply PII guard before storage
2. **Score**: Compute importance (self-eval or heuristic), novelty vs existing memory, and size
3. **Route**:
   - Transient reasoning for current step → WM
   - Actionable observation or outcome → EM
   - Distilled fact/skill/summary with cross-episode utility → SM
4. **Eviction**: If store capacity exceeded, apply store-specific eviction policy
5. **Index update**: Upsert dense + sparse indices

**Test Requirements**:
- Write acceptance rate predictable under thresholds
- No unredacted PII reaches persistent stores
- Routing sends skills to SM with >95% precision in unit fixtures

## Compaction and Eviction

**Policies**:
- **WM**: FIFO with importance guard; periodic summarization of tail to one summary item
- **EM**: Episode summarization after M steps or N tokens; keep summary + pointers to raw chunks
- **SM**: Versioned summaries; merge similar skills; periodic de-duplication via clustering

**Strategies**:
- Time-decay weight: w(t) = exp(-λΔt)
- Reservoir sampling for long-lived logs with high-importance bias
- Evict to target budget (tokens or items) with minimal utility loss (greedy by utility density)

**Test Requirements**:
- Compaction never increases total tokens beyond tolerance (+5%)
- Summaries preserve QA answerability in golden tasks within 2% drop

## Episodic Memory Compaction: Threads and Episodes

For detailed specification of how episodic memory handles conversation threads, episode boundaries, and cross-thread linking, see:
- **`episodic-memory-compaction.md`** - Comprehensive specification of thread boundary detection (explicit/implicit channels), semantic linking, cross-thread episodes, and aggressive context retrieval
- **`episodic-memory-compaction-test-conditions.md`** - Test scenarios for all aspects of thread and episode management

## Example Scenario

1. Turn starts, orchestrator initializes with configured budget
2. Agent receives user query
3. Orchestrator calls assemble_context(prompt, budget=2000)
   - Allocates budget across registered memory types
   - Queries each memory type with allocated budget
   - Combines results into context
4. Agent processes query and calls a tool
5. Orchestrator calls track_tool_invocation() with tool name, params, result
6. Episodic memory receives the tool trace
7. Agent generates response
8. Orchestrator calls memorize() with response item
9. Write gating algorithm routes to appropriate memory types
10. Turn ends, orchestrator calls commit()
11. Episodic and semantic memory exports are generated
12. Frontal cortex updates are exported
13. All operations are logged to turn trace

