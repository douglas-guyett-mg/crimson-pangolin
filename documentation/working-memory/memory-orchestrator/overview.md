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

## Consciousness Alignment (Design-Time Integration)

The Memory Orchestrator is designed to support consciousness requirements through design-time alignment, not runtime coupling. This means the orchestrator's architecture reflects consciousness principles, but consciousness artifacts are read at runtime to inform behavior.

### Charter & Mandates Alignment
- **IC-1 (Mission Alignment)**: MO links pending actions to active goals and mission context
- **IC-2 (Safety First)**: MO enforces safety policies during context assembly and tool tracking
- **IC-3 (Transparency)**: MO logs all decisions to Turn Trace with rationale and provenance
- **IC-4 (Test Integrity)**: MO behavior is fully specified with acceptance criteria and tests

The orchestrator provides interfaces for mandate checks (e.g., `check_mandate(action, layer)`) and tracks mandate evaluation results in Turn Trace for auditability.

### Modes & Capabilities Alignment
- MO interfaces allow filtering context by active mode
- MO enforces mode-specific tool filters and context budgets
- MO tracks which capabilities are active and logs them to Turn Trace
- MO supports CAP-WM (Working Memory Integration capability) by providing structured prompt fragments with metadata

### Prompt Assembly Pattern
The orchestrator assembles context in a prescribed order:
1. Consciousness fragments (charter, mandates, modes, capabilities)
2. Frontal Cortex context (active goals, pending actions, constraints)
3. Conversation History (recent dialogue + rolling summary)
4. Retrieved Episodic Memory (relevant events and observations)
5. Retrieved Semantic Memory (facts, skills, patterns)
6. Tool specifications

All injected fragments include provenance metadata (source path, version hash) for transparency and auditability.

## Core Responsibilities

### Context Assembly
The orchestrator assembles context by:

1. **Querying All Memory Types**: Calls remember() on all registered memory types
2. **Respecting Budgets**: Allocates token budget across memory types according to configured allocation strategy
3. **Ranking Results**: Combines results from multiple stores
4. **Formatting Output**: Formats context for injection into prompts

**Process**:
- Receives context request with token budget and prompt
- Allocates budget across memory types using configured allocation strategy
- Calls remember(prompt, metadata) on each memory type with allocated budget
- Combines results in priority order
- Returns formatted context

**Budget Allocation**:
Budget allocation across memory types is configurable. The orchestrator distributes the total token budget to each registered memory type according to the configured allocation strategy. Different allocation strategies may prioritize different memory types based on task requirements.

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
- **Memory Plugins**: Calls operations on registered types
- **Budget Manager**: Enforces budgets
- **Turn Trace**: Logs all operations
- **Frontal Cortex**: Reads goals, writes progress deltas
- **Skill Extractor**: Receives episodic exports for analysis

## Configuration

The Memory Orchestrator can be configured with:

- **budget_allocation_strategy**: Strategy for allocating token budget across memory types. Specifies how to distribute the total token budget among registered memory types. The strategy determines the proportion of budget allocated to each type.
- **default_filters**: Default filters for context assembly
- **enable_tool_tracking**: Whether to track tool invocations (default: true)
- **enable_skill_extraction**: Whether to enable skill extraction (default: true)

## Example Scenario

1. Turn starts, orchestrator initializes with configured budget_allocation_strategy
2. Agent receives user query
3. Orchestrator calls assemble_context(prompt, budget=2000)
   - Applies budget_allocation_strategy to distribute 2000 tokens across registered memory types
   - Calls remember(prompt) on each memory type with allocated budget
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

