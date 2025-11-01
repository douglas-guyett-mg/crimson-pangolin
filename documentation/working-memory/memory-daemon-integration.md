# Memory Daemon Integration

**Status**: Specification v0.1  
**Last Updated**: 2025-10-31  
**Purpose**: Document what memory daemons are and how they receive state updates

## Overview

Memory Daemons are autonomous background processes that receive state updates from the Memory Orchestrator at turn end. They process these updates asynchronously to maintain and evolve Si's memory systems.

This document specifies the daemon architecture, state update protocol, and integration with the turn lifecycle.

## Memory Daemon Architecture

### What Are Memory Daemons?

Memory Daemons are autonomous processes that:

1. **Receive State Updates**: Get notified when turns complete
2. **Process Asynchronously**: Run independently without blocking turns
3. **Maintain Memory Systems**: Update episodic, semantic, and other memory stores
4. **Extract Patterns**: Learn from observations and experiences
5. **Evolve Knowledge**: Improve memory quality over time
6. **Manage Lifecycle**: Archive old memories, consolidate patterns

### Daemon Types

Si has multiple specialized daemons:

1. **Episodic Memory Daemon**
   - Stores turn events and observations
   - Manages episodic memory lifecycle
   - Handles memory archival and cleanup

2. **Semantic Memory Daemon**
   - Extracts patterns from episodic memory
   - Consolidates facts and skills
   - Updates semantic knowledge base

3. **Skill Extraction Daemon**
   - Identifies learned skills from turns
   - Consolidates skill knowledge
   - Updates capability registry

4. **Pattern Recognition Daemon**
   - Detects recurring patterns in behavior
   - Identifies user preferences and habits
   - Updates pattern knowledge base

5. **Memory Consolidation Daemon**
   - Merges related memories
   - Removes duplicates
   - Optimizes memory storage

6. **Lifecycle Management Daemon**
   - Archives old memories
   - Enforces retention policies
   - Manages memory quotas

## State Update Protocol

### State Export at Turn End

At turn end, Memory Orchestrator exports state:

```
Turn Finalization
  ↓
1. Collect turn data
   - Turn ID, timestamp, duration
   - User input, assistant response
   - Tools executed, results
   - Errors and warnings
   - Observations added
   - Goals updated
   ↓
2. Create state export
   - Format: StateExport object
   - Include: all relevant turn data
   - Include: metadata and provenance
   ↓
3. Publish to daemons
   - Send to message queue
   - Each daemon receives copy
   - Daemons process independently
   ↓
4. Continue to next turn
   - Don't wait for daemon processing
   - Daemons work asynchronously
```

### StateExport Data Structure

```yaml
StateExport:
  turn_id: string (ULID)
  agent_id: string
  timestamp: ISO-8601
  duration_ms: integer
  
  # Turn execution data
  turn_data:
    user_input: string
    assistant_response: string
    urgency: "HIGH" | "MEDIUM" | "LOW"
    mode: string
    parent_turn_id: string | null
    
  # Tool execution data
  tool_executions:
    - tool_id: string
      status: "success" | "failure" | "partial"
      inputs: object
      outputs: object
      duration_ms: integer
      tokens_used: integer
      error: string | null
      observations_added: ObservationItem[]
      memory_writes: MemoryWrite[]
  
  # Observations and insights
  observations:
    - id: string
      type: string
      content: string
      source: string
      confidence: number
      tags: string[]
  
  # Goal and action updates
  goal_updates:
    - goal_id: string
      status: "active" | "completed" | "failed"
      progress: number (0-1)
      notes: string
  
  action_updates:
    - action_id: string
      status: "pending" | "completed" | "failed"
      result: object
  
  # Errors and warnings
  errors: string[]
  warnings: string[]
  
  # Metadata
  metadata:
    hobgoblins_activated: string[]
    tools_used: string[]
    memory_types_accessed: string[]
    budget_used: integer
    budget_total: integer
```

### Message Queue

Daemons receive updates via message queue:

