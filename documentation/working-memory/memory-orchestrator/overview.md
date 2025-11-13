# Memory Orchestrator

**Status**: Specification v0.2
**Last Updated**: 2025-10-31
**Mental Model**: Consciousness-Aware Mini-Agent/Daemon

## Overview

The Memory Orchestrator is the central coordinator of the working memory system. It manages interactions between the agent and all registered memory types, assembles context from multiple memory stores, tracks tool invocations, and coordinates writes to episodic and semantic memory.

The Memory Orchestrator is designed as a mini-agent (daemon) that reads consciousness artifacts (core_identity, mandates, modes, capabilities) at turn start, assembles a MemoryBundle by querying Frontal Cortex, Episodic Memory, Semantic Memory, and Conversation History, enforces budgets and policies, tracks tool invocations, and exports commit artifacts.

## Purpose

The Memory Orchestrator serves to:

1. **Assemble Context**: Query all memory types and combine results into a coherent context
2. **Track Tool Invocations**: Log tool calls and their outcomes for learning
3. **Coordinate Writes**: Route memory items to appropriate stores
4. **Enforce Budgets**: Ensure all operations respect token and event budgets
5. **Manage Lifecycle**: Initialize, update, and finalize memory state across turns
6. **Align with Consciousness**: Read consciousness mandates and inject relevant fragments into context
7. **Support Transparency**: Log all decisions to Turn Trace with rationale and provenance

## Consciousness Integration (Daemon Query Pattern)

The Memory Orchestrator queries the consciousness daemon to retrieve mandates, capabilities, and modes at runtime.

### Consciousness Daemon Queries
- **Query 1**: `consciousness.query("What is my core identity and charter?", {})` → Returns core identity mandate
- **Query 2**: `consciousness.query("What mandates apply to my current mode?", {current_mode: active_mode})` → Returns applicable mandates
- **Query 3**: `consciousness.query("What capabilities do I have?", {current_mode: active_mode})` → Returns active capabilities
- **Query 4**: `consciousness.query("What governance rules apply?", {})` → Returns governance rules

All queries are asynchronous (return Futures), enabling parallel retrieval.

### Prompt Assembly Pattern
The orchestrator assembles context in a prescribed order:
1. Consciousness items (charter, mandates, modes, capabilities) - retrieved via daemon queries
2. Frontal Cortex context (active goals, pending actions, constraints)
3. Conversation History (recent dialogue + rolling summary)
4. Retrieved Episodic Memory (relevant events and observations)
5. Retrieved Semantic Memory (facts, skills, patterns)
6. Tool specifications

All injected fragments include provenance metadata (source, version, rationale) for transparency and auditability.

## Core Responsibilities

### Context Assembly Algorithm

The orchestrator assembles context by querying memory types in a fixed order (v1), with each query using results from previous queries to inform what to retrieve.

#### Query Order (v1 - Fixed)

```
1. Consciousness daemon
   ↓ (get mandates, capabilities, modes, governance rules)
2. Conversation History
   ↓ (get recent dialogue, using consciousness context)
3. Working Memory Store
   ↓ (get recent items, using consciousness + conversation context)
4. Frontal Cortex daemon
   ↓ (get active goals, using consciousness + conversation + WM context)
5. Episodic Memory Store
   ↓ (get past events, using consciousness + conversation + WM + FC context)
6. Semantic Memory Store
   ↓ (get facts/skills, using all previous context)
```

#### Process

1. **Query Consciousness daemon**
   - Call: `consciousness.query("What mandates/capabilities apply?", metadata)`
   - Returns: `{items: [{id, content, rationale}], effective_at}`
   - Each memory store receives this context for its queries

2. **Query Conversation History**
   - Call: `conversation_history.remember(prompt, metadata)`
   - Metadata includes: consciousness items from step 1
   - Returns: formatted prompt section with recent dialogue

3. **Query Working Memory Store**
   - Call: `working_memory.remember(prompt, metadata)`
   - Metadata includes: consciousness items + conversation history from steps 1-2
   - Returns: formatted prompt section with recent items

4. **Query Frontal Cortex daemon**
   - Call: `frontal_cortex.query("What are my active goals?", metadata)`
   - Metadata includes: consciousness + conversation + WM context
   - Returns: formatted prompt section with goals and pending actions

5. **Query Episodic Memory Store**
   - Call: `episodic_memory.remember(prompt, metadata)`
   - Metadata includes: all previous context
   - Returns: formatted prompt section with relevant past events

6. **Query Semantic Memory Store**
   - Call: `semantic_memory.remember(prompt, metadata)`
   - Metadata includes: all previous context
   - Returns: formatted prompt section with facts and skills

#### Result Formatting

Each memory store returns a **formatted prompt section** (not raw data):

