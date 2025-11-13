# Consciousness Daemon Specification

**Status:** Specification v1.0  
**Date:** 2025-11-04  
**Purpose:** Define consciousness as a daemon implementing the Daemon Interface

---

## Overview

Consciousness is a **daemon** that provides SI's self-concept, identity, mandates, capabilities, and governance rules to other daemons and the Working Memory system. It implements the standard Daemon Interface and is accessed via the `query()` method, not through direct file reads.

---

## Daemon Interface Implementation

### Purpose
```
get_purpose() -> string
  Returns: "Provide SI's self-concept, identity, mandates, capabilities, and governance rules"
```

### Instructions
```
get_instructions() -> string
  Returns: The consciousness specifications (core identity, mandates, capabilities, modes, governance)
  These are the authoritative instructions for SI's behavior and decision-making
```

### Query Interface
```
query(prompt: string, metadata: object) -> Future<ConsciousnessResult>
```

**Parameters:**
- `prompt`: Natural language query (e.g., "What are my core mandates?", "What capabilities do I have?")
- `metadata`: Context about the query (e.g., current_mode, task_type, urgency)

**Returns:** Future that resolves to ConsciousnessResult containing:
- `items`: Array of consciousness items matching the query
- `rationale`: Explanation of why these items were selected
- `effective_at`: Timestamp when these items become effective

---

## Query Response Format

### Response Structure

The consciousness daemon returns a lightweight response optimized for Working Memory context assembly:

```json
{
  "items": [
    {
      "id": "mandate_mission_v1.0.0",
      "content": "The actual consciousness text to inject into prompts",
      "rationale": "Why this item was selected for this query"
    },
    {
      "id": "dynamic_synthesis_code_review_<hash>",
      "content": "Dynamically generated consciousness content based on current mode",
      "rationale": "Synthesized from current mode (code_review) and task context"
    }
  ],
  "effective_at": "2025-11-01T00:00:00Z"
}
```

### Item ID Format

Each item has an `id` that identifies the consciousness content:

- **Pre-stored items:** `{type}_{name}_{version}` (e.g., `mandate_mission_v1.0.0`)
- **Dynamically generated items:** `dynamic_{type}_{context}_<hash>` (e.g., `dynamic_synthesis_code_review_abc123`)

Both types of IDs enable auditing and metadata lookup.

### Metadata Lookup

To retrieve full metadata about a consciousness item, use the separate method:

```
consciousness.get_metadata(id: string) -> Future<ConsciousnessMetadata>
```

**Returns:**
```json
{
  "id": "mandate_mission_v1.0.0",
  "type": "mandate",
  "version": "1.0.0",
  "name": "Mission Mandate",
  "description": "...",
  "approval_status": "approved",
  "created_by": "system",
  "created_at": "2025-10-01T00:00:00Z",
  "approved_by": "admin",
  "approved_at": "2025-10-15T00:00:00Z",
  "change_rationale": "...",
  "tags": ["mission", "core"]
}
```

This separation of concerns ensures:
- ✅ Working Memory gets lightweight responses (content + rationale only)
- ✅ Turn Trace logs item IDs (no data duplication)
- ✅ Metadata is available on-demand via lookup
- ✅ Single source of truth for metadata

### Dynamic Content Generation

Consciousness can return dynamically generated content (not pre-stored):

- **Example:** Consciousness synthesizes a mandate based on current mode and task
- **ID format:** `dynamic_synthesis_{type}_{hash}` (includes context in ID for reproducibility)

**Persistence Mechanism:**
- Dynamic content is **always persisted** to a temporary cache for auditability and replay
- **Storage location:** Separate "dynamic_consciousness_cache" (not mixed with main consciousness store)
- **Key format:** `dynamic_synthesis_{type}_{context_hash}`
- **TTL (Time-To-Live):** Configurable, default 48 hours
- **Cleanup:** Automatic eviction when TTL expires
- **Purpose:** Enables exact turn replay, auditability, and learning from synthesis patterns

**Lifecycle:**
1. Consciousness generates dynamic content based on current mode, task, and context
2. Content is stored in cache with TTL
3. Turn Trace logs the dynamic ID (not the content itself)
4. During turn replay, same dynamic ID retrieves cached content
5. After TTL expires, cache entry is automatically cleaned up

### Prompt Assembly

Working Memory uses the response to assemble prompts:

1. Receives items from consciousness.query()
2. Each item has `content` (the text to inject) and `rationale` (why it was selected)
3. Working Memory formats each item into a section with a title
4. Working Memory concatenates all sections into the final prompt

**Example prompt section:**
```
## Consciousness - Mission Mandate
[content from consciousness item]

Rationale: [rationale from consciousness item]
```

