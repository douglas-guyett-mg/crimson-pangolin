# Tool Memory Access

**Status**: Specification v0.1  
**Last Updated**: 2025-10-31  
**Purpose**: Document how tools query and access existing memories

## Overview

Tools can query Si's memory systems to understand context and make informed decisions. This document specifies the memory access APIs available to tools, access patterns, and integration with the tool execution model.

## Memory Access Model

### Access Levels

Tools have read-only access to memory systems:

1. **Working Memory**: Current turn context (read-only)
2. **Episodic Memory**: Historical events and observations (read-only)
3. **Semantic Memory**: Facts, skills, and patterns (read-only)
4. **Scratch Page**: Pending observations and alerts (read-only)
5. **Conversation History**: Recent dialogue (read-only)

Tools can **write** to memory via:
- `memory_writes` in ToolResponse (episodic memory)
- `scratch_page_writes` in ToolResponse (scratch page)

### Access Control

Tools can only access:
- Memories scoped to the current agent_id
- Memories matching the tool's declared capabilities
- Memories not marked as private/restricted

Tools cannot:
- Access memories from other agents
- Modify existing memories
- Delete memories
- Access restricted memories

## Query API

### Memory Query Request

Tools include memory queries in ToolRequest:

```yaml
ToolRequest:
  request_id: string
  tool_id: string
  tool_version: string
  timestamp: ISO-8601
  agent_context:
    agent_id: string
    task_id: string
    plan_id: string
  inputs: object
  
  # Memory access requests
  memory_queries:
    - query_id: string
      memory_type: "episodic" | "semantic" | "working" | "scratch_page" | "conversation"
      query_text: string
      filters:
        tags: string[]
        types: string[]
        time_range: { start: ISO-8601, end: ISO-8601 }
        importance_min: number (0-1)
        source: string
      limit: integer (default: 10)
      budget_tokens: integer (default: 500)
```

### Memory Query Response

Memory system returns results in ToolRequest context:

```yaml
ToolRequest (with memory context):
  memory_context:
    - query_id: string
      memory_type: string
      results:
        - id: string
          type: string
          content: string
          source: string
          timestamp: ISO-8601
          importance: number (0-1)
          tags: string[]
          metadata: object
      total_results: integer
      returned_results: integer
      tokens_used: integer
      search_metadata:
        query_time_ms: integer
        ranking_scores: number[]
```

## Query Types

### Type 1: Episodic Memory Query

Query historical events and observations:

```yaml
memory_queries:
  - query_id: "ep_1"
    memory_type: "episodic"
    query_text: "What wine preferences has the user expressed?"
    filters:
      tags: ["wine", "preference"]
      types: ["observation", "user_preference"]
      time_range:
        start: "2025-01-01T00:00:00Z"
        end: "2025-10-31T23:59:59Z"
    limit: 5
    budget_tokens: 300
```

**Use Cases**:
- Wine recommender: "What wines has user liked?"
- Travel planner: "Where has user traveled before?"
- Health advisor: "What health concerns has user mentioned?"

### Type 2: Semantic Memory Query

Query facts, skills, and patterns:

```yaml
memory_queries:
  - query_id: "sem_1"
    memory_type: "semantic"
    query_text: "What do we know about Burgundy wines?"
    filters:
      tags: ["wine", "burgundy"]
      types: ["fact", "skill", "pattern"]
    limit: 10
    budget_tokens: 400
```

**Use Cases**:
- Wine recommender: "What are characteristics of Burgundy wines?"
- Travel planner: "What are best times to visit Paris?"
- Health advisor: "What are symptoms of common conditions?"

### Type 3: Scratch Page Query

Query pending observations and alerts:

```yaml
memory_queries:
  - query_id: "sp_1"
    memory_type: "scratch_page"
    query_text: "Are there any active alerts or pending confirmations?"
    filters:
      types: ["risk_alert", "pending_confirmation"]
      status: "active"
    limit: 20
    budget_tokens: 200
```

**Use Cases**:
- Any tool: "Are there security alerts I should know about?"
- Any tool: "Are there pending confirmations blocking progress?"
- Financial tool: "Are there fraud alerts for this user?"

### Type 4: Conversation History Query

Query recent dialogue:

```yaml
memory_queries:
  - query_id: "ch_1"
    memory_type: "conversation"
    query_text: "What has the user said about their budget?"
    filters:
      types: ["user_message"]
      time_range:
        start: "2025-10-30T00:00:00Z"
        end: "2025-10-31T23:59:59Z"
    limit: 5
    budget_tokens: 300
```

**Use Cases**:
- Travel planner: "What budget did user mention?"
- Shopping assistant: "What preferences did user express?"
- Any tool: "What context from recent conversation is relevant?"

### Type 5: Working Memory Query

Query current turn context:

```yaml
memory_queries:
  - query_id: "wm_1"
    memory_type: "working"
    query_text: "What are the active goals for this turn?"
    filters:
      types: ["goal", "context"]
    limit: 10
    budget_tokens: 200
```

**Use Cases**:
- Any tool: "What are the current goals?"
- Any tool: "What context is available for this turn?"

## Query Execution

### Query Processing Flow

```
1. Tool includes memory_queries in ToolRequest
   ↓
2. Orchestrator receives ToolRequest
   ↓
3. For each query:
   a. Validate query (schema, filters, budget)
   b. Query appropriate memory type
   c. Apply filters
   d. Rank results
   e. Pack within budget
   f. Format response
   ↓
4. Combine all query results
   ↓
5. Include in ToolRequest context
   ↓
6. Pass to tool for execution
```

