# Working Memory Store

## Overview

The Working Memory Store is a bounded, short-term memory system that maintains the most recent and important information needed for the current task or interaction. It implements a fixed-capacity queue with intelligent eviction policies to ensure that the most relevant information is always available while respecting memory constraints.

## Purpose

The Working Memory Store serves to:

1. **Maintain Current Context**: Keep the most recent and relevant information immediately accessible
2. **Enforce Capacity Limits**: Prevent unbounded memory growth through size and token budgets
3. **Prioritize Important Items**: Preserve high-importance items even when capacity is exceeded
4. **Support Rapid Access**: Enable fast retrieval of recent information

## Core Characteristics

### Bounded Queue
- **Max Items**: 64 items maximum
- **Max Tokens**: 4,000 tokens maximum
- **FIFO Eviction**: Items are evicted in first-in-first-out order when capacity is exceeded
- **Importance Guard**: High-importance items are protected from eviction

### Capacity Management
The store respects two types of limits:

1. **Item Count Limit**: Maximum 64 items in the queue
2. **Token Budget**: Maximum 4,000 tokens total

When either limit is exceeded, the store evicts items according to the eviction policy.

### Eviction Policy
When capacity is exceeded, the store evicts items in this order:

1. **Low-Importance Items First**: Items with importance_score < 0.3 are evicted first
2. **FIFO Within Priority**: Among items of similar importance, older items are evicted first
3. **Preserve High-Importance**: Items with importance_score >= 0.7 are protected from eviction
4. **Recency Consideration**: Recent items (within last 5 steps) are weighted more heavily

### Time-to-Live (TTL)
Items can be evicted based on age:

1. **Step-Count TTL**: Items older than 20 steps are candidates for eviction
2. **Wall-Time TTL**: Items older than 1 hour are candidates for eviction
3. **Soft Eviction**: TTL-expired items are evicted only when capacity is needed

### Position Tracking
Each item in the queue maintains:

- **position**: Index in the queue (0 = oldest, n = newest)
- **step_index**: Which step/turn the item was added
- **importance_score**: Float (0-1) indicating importance for eviction decisions

## Operations

### memorize(item, metadata)
Adds a new item to the working memory queue.

**Behavior**:
- Appends item to the end of the queue
- Updates position indices for all items
- Checks capacity limits
- Evicts items if necessary
- Returns eviction information

**Returns**:
- Confirmation of memorization
- Item reference (position in queue)
- List of evicted items (if any)
- Current queue size and token count

### remember(prompt, metadata)
Retrieves items from the queue based on a prompt.

**Behavior**:
- Searches all items in the queue for relevance to prompt
- Applies filters (agent_id, tags, etc.)
- Returns items ranked by importance and recency
- Newest/most important items first

**Returns**:
- List of matching items
- Ranked by importance score and recency

### forget(forget_instructions, mode)
Removes an item from the queue based on natural language instructions.

**Supported Instructions**:
- `"oldest"` - Remove the oldest item in the queue
- `"least important"` - Remove the item with lowest importance score
- `"position:N"` - Remove item at position N
- `"before:step_N"` - Remove all items from before step N

**Behavior**:
- Interprets natural language instructions
- If mode="soft": marks as deleted but keeps in queue
- If mode="hard": removes from queue and updates positions
- Updates position indices

**Returns**:
- Confirmation of forget operation
- Item reference that was forgotten
- Updated queue size and token count

### summarize(scope, token_limit)
Generates a summary of queue contents.

**Behavior**:
- Summarizes all items or recent items based on scope
- Respects token limit
- Preserves key information from high-importance items

**Returns**:
- Summary text
- Token count
- Source items included

### get_capacity_info()
Returns information about queue capacity and usage.

**Behavior**:
- Returns current queue state

**Returns**:
- Current item count
- Current token usage
- Max items limit (64)
- Max tokens limit (4000)
- Available space
- Eviction policy information

## Design Principles

1. **Bounded**: Always respects capacity limits
2. **Intelligent Eviction**: Preserves important information
3. **Fast Access**: Optimized for rapid retrieval
4. **Deterministic**: Eviction decisions are reproducible
5. **Auditable**: All evictions are logged

## Interaction with Other Components

- **Memory Orchestrator**: Calls memorize/remember/forget/summarize/get_capacity_info operations
- **Budget Manager**: Enforces token budgets
- **Turn Trace**: Logs all evictions and capacity events
- **Frontal Cortex**: May mark items as high-importance

## Configuration

The Working Memory Store can be configured with:

- **max_items**: Maximum number of items (default: 64)
- **max_tokens**: Maximum token budget (default: 4000)
- **step_ttl**: Items older than this many steps are candidates for eviction (default: 20)
- **wall_ttl**: Items older than this duration are candidates for eviction (default: 1 hour)
- **importance_threshold_low**: Importance below this is evicted first (default: 0.3)
- **importance_threshold_high**: Importance at or above this is protected (default: 0.7)

## Example Scenario

1. User starts a conversation (step 0)
2. Working Memory receives 10 items (total: 500 tokens)
3. User continues conversation (step 5)
4. Working Memory receives 20 more items (total: 1500 tokens)
5. User continues (step 10)
6. Working Memory receives 30 more items (total: 2500 tokens)
7. User continues (step 15)
8. Working Memory receives 25 more items (total: 3500 tokens)
9. User continues (step 20)
10. Working Memory receives 20 more items (would be 4500 tokens)
11. Store evicts low-importance items from steps 0-5 to stay within 4000 token limit
12. Final state: 65 items, 3950 tokens, newest items preserved

