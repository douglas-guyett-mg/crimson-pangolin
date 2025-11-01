# MemoryPlugin Interface

## Overview

The MemoryPlugin Interface is the contract that all memory type implementations must fulfill. It defines a small set of intent-based operations that the Memory Orchestrator can perform on any memory type, enabling a uniform, pluggable architecture where each memory type can implement operations completely differently according to its own semantics.

## Purpose

The interface serves to:

1. **Intent-Based Operations**: Define operations by what the orchestrator wants to accomplish, not by technical implementation
2. **Enable Polymorphism**: Allow the orchestrator to treat all memory types uniformly without knowing their internals
3. **Support Extensibility**: Enable new memory types to be added by implementing this interface
4. **Enforce Contracts**: Ensure all implementations provide required capabilities and return types
5. **Maximize Flexibility**: Each memory type interprets operations according to its own semantics and storage model

## Core Operations

All memory types must implement these five intent-based operations. Each type interprets these operations according to its own semantics and storage model.

### memorize(item, metadata)
Store a memory item in this memory type.

**Parameters**:
- `item`: MemoryItem to store (includes text, type, tags, metadata)
- `metadata`: Additional metadata (e.g., importance score, source, tags)

**Returns**:
- Confirmation of storage (success/failure)
- Item ID or reference (format depends on memory type)
- Any eviction information (if items were removed due to capacity limits)

**How Different Memory Types Implement This**:
- **Working Memory**: Adds to queue, scores importance, may evict low-importance items
- **Episodic Memory**: Appends to chronological log with timestamp
- **Semantic Memory**: Upserts by semantic key, creates new version if key exists
- **Conversation History**: Adds to rolling buffer, may evict oldest turns

**Purpose**: Store new memories in the system

---

### remember(prompt, metadata)
Find relevant memories based on a prompt or description.

**Parameters**:
- `prompt`: Natural language description of what to remember (e.g., "What did the user say about their job?")
- `metadata`: Optional metadata filters (e.g., agent_id, time range, tags)

**Returns**:
- List of MemoryItem objects relevant to the prompt
- Ranked by relevance (ranking strategy depends on memory type)
- Includes metadata and confidence scores

**How Different Memory Types Implement This**:
- **Working Memory**: Searches by importance/recency, returns high-importance items matching prompt
- **Episodic Memory**: Searches chronological log by time range and content similarity
- **Semantic Memory**: Performs hybrid semantic search (dense + sparse)
- **Conversation History**: Searches turn content by keywords and semantic similarity

**Purpose**: Retrieve relevant memories for context assembly and decision-making

---

### forget(forget_instructions, mode)
Remove or fade memories based on natural language instructions.

**Parameters**:
- `forget_instructions`: Natural language instructions describing what to forget (e.g., "memory id 5", "oldest item", "slightly fade the memory about being on a bike at 5 years old")
- `mode`: How to forget - "soft" (mark as deleted, keep in store) or "hard" (permanently remove)

**Returns**:
- Confirmation of forget operation
- Metadata about what was forgotten
- Any side effects (e.g., items that were faded or removed)

**How Different Memory Types Implement This**:
- **Working Memory**: Interprets "oldest" as least important/oldest, removes from queue
- **Episodic Memory**: Interprets "before timestamp X" as time-based deletion, supports soft/hard delete
- **Semantic Memory**: Interprets "key:X" as semantic key lookup, supports soft/hard delete with versioning
- **Conversation History**: Interprets "before turn N" as turn-based deletion, removes from buffer

**Supported Instructions** (each memory type documents which it supports):
- Common: "oldest", "least important", "before [time/reference]"
- Type-specific: Memory types may support additional instructions based on their semantics

**Purpose**: Remove obsolete, sensitive, or low-value memories

---

### summarize(scope, token_limit)
Generate a summary of memories within a given scope.

**Parameters**:
- `scope`: What to summarize (e.g., "all", "recent", "important", "by_tag:goal")
- `token_limit`: Maximum tokens for the summary

**Returns**:
- Summary text
- Actual token count used
- List of source items included in summary

**How Different Memory Types Implement This**:
- **Working Memory**: Summarizes high-importance items, preserves recent context
- **Episodic Memory**: Summarizes by time period, preserves key events
- **Semantic Memory**: Summarizes key facts and patterns, preserves important knowledge
- **Conversation History**: Summarizes conversation turns, preserves key exchanges

**Purpose**: Compress memories into concise form for context injection

---

### get_capacity_info()
Return information about this memory type's capacity and current usage.

**Parameters**: None

**Returns**:
- Current item count
- Current token usage
- Maximum item limit (if applicable)
- Maximum token limit (if applicable)
- Available space (if applicable)
- Eviction policy information

**Purpose**: Allow the orchestrator to understand capacity constraints and make decisions about memory management

---

## Common Responsibilities

All implementations must handle:

1. **Budgeting**: Respect token, event, and time budgets set by the orchestrator
2. **Redaction**: Apply redaction policies to sensitive data before returning
3. **Instruction Interpretation**: Document and handle the natural language instructions they support
4. **Serialization**: Support serialization/deserialization for persistence
5. **Error Handling**: Return clear errors for unsupported instructions or invalid operations
6. **Capacity Management**: Manage capacity according to their own semantics (queue, log, versioning, etc.)

## Design Principles

1. **Intent-Based Interface**: Operations are defined by what the orchestrator wants to accomplish, not by technical implementation
2. **Semantic Flexibility**: Each memory type interprets operations according to its own storage model and semantics
3. **Natural Language Instructions**: Operations like `forget()` use natural language to maximize flexibility
4. **Documented Semantics**: Each memory type clearly documents how it interprets each operation
5. **Async-Ready**: Operations should support both synchronous and asynchronous execution
6. **Deterministic**: Given the same input, operations should produce the same output
7. **Auditable**: All operations should be loggable for debugging and compliance

## Interaction with Other Components

- **Memory Orchestrator**: Calls these methods to assemble context and manage memory
- **Memory Type Registry**: All implementations register themselves and expose this interface
- **Budget Manager**: Enforces budgets during read/write operations
- **Turn Trace**: Logs all operations for debugging

## Implementation Notes

- **Semantic Documentation**: Each implementation must document how it interprets each operation
- **Supported Instructions**: Each implementation must document which natural language instructions it supports for `forget()`
- **Additional Methods**: Implementations may add additional methods beyond this interface for type-specific operations
- **Optimization**: Implementations may optimize operations based on their storage model
- **Configuration**: Implementations should support configuration at instantiation time
- **Error Handling**: Implementations should clearly document what happens when unsupported instructions are provided

## Example: How Three Memory Types Implement `remember()`

**Orchestrator calls**: `remember("What did the user say about their job?", metadata)`

**Working Memory interprets this as**:
- Search recent items by importance score
- Return high-importance items from last N turns that match the prompt
- Ranking: by importance score

**Episodic Memory interprets this as**:
- Search chronological log by time range and content
- Return items from relevant time periods that match the prompt
- Ranking: by time recency and content similarity

**Semantic Memory interprets this as**:
- Perform hybrid semantic search (dense + sparse)
- Return facts semantically similar to the prompt
- Ranking: by combined semantic similarity score

All three return the same type (list of MemoryItems), but the search strategy is completely different.

