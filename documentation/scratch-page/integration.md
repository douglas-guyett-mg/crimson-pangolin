# Scratch Page Integration

**Status**: Specification v0.1  
**Last Updated**: 2025-10-31

## Overview

The Scratch Page integrates with multiple Si systems to enable cross-turn awareness and informed decision-making. This document specifies how each system interacts with the Scratch Page.

---

## Tool Integration

### How Tools Add Observations

Tools can add observations during execution via the ToolResponse:

```
ToolResponse {
  request_id: "...",
  status: "ok",
  outputs: { ... },
  observations: [ ... ],
  memory_writes: [ ... ],
  scratch_page_writes: [
    {
      type: "contextual_insight",
      content: "User will like 2020 Marcel La Pierre at bistro vendome",
      confidence: 0.95,
      tags: ["wine", "recommendation"]
    }
  ]
}
```

### Tool Memory Access

Tools can query the Scratch Page to understand context:

```
ToolRequest {
  request_id: "...",
  tool_id: "wine_recommender",
  inputs: { ... },
  scratch_page_query: {
    filters: { tags: ["wine"], owner: "agent" }
  }
}

ToolResponse includes:
  scratch_page_context: [
    {
      id: "obs_123",
      type: "contextual_insight",
      content: "User prefers Burgundy wines",
      confidence: 0.9
    }
  ]
```

### Tool Observation Types

Tools typically add:
- **contextual_insight**: Findings relevant to future decisions
- **observation**: General findings or notes
- **action_suggestion**: Recommended actions
- **pattern_detected**: Learned patterns about user behavior

### Integration Points

1. **Tool Manifest**: Declare which observation types tool can add
2. **Tool Execution**: Include scratch_page_writes in ToolResponse
3. **Tool Context**: Include scratch_page_query in ToolRequest
4. **Tool Testing**: Test fixtures include scratch page scenarios

---

## Hobgoblin Integration

### Router Integration

**Router reads observations for urgency assessment:**

```
Router Decision Process:
  1. Receive input
  2. Query Scratch Page: get_active_observations()
  3. Check for risk_alerts or pending_confirmations
  4. Adjust urgency based on observations
  5. Make routing decision
```

**Example**: If risk_alert exists for fraud, Router increases urgency for financial transactions.

### Executor Integration

**Executor reads observations for tool selection:**

```
Executor Planning:
  1. Receive plan from Plan Generator
  2. Query Scratch Page: list_observations({ owner: "agent" })
  3. Use observations to refine tool selection
  4. Adjust execution order based on pending states
  5. Execute tools
```

**Example**: If pending_confirmation exists for email, Executor prioritizes verification tools.

### Evaluator Integration

**Evaluator reviews observations for completion:**

```
Evaluator Assessment:
  1. Check if work is complete
  2. Query Scratch Page: list_observations({ status: "pending_review" })
  3. Review pending observations
  4. Determine if additional work needed
  5. Signal completion or need for more work
```

### Reflector Integration

**Reflector archives resolved observations:**

```
Reflector Learning:
  1. Analyze turn outcomes
  2. Query Scratch Page: list_observations({ status: "resolved" })
  3. Archive resolved observations
  4. Extract patterns from observations
  5. Update episodic memory with patterns
```

### Clarifier Integration

**Clarifier may add pending_confirmation observations:**

```
Clarifier Decision:
  1. Detect ambiguity in input
  2. Add observation: { type: "pending_confirmation", content: "Waiting for clarification" }
  3. Ask user for clarification
  4. Update observation when clarification received
```

---

## Memory System Integration

### Working Memory Integration

**Working Memory includes Scratch Page context in context assembly:**

```
assemble_context(prompt, budget):
  1. Allocate budget across memory types
  2. Query Scratch Page: list_observations({ status: "active" })
  3. Include active observations in context
  4. Combine with episodic/semantic memory
  5. Return formatted context
```

**Budget Allocation**: Scratch Page observations get dedicated budget (default: 10% of total).

### Episodic Memory Integration

**Episodic Memory stores observation history:**

```
At turn end:
  1. Export all observations (active, resolved, archived)
  2. Store in episodic memory with turn_id reference
  3. Enable future analysis of observation patterns
  4. Support learning about observation accuracy
```

### Semantic Memory Integration

**Semantic Memory extracts patterns from observations:**

```
Skill Extraction:
  1. Analyze episodic memory observations
  2. Identify recurring patterns
  3. Extract as semantic facts/skills
  4. Example: "User prefers Burgundy wines" → semantic fact
```

---

## Frontal Cortex Integration

### Goal Linking

Observations can reference Frontal Cortex goals:

```
Observation {
  type: "contextual_insight",
  content: "Found 3 flights matching criteria",
  context: { goal_id: "book_flight_to_paris" }
}
```

### Pending Action Linking

