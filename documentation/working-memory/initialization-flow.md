# Working Memory Initialization Flow

**Status**: Specification v0.1  
**Last Updated**: 2025-10-31  
**Purpose**: Document the sequence of how Working Memory initializes and loads relevant information at turn start

## Overview

At the start of each turn, the Memory Orchestrator initializes the Working Memory system by loading relevant information from multiple sources. This initialization creates a coherent context that hobgoblins use for decision-making.

The initialization flow is deterministic and follows a prescribed sequence to ensure consistent behavior across turns.

## Initialization Sequence

### Phase 1: Turn Context Setup (0-50ms)

```
Turn Start Event
  ↓
1. Create Turn Context
   - turn_id: Generate unique ULID
   - agent_id: From request
   - user_id: From request
   - timestamp: Current time
   - urgency: Not yet determined (set by Router)
   - parent_turn_id: If continuation of previous turn
   ↓
2. Initialize Budget
   - total_token_budget: From configuration (default: 4000)
   - allocated_budget: {}  (empty, will be allocated in Phase 3)
   - used_tokens: 0
   - remaining_tokens: total_token_budget
   ↓
3. Initialize Stores
   - working_memory_store: Empty buffer
   - conversation_history: Ready to query
   - episodic_memory: Ready to query
   - semantic_memory: Ready to query
   - scratch_page: Ready to query
```

### Phase 2: Load Consciousness (50-150ms)

```
2a. Load Core Identity
   - agent_id: Retrieve from consciousness
   - agent_name: Retrieve from consciousness
   - agent_values: Retrieve from consciousness
   - agent_mandates: Retrieve from consciousness
   ↓
2b. Load Governance Rules
   - safety_policies: Retrieve from consciousness
   - access_control_rules: Retrieve from consciousness
   - escalation_policies: Retrieve from consciousness
   ↓
2c. Load Active Modes
   - current_mode: Determine from context
   - mode_constraints: Retrieve from consciousness
   - mode_capabilities: Retrieve from consciousness
   ↓
2d. Load Capabilities
   - available_capabilities: Retrieve from consciousness
   - capability_constraints: Retrieve from consciousness
   - capability_versions: Retrieve from consciousness
   ↓
Result: Consciousness Bundle
  - core_identity: { agent_id, agent_name, values, mandates }
  - governance: { safety_policies, access_control, escalation }
  - modes: { current_mode, constraints, capabilities }
  - capabilities: { available, constraints, versions }
```

### Phase 3: Load Frontal Cortex (150-250ms)

```
3a. Load Active Goals
   - Query FC: get_active_goals(agent_id, limit=5)
   - Filter by: relevance to current context
   - Sort by: priority, due_date, blocking status
   ↓
3b. Load Pending Actions
   - Query FC: list_pending_actions(agent_id, status="active")
   - Filter by: owner (agent vs user), priority
   - Sort by: due_date, blocking status, priority
   ↓
3c. Load Questions & Stories
   - Query FC: get_questions(agent_id, limit=3)
   - Query FC: get_stories(agent_id, limit=3)
   - Filter by: relevance to current context
   ↓
3d. Load Progress Deltas
   - Query FC: get_progress_deltas(agent_id, since=last_turn)
   - Include: completed actions, updated goals, new insights
   ↓
Result: Frontal Cortex Bundle
  - active_goals: [ Goal, ... ]
  - pending_actions: [ PendingAction, ... ]
  - questions: [ Question, ... ]
  - stories: [ Story, ... ]
  - progress_deltas: { completed, updated, new }
```

### Phase 4: Load Conversation History (250-350ms)

```
4a. Load Recent Turns
   - Query Conversation History: get_recent_turns(agent_id, limit=10)
   - Include: user messages, assistant responses, turn summaries
   ↓
4b. Load Rolling Summary
   - Query Conversation History: get_summary(agent_id)
   - Include: key topics, decisions, outcomes
   ↓
4c. Load Context from Previous Turn
   - If parent_turn_id exists:
     - Load turn trace from previous turn
     - Load any pending work from previous turn
     - Load any errors or issues from previous turn
   ↓
Result: Conversation History Bundle
  - recent_turns: [ Turn, ... ]
  - summary: ConversationSummary
  - previous_turn_context: TurnContext (if applicable)
```

### Phase 5: Load Episodic Memory (350-500ms)

```
5a. Build Composite Query
   - Current user input/prompt
   - Active goals from FC
   - Recent conversation context
   - Current mode and capabilities
   ↓
5b. Query Episodic Memory
   - Query: hybrid_search(composite_query, budget=500_tokens)
   - Include: relevant events, observations, tool results
   - Filter by: recency, relevance, importance
   ↓
5c. Re-rank Results
   - Score by: similarity, recency, importance, novelty
   - Remove duplicates with working memory
   - Select top-K items within budget
   ↓
Result: Episodic Memory Bundle
  - relevant_events: [ MemoryItem, ... ]
  - token_count: integer
  - search_metadata: { query, filters, ranking_scores }
```

### Phase 6: Load Semantic Memory (500-650ms)

```
6a. Build Semantic Query
   - Extract key concepts from user input
   - Extract key concepts from active goals
   - Extract key concepts from recent conversation
   ↓
6b. Query Semantic Memory
   - Query: hybrid_search(semantic_query, budget=300_tokens)
   - Include: facts, skills, patterns, generalizations
   - Filter by: relevance, validity, agent_id
   ↓
6c. Re-rank Results
   - Score by: similarity, importance, recency
   - Remove duplicates with episodic memory
   - Select top-K items within budget
   ↓
Result: Semantic Memory Bundle
  - relevant_facts: [ MemoryItem, ... ]
  - relevant_skills: [ MemoryItem, ... ]
  - relevant_patterns: [ MemoryItem, ... ]
  - token_count: integer
```