```
## [Memory Type Name]
[Formatted content ready for prompt injection]
```

The Memory Orchestrator concatenates all sections in order:

```
## Consciousness
[consciousness items]

## Conversation History
[recent dialogue]

## Working Memory
[recent items]

## Frontal Cortex
[active goals]

## Episodic Memory
[relevant past events]

## Semantic Memory
[facts and skills]

## Tool Specifications
[available tools]
```

#### Future Enhancement (v2 - Dynamic Query Order)

Eventually, the query order can be learned based on patterns:
- Working Memory learns which query order works best for different types of queries
- Learns which memory stores are most useful for different contexts
- Adapts the order dynamically based on patterns
- All context logged to Turn Trace for learning

#### Token Budget Enforcement

**Note**: Turn-level token budgets (if any) are enforced at the turn level, not at the memory store level. Memory stores do not have hard token limits. Each memory store is responsible for formatting its section to fit within reasonable bounds.

### Tool Invocation Tracking
The orchestrator tracks tool invocations by:

1. **Recording Tool Calls**: Logs when tools are invoked
2. **Recording Outcomes**: Logs tool results and errors
3. **Storing in Episodic Memory**: Writes complete trace to episodic store
4. **Extracting Patterns**: Identifies reusable patterns for skill extraction

**Information Tracked**:
- Tool name and parameters
- Execution time
- Result/output
- Errors or exceptions
- Relevance to current task

### Memorization Coordination
The orchestrator coordinates memorization by:

1. **Receiving Memory Items**: Accepts items to be memorized
2. **Applying Gating Algorithm**: Determines which stores should receive the item
3. **Enforcing Budgets**: Checks budget constraints
4. **Routing to Stores**: Calls memorize() on appropriate memory types
5. **Handling Evictions**: Processes eviction information

### Commit Operation
The orchestrator commits memory state by:

1. **Finalizing Writes**: Ensures all pending writes are persisted
2. **Exporting Episodic Memory**: Exports episodic memory writes
3. **Exporting Semantic Memory**: Exports semantic memory writes
4. **Exporting Frontal Cortex Updates**: Exports goal progress deltas
5. **Logging to Turn Trace**: Records all operations

## Operations

### assemble_context(prompt, budget, filters)
Assembles context from all memory types based on a prompt.

**Parameters**:
- `prompt`: The prompt/query to search memory for
- `budget`: Total token budget for context
- `filters`: Optional filters (agent_id, tags, etc.)

**Returns**:
- Formatted context string
- Actual token count
- Source breakdown (which memory types contributed)

**Process**:
- Calls remember(prompt, metadata) on each memory type
- Combines results in priority order
- Formats for injection into prompts

### track_tool_invocation(tool_name, parameters, result, error)
Tracks a tool invocation.

**Parameters**:
- `tool_name`: Name of the tool
- `parameters`: Tool parameters
- `result`: Tool result/output
- `error`: Error message (if any)

**Returns**:
- Confirmation of tracking
- Item ID in episodic memory

### memorize(item, metadata)
Memorizes a memory item to appropriate stores.

**Parameters**:
- `item`: MemoryItem to memorize
- `metadata`: Additional metadata

**Returns**:
- Confirmation of memorization
- Routing information (which stores received the item)
- Eviction information (if any)

### commit()
Finalizes memory state for the turn.

**Returns**:
- Confirmation of commit
- Summary of writes
- Episodic memory exports
- Semantic memory exports
- Frontal cortex updates

### get_budget_status()
Returns current budget status.

**Returns**:
- Current token usage
- Token budget remaining
- Event count
- Event budget remaining
- Time budget remaining

## Design Principles

1. **Centralized Coordination**: All memory operations go through orchestrator
2. **Budget-First**: All operations respect budgets
3. **Transparent Routing**: Items are routed based on type and content
4. **Auditable**: All operations are logged
5. **Deterministic**: Same inputs produce same outputs

## Interaction with Other Components

- **Memory Type Registry**: Discovers available memory types
- **Memory Interfaces**: Calls operations on registered types
- **Budget Manager**: Enforces budgets
- **Turn Trace**: Logs all operations
- **Frontal Cortex**: Reads goals, writes progress deltas
- **Skill Extractor**: Receives episodic exports for analysis

## Configuration

The Memory Orchestrator can be configured with:

- **default_filters**: Default filters for context assembly
- **enable_tool_tracking**: Whether to track tool invocations (default: true)
- **enable_skill_extraction**: Whether to enable skill extraction (default: true)

## Example Scenario

1. Turn starts, orchestrator initializes
2. Agent receives user query
3. Orchestrator calls assemble_context(prompt)
   - Calls remember(prompt) on each registered memory type
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