Observations can reference pending actions:

```
Observation {
  type: "pending_confirmation",
  content: "Waiting for user to confirm flight selection",
  context: { pending_action_id: "confirm_flight_123" }
}
```

### Progress Tracking

Observations inform FC progress updates:

```
At turn end:
  1. Query Scratch Page for observations linked to goals
  2. Use observations to update goal progress
  3. Mark pending actions complete if observation resolved
  4. Export updates to Frontal Cortex
```

---

## Turn Architecture Integration

### Turn Trace Integration

All Scratch Page operations are logged to Turn Trace:

```
Turn Trace Entry:
  {
    timestamp: "2025-10-31T10:00:00Z",
    component: "scratch_page",
    operation: "add_observation",
    observation_id: "obs_123",
    observation_type: "contextual_insight",
    source: "wine_recommender",
    status: "success"
  }
```

### Parallel Turn Coordination

When turns run in parallel, they share Scratch Page:

```
Turn 1: Adds observation about wine preference
Turn 2: Queries Scratch Page, sees observation
Turn 3: Uses observation in decision-making

All turns see consistent Scratch Page state
```

### Background Turn Integration

Background turns can add observations:

```
Turn 1: Send immediate response
Background: Continue research
  → add_observation({ type: "observation", content: "Research findings" })
Turn 2: Use observation in follow-up response
```

---

## Data Flow Diagrams

### Tool → Scratch Page → Future Turn

```
Tool Execution
  ↓
Tool adds observation
  ↓
Scratch Page stores observation
  ↓
Future turn starts
  ↓
Router/Executor query Scratch Page
  ↓
Observation informs decision-making
```

### Hobgoblin → Scratch Page → Other Hobgoblins

```
Hobgoblin A (e.g., Executor)
  ↓
Adds observation (e.g., "tool X failed")
  ↓
Scratch Page stores observation
  ↓
Hobgoblin B (e.g., Error Handler)
  ↓
Queries Scratch Page
  ↓
Uses observation in error handling
```

### Scratch Page → Memory Systems → Future Turns

```
Scratch Page observations
  ↓
Exported to episodic memory at turn end
  ↓
Patterns extracted to semantic memory
  ↓
Future turns query memory systems
  ↓
Observations inform decisions
```

---

## API Contracts

### Tool ToolRequest Extension

```yaml
ToolRequest:
  scratch_page_query:
    filters:
      type: string[]  # Observation types to query
      status: string[]  # Statuses to include
      tags: string[]  # Tags to match
      owner: string  # Owner filter
    limit: integer  # Max results (default: 10)
```

### Tool ToolResponse Extension

```yaml
ToolResponse:
  scratch_page_context:
    - id: string
      type: string
      content: string
      confidence: number
      created_at: string
  
  scratch_page_writes:
    - type: string
      content: string
      confidence: number
      tags: string[]
      metadata: object
```

### Hobgoblin API

```
scratch_page.add_observation(item: ObservationItem) → ObservationItem
scratch_page.list_observations(filters: ObservationFilter) → ObservationItem[]
scratch_page.get_observation(id: string) → ObservationItem
scratch_page.update_observation(id: string, updates: Partial<ObservationItem>) → ObservationItem
scratch_page.archive_observation(id: string) → void
scratch_page.get_active_observations() → ObservationItem[]
```

---

## Error Handling

### Tool Observation Errors

If tool adds invalid observation:
- Log error to Turn Trace
- Continue tool execution
- Don't block tool result

### Query Errors

If Scratch Page query fails:
- Return empty result set
- Log error to Turn Trace
- Don't block hobgoblin decision-making

### Concurrent Access Errors

If concurrent update conflict:
- Use last-write-wins strategy
- Log conflict to Turn Trace
- Ensure consistency

---

## Performance Considerations

### Query Performance

- Index observations by type, status, owner, tags
- Support efficient filtering on common queries
- Limit default query results (default: 10)

### Storage Performance

- Archive old observations to reduce active set
- Compress archived observations
- Implement pagination for large result sets

### Memory Performance

- Keep active observations in memory
- Archive observations older than 7 days (configurable)
- Implement LRU eviction if needed

---

## Security Considerations

### Access Control

- All observations scoped to agent_id
- Tools can only add observations, not modify others'
- Hobgoblins can read all observations for their agent

### Data Privacy

- Observations may contain PII
- Apply same redaction rules as working memory
- Archive observations with PII after retention period

---

## Testing Integration

### Tool Testing

- Test fixtures include scratch page scenarios
- Verify tools can add observations
- Verify tools can query observations

### Hobgoblin Testing

- Test Router with active risk_alerts
- Test Executor with pending_confirmations
- Test Evaluator with pending_review observations

### Integration Testing

- Test end-to-end flows with observations
- Test concurrent access patterns
- Test memory system integration

