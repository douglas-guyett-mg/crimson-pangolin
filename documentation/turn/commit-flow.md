# Turn Commit Flow & State Export

**Status:** v0.1 (Draft)
**Last Updated:** 2025-11-07
**Purpose:** Document the detailed sequence of turn finalization and state export

## Overview

At the end of each turn, Si commits its state to memory systems and exports updates to memory daemons. This process ensures that learning persists across turns and that the system evolves over time.

This document specifies the turn finalization sequence, state selection criteria, and export mechanisms.

## Turn Finalization Sequence

### Phase 1: Execution Completion (0-100ms)

```
Turn execution completes
  ↓
1. Signal completion
   - Set turn_status = "completing"
   - Record completion_time
   ↓
2. Collect execution results
   - Gather all daemon outputs
   - Gather all tool results
   - Gather all errors/warnings
   ↓
3. Finalize response
   - Responder generates final response
   - Response is ready to send
   ↓
4. Prepare for commit
   - Set turn_status = "ready_for_commit"
```

### Phase 2: Evaluation & Reflection (100-300ms)

```
Turn ready for commit
  ↓
1. Evaluator final assessment
   - Evaluate turn success
   - Identify issues
   - Determine if follow-up needed
   ↓
2. Reflector learning
   - Analyze turn outcomes
   - Extract lessons learned
   - Identify patterns
   ↓
3. Update Frontal Cortex
   - Mark completed goals
   - Update pending actions
   - Record progress
   ↓
4. Prepare state for export
   - Collect all state changes
   - Prepare for memory systems
```

### Phase 3: State Selection (300-400ms)

```
Evaluation complete
  ↓
1. Determine what to remember
   - Select important observations
   - Select important events
   - Select important learnings
   ↓
2. Apply retention policies
   - Filter by importance
   - Filter by relevance
   - Filter by retention rules
   ↓
3. Prepare state export
   - Format selected state
   - Add metadata
   - Add provenance
```

### Phase 4: Memory Commit (400-600ms)

```
State selected
  ↓
1. Commit to episodic memory
   - Store turn events
   - Store observations
   - Store tool results
   ↓
2. Commit to Frontal Cortex
   - Update goals
   - Update pending actions
   - Update progress
   ↓
3. Update Scratch Page
   - Archive resolved observations
   - Update pending observations
   - Clear completed items
   ↓
4. Update conversation history
   - Store user input
   - Store assistant response
   - Store turn summary
```

### Phase 5: Daemon Notification (600-700ms)

```
Memory commit complete
  ↓
1. Create StateExport
   - Package all state changes
   - Include metadata
   - Include provenance
   ↓
2. Publish to daemons
   - Send to message queue
   - Each daemon receives copy
   ↓
3. Record publication
   - Log to Turn Trace
   - Record timestamp
```

### Phase 6: Finalization (700-800ms)

```
Daemons notified
  ↓
1. Update Turn Trace
   - Record all phases
   - Record timings
   - Record any errors
   ↓
2. Clean up resources
   - Release working memory
   - Close file handles
   - Release locks
   ↓
3. Signal completion
   - Set turn_status = "complete"
   - Record final_time
   ↓
4. Ready for next turn
```

## State Selection Criteria

### What Gets Remembered?

Not all turn data is stored. State selection uses criteria:

```
For each piece of state:
  1. Is it important?
     - Importance score > threshold (default: 0.3)
  ↓
  2. Is it relevant?
     - Relevant to goals or user interests
     - Relevant to future decisions
  ↓
  3. Does it match retention policy?
     - User data: always store
     - Tool results: store if important
     - Errors: store if actionable
     - Observations: store if high confidence
  ↓
  4. Is there space?
     - Check memory quotas
     - Prioritize by importance
     - Archive if needed
  ↓
  Decision: Store or discard
```

### Importance Scoring

Importance is calculated as:

```
importance = (
  0.3 * relevance_to_goals +
  0.2 * relevance_to_user +
  0.2 * confidence +
  0.15 * novelty +
  0.15 * actionability
)

where:
  relevance_to_goals: 0-1 (does it relate to active goals?)
  relevance_to_user: 0-1 (is it about the user?)
  confidence: 0-1 (how confident are we?)
  novelty: 0-1 (is it new information?)
  actionability: 0-1 (can we act on it?)
```

### Retention Policies

Different data types have different retention:

```
User Data:
  - Retention: permanent (until user deletes)
  - Importance threshold: 0.0 (always store)
  - Examples: preferences, history, goals

Tool Results:
  - Retention: 30 days
  - Importance threshold: 0.5
  - Examples: search results, recommendations

Observations:
  - Retention: 7 days
  - Importance threshold: 0.6
  - Examples: insights, patterns, alerts

Errors:
  - Retention: 7 days
  - Importance threshold: 0.7
  - Examples: failures, issues, warnings

Scratch Page:
  - Retention: 1 day (active), 7 days (archived)
  - Importance threshold: 0.0 (all stored)
  - Examples: pending items, alerts
```

## State Export Format

### StateExport Structure