### Query Validation

Orchestrator validates:
- Query schema matches specification
- Memory type is valid
- Filters are valid
- Budget is reasonable (max: 1000 tokens)
- Tool has access to memory type

### Query Ranking

Results are ranked by:
- Relevance to query (semantic similarity)
- Recency (newer items ranked higher)
- Importance (higher importance ranked higher)
- Novelty (avoid duplicates)

### Query Budget

Each query has token budget for context assembly:
- Default: 500 tokens
- Max: 1000 tokens
- Shared across all queries in request
- Enforced during packing

**Note**: This is for context assembly only. LLM call budgets are enforced by the LLM Budget System. See `documentation/llm-budget-system/overview.md`.

## Tool Memory Access Patterns

### Pattern 1: Context-Aware Decision Making

```
Tool: wine_recommender
  ↓
1. Query episodic memory: "What wines has user liked?"
   ↓
2. Query semantic memory: "What are characteristics of those wines?"
   ↓
3. Use results to make recommendation
   ↓
4. Add observation to scratch_page: "User will like X wine"
```

### Pattern 2: Constraint Checking

```
Tool: financial_advisor
  ↓
1. Query scratch_page: "Are there fraud alerts?"
   ↓
2. If alerts exist: increase scrutiny
   ↓
3. Query episodic memory: "What transactions has user made?"
   ↓
4. Compare current transaction against history
```

### Pattern 3: Dependency Resolution

```
Tool: travel_planner
  ↓
1. Query working memory: "What are active goals?"
   ↓
2. Query episodic memory: "Where has user traveled?"
   ↓
3. Query semantic memory: "What are best times to visit?"
   ↓
4. Combine all context for planning
```

## Tool Manifest Declaration

Tools declare memory access in manifest:

```yaml
tool:
  id: wine_recommender
  version: 1.0.0
  
  # Declare memory access
  memory_access:
    episodic:
      enabled: true
      query_types: ["observation", "user_preference"]
      tags: ["wine", "preference"]
      max_queries: 3
      max_budget_tokens: 500
    
    semantic:
      enabled: true
      query_types: ["fact", "skill"]
      tags: ["wine"]
      max_queries: 2
      max_budget_tokens: 400
    
    scratch_page:
      enabled: true
      query_types: ["contextual_insight", "risk_alert"]
      max_queries: 1
      max_budget_tokens: 200
    
    conversation:
      enabled: true
      max_queries: 1
      max_budget_tokens: 300
    
    working:
      enabled: false  # Not needed for this tool
```

## Error Handling

### Query Errors

If query fails:
- Return empty results
- Log error to Turn Trace
- Don't block tool execution
- Tool continues with available context

### Access Denied

If tool lacks access:
- Return empty results
- Log access denial to Turn Trace
- Tool continues without restricted memory

### Budget Exceeded

If query exceeds budget:
- Return partial results within budget
- Log budget exceeded to Turn Trace
- Tool continues with available results

## Performance Considerations

### Query Latency

- Episodic query: < 100ms
- Semantic query: < 100ms
- Scratch page query: < 50ms
- Conversation query: < 50ms
- Working memory query: < 10ms

### Query Caching

Orchestrator caches queries:
- Cache key: (agent_id, query_text, filters)
- TTL: 5 minutes
- Reduces redundant queries

### Query Batching

Tools can batch multiple queries:
- All queries processed in parallel
- Results combined in single response
- More efficient than sequential queries

## Testing Requirements

### Test Suite 1: Query Execution

**TC-1.1: Episodic memory query**
- Tool queries: "What wines has user liked?"
- Expected: Relevant episodic items returned
- Verify: Results are accurate and ranked

**TC-1.2: Semantic memory query**
- Tool queries: "What are Burgundy wine characteristics?"
- Expected: Relevant semantic items returned
- Verify: Results are facts/skills, not events

**TC-1.3: Scratch page query**
- Tool queries: "Are there active alerts?"
- Expected: Active observations returned
- Verify: Only active items included

### Test Suite 2: Access Control

**TC-2.1: Tool can access declared memory types**
- Tool declares: episodic access enabled
- Tool queries: episodic memory
- Expected: Query succeeds
- Verify: Results returned

**TC-2.2: Tool cannot access undeclared memory types**
- Tool declares: episodic access only
- Tool queries: semantic memory
- Expected: Access denied
- Verify: Empty results, no error

### Test Suite 3: Budget Enforcement

**TC-3.1: Query respects budget**
- Budget: 300 tokens
- Query results: 500 tokens worth
- Expected: Results packed within 300 tokens
- Verify: Total tokens ≤ 300

**TC-3.2: Multiple queries share budget**
- Total budget: 500 tokens
- Query 1: 300 tokens
- Query 2: 300 tokens
- Expected: Query 2 gets 200 tokens
- Verify: Total ≤ 500 tokens

### Test Suite 4: Error Handling

**TC-4.1: Query error doesn't block tool**
- Query fails (e.g., timeout)
- Expected: Tool continues
- Verify: Tool execution not blocked

**TC-4.2: Access denied doesn't block tool**
- Tool lacks access to memory type
- Expected: Tool continues
- Verify: Tool execution not blocked

## Next Steps

See related documentation:
- `documentation/tools/si_tools_architecture.md` - Tool architecture
- `documentation/working-memory/overview.md` - Memory system
- `documentation/scratch-page/overview.md` - Scratch page system