```
StateExport published
  ↓
Message Queue
  ├─→ Episodic Memory Daemon
  ├─→ Semantic Memory Daemon
  ├─→ Skill Extraction Daemon
  ├─→ Pattern Recognition Daemon
  ├─→ Memory Consolidation Daemon
  └─→ Lifecycle Management Daemon
  
Each daemon:
  1. Receives message
  2. Processes independently
  3. Updates its memory store
  4. Publishes completion event
```

## Daemon Processing

### Episodic Memory Daemon

**Responsibilities**:
- Store turn events in episodic memory
- Index by timestamp, tags, importance
- Link to related memories
- Manage episodic memory lifecycle

**Processing**:
```
Receive StateExport
  ↓
1. Extract turn events
   - Tool executions
   - Observations
   - Goal updates
   - Errors/warnings
   ↓
2. Create episodic items
   - Format: MemoryItem objects
   - Include: metadata, provenance
   ↓
3. Store in episodic memory
   - Add to episodic store
   - Index for retrieval
   ↓
4. Link related memories
   - Find related past events
   - Create links/references
   ↓
5. Publish completion event
```

### Semantic Memory Daemon

**Responsibilities**:
- Extract patterns from episodic memory
- Consolidate facts and skills
- Update semantic knowledge base
- Manage semantic memory lifecycle

**Processing**:
```
Receive StateExport
  ↓
1. Analyze observations
   - Extract facts from observations
   - Extract skills from tool results
   - Extract patterns from behavior
   ↓
2. Consolidate with existing knowledge
   - Check for duplicates
   - Merge related facts
   - Update confidence scores
   ↓
3. Update semantic memory
   - Add new facts/skills
   - Update existing entries
   - Remove contradictions
   ↓
4. Publish completion event
```

### Skill Extraction Daemon

**Responsibilities**:
- Identify learned skills from turns
- Consolidate skill knowledge
- Update capability registry
- Track skill proficiency

**Processing**:
```
Receive StateExport
  ↓
1. Analyze tool executions
   - Identify successful patterns
   - Identify failure patterns
   - Extract learned skills
   ↓
2. Consolidate skills
   - Group related skills
   - Update proficiency scores
   - Track skill evolution
   ↓
3. Update capability registry
   - Add new capabilities
   - Update existing capabilities
   - Remove obsolete capabilities
   ↓
4. Publish completion event
```

### Pattern Recognition Daemon

**Responsibilities**:
- Detect recurring patterns in behavior
- Identify user preferences and habits
- Update pattern knowledge base
- Track pattern evolution

**Processing**:
```
Receive StateExport
  ↓
1. Analyze user behavior
   - Extract user preferences
   - Identify habits
   - Detect patterns
   ↓
2. Compare with historical patterns
   - Find similar past patterns
   - Update pattern confidence
   - Detect new patterns
   ↓
3. Update pattern knowledge base
   - Add new patterns
   - Update existing patterns
   - Remove obsolete patterns
   ↓
4. Publish completion event
```

### Memory Consolidation Daemon

**Responsibilities**:
- Merge related memories
- Remove duplicates
- Optimize memory storage
- Improve memory quality

**Processing**:
```
Receive StateExport
  ↓
1. Identify consolidation opportunities
   - Find duplicate memories
   - Find related memories
   - Find redundant memories
   ↓
2. Consolidate memories
   - Merge duplicates
   - Combine related items
   - Remove redundancy
   ↓
3. Update memory stores
   - Remove old items
   - Add consolidated items
   - Update indices
   ↓
4. Publish completion event
```

### Lifecycle Management Daemon

**Responsibilities**:
- Archive old memories
- Enforce retention policies
- Manage memory quotas
- Clean up expired items

**Processing**:
```
Receive StateExport
  ↓
1. Check retention policies
   - Identify items past retention
   - Identify items to archive
   - Identify items to delete
   ↓
2. Check memory quotas
   - Calculate current usage
   - Identify over-quota stores
   - Determine items to archive
   ↓
3. Execute lifecycle actions
   - Archive old items
   - Delete expired items
   - Compress archived items
   ↓
4. Publish completion event
```