```yaml
StateExport:
  # Metadata
  export_id: string (ULID)
  turn_id: string
  agent_id: string
  timestamp: ISO-8601
  
  # Turn summary
  turn_summary:
    duration_ms: integer
    user_input: string
    assistant_response: string
    urgency: "HIGH" | "MEDIUM" | "LOW"
    success: boolean
    
  # Episodic memory exports
  episodic_exports:
    - type: "tool_execution" | "observation" | "event" | "error"
      data: object
      importance: number (0-1)
      tags: string[]
      
  # Frontal Cortex updates
  fc_updates:
    completed_goals: string[] (goal ids)
    updated_goals: { [goal_id]: GoalUpdate }
    completed_actions: string[] (action ids)
    updated_actions: { [action_id]: ActionUpdate }
    
  # Scratch Page updates
  scratch_page_updates:
    archived_observations: string[] (observation ids)
    updated_observations: { [obs_id]: ObservationUpdate }
    cleared_observations: string[] (observation ids)
    
  # Conversation history
  conversation_update:
    user_input: string
    assistant_response: string
    turn_summary: string
    
  # Metadata
  metadata:
    daemons_activated: string[]
    tools_executed: string[]
    errors: string[]
    warnings: string[]
    budget_used: integer
    budget_total: integer
```

## Commit Operations

### Episodic Memory Commit

```
Episodic Memory Commit:
  1. For each episodic_export:
     a. Create MemoryItem
     b. Add to episodic store
     c. Index by timestamp, tags
     d. Link to related items
  ↓
  2. Update episodic indices
     - Update timestamp index
     - Update tag indices
     - Update importance index
  ↓
  3. Verify storage
     - Confirm items stored
     - Verify indices updated
```

### Frontal Cortex Commit

```
Frontal Cortex Commit:
  1. For each completed_goal:
     a. Mark goal as completed
     b. Record completion time
     c. Archive goal
  ↓
  2. For each updated_goal:
     a. Update goal progress
     b. Update goal status
     c. Record update time
  ↓
  3. For each completed_action:
     a. Mark action as completed
     b. Record result
     c. Archive action
  ↓
  4. For each updated_action:
     a. Update action status
     b. Update action result
     c. Record update time
```

### Scratch Page Commit

```
Scratch Page Commit:
  1. For each archived_observation:
     a. Update status to "archived"
     b. Record archive time
  ↓
  2. For each updated_observation:
     a. Update observation fields
     b. Record update time
  ↓
  3. For each cleared_observation:
     a. Remove from active set
     b. Archive if needed
```

### Conversation History Commit

```
Conversation History Commit:
  1. Store user input
     - Record input text
     - Record timestamp
     - Record metadata
  ↓
  2. Store assistant response
     - Record response text
     - Record timestamp
     - Record metadata
  ↓
  3. Create turn summary
     - Summarize key points
     - Extract decisions
     - Extract outcomes
  ↓
  4. Update rolling summary
     - Incorporate turn into summary
     - Update key topics
     - Update decision log
```

## Error Handling

### Commit Failures

If commit fails:
1. Log error to Turn Trace
2. Retry up to 3 times
3. If still failing: escalate
4. Don't block turn completion

### Partial Commits

If some commits succeed, others fail:
1. Record which commits succeeded
2. Record which commits failed
3. Retry failed commits
4. Don't lose successful commits

### Consistency Issues

If consistency issues detected:
1. Log issue to Turn Trace
2. Use last-write-wins strategy
3. Continue with available data
4. Flag for investigation

## Performance Targets

- **Phase 1 (Execution Completion)**: < 100ms
- **Phase 2 (Evaluation & Reflection)**: < 200ms
- **Phase 3 (State Selection)**: < 100ms
- **Phase 4 (Memory Commit)**: < 200ms
- **Phase 5 (Daemon Notification)**: < 100ms
- **Phase 6 (Finalization)**: < 100ms
- **Total Commit Flow**: < 800ms

## Testing Requirements

### Test Suite 1: State Selection

**TC-1.1: Important state is selected**
- State with importance > 0.5
- Expected: State selected for export
- Verify: State in StateExport

**TC-1.2: Unimportant state is filtered**
- State with importance < 0.3
- Expected: State not selected
- Verify: State not in StateExport

**TC-1.3: Retention policies enforced**
- Tool result with 30-day retention
- Expected: Stored in episodic memory
- Verify: Retention metadata set

### Test Suite 2: Commit Operations

**TC-2.1: Episodic commit**
- StateExport with episodic_exports
- Expected: Items stored in episodic memory
- Verify: Items retrievable

**TC-2.2: FC commit**
- StateExport with fc_updates
- Expected: Goals/actions updated
- Verify: Updates reflected in FC

**TC-2.3: Scratch page commit**
- StateExport with scratch_page_updates
- Expected: Observations updated
- Verify: Updates reflected in Scratch Page

### Test Suite 3: Error Handling

**TC-3.1: Commit failure recovery**
- Commit fails
- Expected: Retry and succeed
- Verify: State eventually committed

**TC-3.2: Partial commit**
- Some commits succeed, some fail
- Expected: Successful commits persist
- Verify: Failed commits retried

### Test Suite 4: Performance

**TC-4.1: Commit completes within budget**
- Turn completes
- Expected: Commit < 800ms
- Verify: Performance target met

## Next Steps

See related documentation:
- `documentation/turn/overview.md` - Turn architecture
- `documentation/working-memory/memory-daemon-integration.md` - Memory daemons
- `documentation/frontal-cortex/README.md` - Frontal Cortex
- `documentation/turn/daemons/evaluator.md` - Evaluator daemon
- `documentation/turn/daemons/reflector.md` - Reflector daemon