### Phase 7: Load Scratch Page (650-700ms)

```
7a. Query Active Observations
   - Query: list_observations(agent_id, status="active")
   - Include: pending_confirmations, risk_alerts, contextual_insights
   ↓
7b. Filter by Relevance
   - Filter by: tags matching current context
   - Filter by: owner (agent vs user)
   - Sort by: priority, recency
   ↓
Result: Scratch Page Bundle
  - active_observations: [ ObservationItem, ... ]
  - risk_alerts: [ ObservationItem, ... ]
  - pending_confirmations: [ ObservationItem, ... ]
```

### Phase 8: Allocate Budget (700-750ms)

```
8a. Calculate Total Loaded Tokens
   - Sum tokens from all bundles
   - consciousness_tokens: ~200
   - fc_tokens: ~300
   - conversation_tokens: ~500
   - episodic_tokens: 500 (allocated)
   - semantic_tokens: 300 (allocated)
   - scratch_page_tokens: ~100
   - Total: ~1800 tokens
   ↓
8b. Allocate Remaining Budget
   - Remaining budget: 4000 - 1800 = 2200 tokens
   - Allocate to: working_memory (1500), tools (500), buffer (200)
   ↓
Result: Budget Allocation
  - consciousness: 200 tokens
  - fc: 300 tokens
  - conversation: 500 tokens
  - episodic: 500 tokens
  - semantic: 300 tokens
  - scratch_page: 100 tokens
  - working_memory: 1500 tokens
  - tools: 500 tokens
  - buffer: 0 tokens
```

### Phase 9: Assemble Initial Context (750-850ms)

```
9a. Format Consciousness Fragment
   - Include: core identity, mandates, current mode
   - Format: Structured prompt fragment with metadata
   ↓
9b. Format FC Fragment
   - Include: active goals, pending actions, questions
   - Format: Structured prompt fragment with metadata
   ↓
9c. Format Conversation Fragment
   - Include: recent turns, summary, previous context
   - Format: Structured prompt fragment with metadata
   ↓
9d. Format Memory Fragments
   - Include: episodic, semantic, scratch page
   - Format: Structured prompt fragments with metadata
   ↓
9e. Combine Fragments
   - Order: Consciousness → FC → Conversation → Memory
   - Include: provenance metadata for each fragment
   - Include: token count for each fragment
   ↓
Result: Initial Context Bundle
  - formatted_context: string
  - total_tokens: integer
  - source_breakdown: { consciousness, fc, conversation, episodic, semantic, scratch_page }
  - metadata: { timestamps, versions, checksums }
```

### Phase 10: Initialize Working Memory Store (850-900ms)

```
10a. Create WM Buffer
   - Initialize: empty buffer with max_items=64, max_tokens=4000
   ↓
10b. Add Initial Items
   - Add: consciousness fragment
   - Add: FC fragment
   - Add: conversation fragment
   - Add: memory fragments
   ↓
10c. Set WM State
   - current_position: 0
   - step_index: 0
   - budget_remaining: 2200 tokens
   ↓
Result: Working Memory Store Ready
  - buffer: [ MemoryItem, ... ]
  - position: 0
  - budget_remaining: 2200
```

### Phase 11: Finalize Initialization (900-950ms)

```
11a. Create Initialization Record
   - Record: all bundles loaded
   - Record: context assembly
   - Record: timestamps for each phase
   ↓
11b. Log to Turn Trace
   - Log: initialization started
   - Log: all phases completed
   - Log: total initialization time
   - Log: any errors or warnings
   ↓
11c. Signal Ready
   - Set: initialization_complete = true
   - Set: ready_for_router = true
   - Return: Initialized Memory Orchestrator
   ↓
Result: Initialization Complete
  - Memory Orchestrator ready for Router
  - All context loaded and formatted
  - Turn Trace updated
```

## Error Handling

### Phase Failures

If any phase fails:
1. Log error to Turn Trace
2. Continue with available data
3. Reduce budget for failed phase
4. Signal warning to Router

### Timeout Handling

If initialization exceeds 1000ms:
1. Cancel remaining phases
2. Use partial context
3. Log timeout to Turn Trace
4. Reduce budget accordingly

### Data Consistency

If data is inconsistent:
1. Use most recent version
2. Log inconsistency to Turn Trace
3. Continue with available data
4. Flag for investigation

## Performance Targets

- **Total initialization time**: < 1000ms (target: 950ms)
- **Phase 1 (Context Setup)**: < 50ms
- **Phase 2 (Consciousness)**: < 100ms
- **Phase 3 (Frontal Cortex)**: < 100ms
- **Phase 4 (Conversation)**: < 100ms
- **Phase 5 (Episodic)**: < 150ms
- **Phase 6 (Semantic)**: < 150ms
- **Phase 7 (Scratch Page)**: < 50ms
- **Phase 8-11 (Assembly & Finalization)**: < 200ms

## Testing Requirements

- Deterministic initialization with fixed seeds
- All phases complete successfully
- Budget allocation is correct
- Context assembly is complete
- Turn Trace is updated
- Error handling works correctly
- Performance targets are met
- Concurrent initializations don't interfere

## Next Steps

See related documentation:
- `documentation/turn/hobgoblins/router.md` - How Router uses initialized context
- `documentation/working-memory/memory-orchestrator/overview.md` - Orchestrator operations
- `documentation/turn/overview.md` - Turn architecture