## Daemon Coordination

### Dependency Graph

Daemons have dependencies:

```
StateExport published
  ↓
Phase 1 (parallel):
  - Episodic Memory Daemon
  - Skill Extraction Daemon
  ↓
Phase 2 (parallel):
  - Semantic Memory Daemon (depends on Phase 1)
  - Pattern Recognition Daemon (depends on Phase 1)
  ↓
Phase 3 (parallel):
  - Memory Consolidation Daemon (depends on Phase 2)
  - Lifecycle Management Daemon (depends on Phase 2)
```

### Daemon Scheduling

```
1. Publish StateExport to queue
   ↓
2. Phase 1 daemons start immediately
   ↓
3. Wait for Phase 1 completion
   ↓
4. Phase 2 daemons start
   ↓
5. Wait for Phase 2 completion
   ↓
6. Phase 3 daemons start
   ↓
7. All daemons complete
```

### Completion Events

Each daemon publishes completion event:

```yaml
DaemonCompletionEvent:
  daemon_id: string
  turn_id: string
  timestamp: ISO-8601
  status: "success" | "failure" | "partial"
  items_processed: integer
  items_added: integer
  items_updated: integer
  items_removed: integer
  duration_ms: integer
  error: string | null
```

## Error Handling

### Daemon Failures

If daemon fails:
1. Log error to Turn Trace
2. Publish failure event
3. Continue with other daemons
4. Don't block turn execution

### Retry Logic

Failed daemons can retry:
- Retry up to 3 times
- Exponential backoff: 1s, 2s, 4s
- Log each retry attempt
- Escalate after 3 failures

### Consistency Guarantees

Daemons maintain consistency:
- Atomic updates (all-or-nothing)
- No partial updates visible
- Conflict resolution: last-write-wins
- Eventual consistency model

## Performance Targets

- **StateExport creation**: < 100ms
- **Message queue publish**: < 50ms
- **Episodic daemon processing**: < 500ms
- **Semantic daemon processing**: < 1000ms
- **Skill extraction daemon**: < 500ms
- **Pattern recognition daemon**: < 1000ms
- **Memory consolidation daemon**: < 500ms
- **Lifecycle management daemon**: < 200ms
- **Total daemon processing**: < 3000ms (parallel)

## Testing Requirements

### Test Suite 1: State Export

**TC-1.1: StateExport creation**
- Turn completes
- Expected: StateExport created with all data
- Verify: All fields populated correctly

**TC-1.2: StateExport publication**
- StateExport created
- Expected: Published to all daemons
- Verify: Each daemon receives copy

### Test Suite 2: Daemon Processing

**TC-2.1: Episodic daemon processes**
- StateExport published
- Expected: Episodic daemon processes
- Verify: Events stored in episodic memory

**TC-2.2: Semantic daemon processes**
- StateExport published
- Expected: Semantic daemon processes
- Verify: Facts/skills stored in semantic memory

### Test Suite 3: Daemon Coordination

**TC-3.1: Phase dependencies respected**
- Phase 1 daemons start
- Phase 2 daemons wait
- Expected: Phase 2 starts after Phase 1
- Verify: Correct ordering

**TC-3.2: Parallel execution**
- Phase 1 daemons execute
- Expected: All Phase 1 daemons run in parallel
- Verify: Total time ≈ max(daemon times)

### Test Suite 4: Error Handling

**TC-4.1: Daemon failure**
- Daemon fails during processing
- Expected: Error logged, other daemons continue
- Verify: Turn execution not blocked

## Next Steps

See related documentation:
- `documentation/turn/overview.md` - Turn architecture
- `documentation/working-memory/memory-orchestrator/overview.md` - Memory Orchestrator
- `documentation/turn/hobgoblins/reflector.md` - Reflector hobgoblin (learning)