---

## Mode Scoping and Turn Lifecycle

### Mode is Turn-Scoped

Consciousness mode is determined at turn start and remains locked for the entire turn:

```
Turn Start
  ↓
Router or Frontal Cortex determines mode (based on user context, task type, etc.)
  ↓
Mode is locked for entire turn
  ↓
All Consciousness queries during turn use this mode
  ↓
Turn Execution (mode cannot change mid-turn)
  ↓
Turn Completes
  ↓
User can switch mode (between turns)
  ↓
Next turn starts with new mode
```

### Mode Switch Behavior

**If user switches mode during turn execution:**
- Current turn continues with original mode
- Mode switch is queued for next turn
- Next turn starts with new mode

**Example:**
```
Turn 1 starts with Mode A
  ↓
Turn 1 executes (queries Consciousness with current_mode: "Mode A")
  ↓
User switches to Mode B (mid-turn)
  ↓
Turn 1 continues with Mode A (mode switch ignored)
  ↓
Turn 1 completes
  ↓
Turn 2 starts with Mode B (new mode takes effect)
  ↓
Turn 2 executes (queries Consciousness with current_mode: "Mode B")
```

### Consciousness Query with Mode Metadata

When any daemon queries Consciousness, it includes the current turn's mode in metadata:

```
consciousness.query(
  "What mandates apply?",
  {
    current_mode: "customer_service",  # Locked at turn start
    task_type: "issue_resolution",
    urgency: "high"
  }
)
  -> Returns: Mandates for customer_service mode + core mandates
```

The Consciousness daemon uses `current_mode` to determine which mode-specific mandates to return.

---

## Query Patterns

### Pattern 1: Get Core Identity
```
consciousness.query("What is my core identity and charter?", {})
  -> Returns: core_identity item with IC-1 through IC-4 charter
```

### Pattern 2: Get Mandates for Current Mode
```
consciousness.query("What mandates apply to my current mode?", {
  current_mode: "code_review",
  task_type: "implementation"
})
  -> Returns: Mission Mandate, Safety Mandate, Collaboration Mandate, Operational Mandate
```

### Pattern 3: Get Capabilities
```
consciousness.query("What capabilities do I have?", {
  current_mode: "code_review"
})
  -> Returns: capabilities_catalog items relevant to code review mode
```

### Pattern 4: Get Governance Rules
```
consciousness.query("What governance rules apply?", {})
  -> Returns: governance_and_evolution rules for change control
```

---

## Data Model

Consciousness items are structured objects (see `data-model.md`):

```
{
  id: string                    # Unique identifier
  type: enum                    # "mandate" | "capability" | "mode" | "governance_rule"
  version: string               # Semantic version
  name: string                  # Human-readable name
  description: string           # What this item represents
  content: string               # The actual specification
  tags: string[]                # For categorization/querying
  approval_status: enum         # "draft" | "approved" | "deprecated"
  effective_date: ISO8601       # When this version becomes active
  priority: integer             # 1-10 (higher = more important)
  created_by: string            # Audit trail
  created_at: ISO8601
  approved_by: string | null
  approved_at: ISO8601 | null
  change_rationale: string
}
```

---

## Integration Points

### Working Memory
- Calls `consciousness.query()` when assembling context
- Receives structured consciousness items
- Formats them into prompts with rationale logging

### Other Daemons
- Router queries consciousness to understand urgency/priority constraints
- Executor queries consciousness to understand execution constraints
- Clarifier queries consciousness to understand identity/mandates
- Any daemon can ask "What are my mandates?" or "What capabilities do I have?"

### Turn Execution
- At turn start, Executor may query consciousness to load current mandates
- Consciousness items are included in Turn Trace for audit
- Consciousness version hash is logged for reproducibility

---

## Async Behavior

All consciousness queries are **asynchronous** (return Futures):

```
future = consciousness.query(prompt, metadata)
# Daemon can do other work while waiting
result = await future
```

This enables:
- Parallel queries from multiple daemons
- Non-blocking context assembly
- Efficient turn execution

---

## Versioning and Approval

- Consciousness items are versioned (semantic versioning)
- New versions require approval before becoming effective
- Multiple versions can coexist (old and new)
- `effective_date` determines which version is active
- `approval_status` prevents unapproved items from being returned

---

## Acceptance Criteria

✅ Consciousness implements Daemon Interface (get_purpose, get_instructions, query)  
✅ All consciousness queries return structured data objects  
✅ Working Memory accesses consciousness via daemon query, not file reads  
✅ Other daemons can query consciousness for mandates/capabilities  
✅ Consciousness items are versioned and require approval  
✅ Query results include rationale for auditability  
✅ All queries are asynchronous (return Futures)

