# Daemon Interface

**Status:** Specification v1.0
**Date:** 2025-11-01
**Purpose:** Define the unified interface that all daemons implement

---

## Overview

A **Daemon** is a self-contained agent with a clear purpose, identity, and set of capabilities. All daemons implement a common interface that allows other components to:
- Understand what the daemon does
- Understand how the daemon operates
- Interact with the daemon asynchronously

Examples of daemons:
- **Memory Daemons**: Episodic Memory, Semantic Memory, Working Memory
- **Consciousness Daemon**: Provides SI's self-concept, mandates, capabilities, and governance rules
- **Processing Daemons**: Skill Extraction, Pattern Recognition, Memory Consolidation
- **Coordination Daemons**: Router, Clarifier, Executor, Error Handler, Evaluator, Reflector, Responder (from turn architecture)

**Special Note on Executor**: The Executor daemon is the turn orchestrator. It queries other daemons to decide which lifecycle states to execute, then executes only the necessary states. Executor is a daemon like any other, but it has the special responsibility of orchestrating the entire turn.

**Special Note on Consciousness**: The Consciousness daemon provides SI's self-concept through the standard Daemon Interface. Other daemons query it to retrieve mandates, capabilities, modes, and governance rules. See `documentation/consciousness/daemon-specification.md` for details.

---

## Core Daemon Interface

All daemons must implement these three operations:

### query(natural_language_query, metadata?, data?) → Future<results>

Sends a query to the daemon. The daemon handles it however it wants internally.

**Parameters:**
- `natural_language_query` (string): What you want to know or do
  - Examples: "Tell me about the time I learned to ride the bicycle"
  - Examples: "What mandates apply to financial security?"
  - Examples: "Store this observation about user preferences"

- `metadata` (object, optional): Structured hints to help the daemon
  - Examples: {date_range: "2020-2025", tags: ["learning", "skills"]}
  - Examples: {importance: "high", category: "financial"}
  - The daemon can use or ignore metadata as it sees fit

- `data` (any, optional): Information to store or process
  - Can be any format: JSON, text, structured objects, etc.
  - The daemon's instructions tell you what format it expects
  - Or the daemon can use an LLM to parse flexible formats

**Returns** (via future):
- Results in whatever format makes sense for the query
- Can be structured data, natural language, or anything else
- Daemon decides the format based on the query

**Example:**
```
# Query for information
future = episodic_memory.query(
  "Tell me about the time I learned to ride the bicycle",
  metadata={date_range: "2020-2025", tags: ["learning"]}
)
result = await future
# Returns: {event_id: "...", summary: "...", timestamp: "...", ...}

# Query to store information
future = episodic_memory.query(
  "Store this observation",
  data={observation: "User prefers morning meetings", confidence: 0.9}
)
result = await future
# Returns: {stored: true, id: "..."}
```

### get_purpose() → string

Returns a natural language description of what the daemon does.

**Example responses:**
- "Episodic Memory Daemon: Stores and retrieves turn events, observations, and interactions in chronological order"
- "Router Daemon: Analyzes incoming input and decides urgency level and which daemons to activate"
- "Consciousness Daemon: Stores and retrieves Si's values, mandates, capabilities, and governance rules"

### get_instructions() → string

Returns a natural language description of how the daemon operates, including:
- What types of queries it handles
- What format it expects for data (if applicable)
- How it processes queries internally
- Any constraints or limitations

**Example responses:**
- "Episodic Memory Daemon: Accepts natural language queries about past events. Internally uses vector search on event summaries. Expects data in JSON format with 'observation', 'timestamp', 'tags' fields. Can also parse flexible formats with LLM assistance."
- "Router Daemon: Analyzes urgency by querying consciousness for mandates and episodic memory for patterns. Returns urgency level (low/medium/high) and list of daemons to activate. Queries are processed synchronously but return async futures."

---

## Async Operations

All daemon operations are asynchronous and return futures/promises:

```
future = daemon.query(natural_language_query, metadata?, data?)
# Caller can do other work while waiting
result = await future
```

This enables parallelism: request information from one daemon, do other work, use result when it arrives.

---

## How Daemons Know What to Do

Each daemon's `get_instructions()` tells callers:
1. What types of queries it handles
2. What format it expects for data
3. How it processes queries internally

**Example 1: Flexible parsing**
- Instructions: "I accept any format. I use an LLM to parse and understand what you're asking."
- Caller can pass JSON, text, structured objects, etc.

**Example 2: Strict format**
- Instructions: "I only accept JSON with fields: {observation, timestamp, tags}. I will reject other formats."
- Caller must follow the format

**Example 3: Implementation flexibility**
- Instructions: "I handle queries about past events. Currently I use vector search, but I may switch to graph RAG or other methods internally."
- Caller doesn't care how it's implemented, just passes the query

---

## Daemon Lifecycle

### Key Principle: Fresh Instances Per Turn

**Daemons are created fresh for each turn and destroyed at turn end.** No persistent state lives in the daemon instance itself. However, daemons access persistent storage (episodic memory store, semantic memory store, consciousness store, etc.) that is shared across turns.

**Example:**
- Turn 1: Working Memory daemon instance created, queries episodic memory store, destroyed at turn end
- Turn 2: New Working Memory daemon instance created, queries same episodic memory store, destroyed at turn end
- Both instances access the same persistent episodic memory data

### Creation Phase

**Triggered by:** Executor at turn start

**Process:**
1. Executor creates daemon instance with configuration
2. Executor registers daemon instance with Daemon Registry
3. Daemon initializes (loads instructions, sets up state)
4. Daemon becomes available for queries within its turn

**State:** Daemon instance has no persistent state. It may have in-memory state for this turn only.

### Operation Phase

**During turn execution:**
1. Other daemons query this daemon via `query()`
2. Daemon processes queries and returns results
3. Daemon may query other daemons to fulfill requests (subject to turn scoping rules)
4. Daemon may access persistent storage (episodic memory, semantic memory, consciousness, etc.)

**Parallelism:** Multiple daemons can query the same daemon in parallel. The daemon handles concurrent requests.

### Shutdown Phase

**Triggered by:** Executor at turn end (after all daemons complete)

**Process:**
1. Executor deregisters daemon instance from Daemon Registry
2. Daemon cleans up resources (closes connections, releases memory)
3. Daemon is no longer available for queries

**State Persistence:** If a daemon is interrupted before completion:
- Daemon should save its in-flight state to persistent storage
- Next turn can resume from saved state
- Mechanism is implementation detail (technology decision)

**What to Save (by daemon type):**

**Memory Daemons** (Episodic, Semantic, Working Memory):
- Current query being processed
- Search position/offset (for paginated results)
- Partial results collected so far
- Ranking/scoring state
- Budget consumed so far
- Timestamp of interruption

**Executor Daemon**:
- Current step in execution plan
- Results from completed steps
- In-flight tool calls (which tools are running)
- Error state (if any)
- Budget consumed so far
- Timestamp of interruption

**Frontal Cortex Daemon**:
- Current planning state
- Partial plan generated so far
- Goals being considered
- Constraints being evaluated
- Timestamp of interruption

**Consciousness Daemon**:
- Current query being processed
- Partial results collected
- Mode being queried
- Timestamp of interruption

**Responder Daemon**:
- Partial message being generated
- Context used so far
- Formatting state
- Timestamp of interruption

**General Pattern:**
All daemons should save:
1. **Query/Request**: What was being asked
2. **Progress**: How far through the operation
3. **Partial Results**: What has been computed so far
4. **Execution State**: Where in the algorithm we are
5. **Timestamp**: When the interruption occurred
6. **Metadata**: Turn ID, daemon ID, attempt number

### Turn Synchronization

**Important:** A turn does not end until all daemons complete.

- If a daemon is still processing when turn end is requested, the turn waits
- If a daemon is interrupted (e.g., system shutdown), its state is saved
- Next turn can resume from saved state

**Note:** See `documentation/daemon-registry/overview.md` for details on daemon registration, discovery, and turn scoping rules.

---

## Design Principles

1. **Self-Contained**: Each daemon has clear purpose and operates independently
2. **Introspectable**: Every daemon can explain its purpose and instructions
3. **Asynchronous**: All operations are non-blocking, enabling parallelism
4. **Composable**: Daemons can call other daemons without blocking
5. **Replaceable**: Daemons can be swapped via interface without changing callers
6. **Flexible**: Daemons handle queries however they want internally
7. **Auditable**: All operations can be logged for debugging and compliance
8. **Technology-Agnostic**: Interface doesn't prescribe implementation details

---

## Interaction with Other Components

- **Daemon Registry**: Executor registers daemon instances and manages their lifecycle. Registry enforces turn scoping rules and executor hierarchy for cross-turn communication. See `documentation/daemon-registry/overview.md`.
- **Turn Trace**: All daemon operations are logged for debugging, auditing, and monitoring
- **Working Memory**: Daemons may be queried by working memory for context
- **Other Daemons**: Daemons query each other to gather information and coordinate (subject to turn scoping rules)

---

## Example: Router Daemon Querying Other Daemons

```
Router receives user input: "I got a suspicious email from my bank"

1. Router queries Consciousness Daemon:
   future_consciousness = consciousness.query(
     "What mandates apply to financial security?"
   )

2. Router queries Episodic Memory Daemon:
   future_episodic = episodic_memory.query(
     "What patterns have we learned about suspicious financial emails?",
     metadata={tags: ["financial", "security"]}
   )

3. Router queries Working Memory Daemon:
   future_working = working_memory.query(
     "What is the current conversation context?"
   )

4. While waiting for results, Router can do other work
   # (In practice, Router would activate other daemons in parallel)

5. Results arrive:
   consciousness_result = await future_consciousness
   episodic_result = await future_episodic
   working_result = await future_working

6. Router makes decision: HIGH URGENCY
   - Logs decision to Turn Trace
   - Activates Executor daemon to handle immediately
   - Activates Reflector daemon to learn from outcome
```

All queries are async, so Router can parallelize them.

---

## Extending the Interface

New daemon types can be added without changing the core interface. Each daemon type may have additional methods specific to its purpose, but all must implement:
- `query(natural_language_query, metadata?, data?) → Future<results>`
- `get_purpose() → string`
- `get_instructions() → string`

