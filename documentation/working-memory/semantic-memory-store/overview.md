# Semantic Memory Store

## Overview

The Semantic Memory Store is a fact and pattern repository that maintains knowledge about the world, the agent, and the user. It uses an upsert-based model with versioning to support updates and corrections, and employs hybrid search (dense + sparse) to enable flexible retrieval of relevant knowledge.

## Purpose

The Semantic Memory Store serves to:

1. **Store Facts and Patterns**: Maintain knowledge about entities, relationships, and patterns
2. **Support Updates**: Allow facts to be corrected or refined over time
3. **Enable Hybrid Search**: Support both semantic and keyword-based retrieval
4. **Manage Capacity**: Use LRU + importance retention for intelligent eviction
5. **Track Lineage**: Maintain version history for audit and learning

## Core Characteristics

### Upsert-Based Model
- **Upsert Operation**: Update if exists, insert if new
- **Key-Based Lookup**: Items are identified by a semantic key provided by Si's reasoning components
- **Versioning**: Each update creates a new version
- **Lineage Tracking**: Previous versions are linked
- **Key Format**: `{entity_type}:{entity_identifier}` (e.g., "location:new_york", "person:alice")

### Soft Delete Support
- **Soft Delete**: Mark as deleted but keep in store
- **Audit Trail**: Deleted items remain for compliance
- **Undelete**: Can restore soft-deleted items
- **Hard Delete**: Optional permanent removal

### LRU + Importance Retention
- **LRU Eviction**: Least recently used items are evicted first
- **Importance Protection**: High-importance items are protected
- **Access Tracking**: Records when items are accessed
- **Importance Scoring**: Items have importance scores

### Hybrid Search
- **Dense Search**: Vector embeddings for semantic similarity
- **Sparse Search**: Keyword/term matching for exact matches
- **Combined Ranking**: Merges dense and sparse scores
- **Metadata Filtering**: Filter by tags, source, etc.

### Versioning and Lineage
- **Version Numbers**: Each update increments version (v1, v2, v3, etc.)
- **Lineage Tracking**: Links to previous versions form an audit trail
- **Update Timestamps**: Records when each version was created
- **Version Queries**: Can retrieve specific versions for historical analysis
- **Version Retention**: All versions are retained in lineage; subject to eviction only when capacity limits are exceeded

## Versioning and Upsert Strategy

### Key Generation and Semantics
Keys are semantic identifiers provided by Si's reasoning components. They represent the entity or concept being stored.

**Key Format**: `{entity_type}:{entity_identifier}`

**Key Examples**:
- `location:new_york` - A fact about New York
- `person:alice` - Information about Alice
- `skill:web_scraping` - A learned skill
- `relationship:alice_knows_bob` - A relationship between entities
- `fact:weather_today` - A time-sensitive fact

**Key Responsibility**: Si's reasoning and learning components are responsible for generating keys. The same key indicates the same entity/concept; different keys indicate different entities/concepts.

### Upsert Semantics
When Si calls `upsert(key, item)`:

**If key exists**:
- The item is updated
- A new version is created (version number incremented)
- The previous version is linked in lineage
- The update timestamp is recorded
- The item's access timestamp is updated

**If key is new**:
- The item is inserted as a new fact
- Version is set to 1
- Lineage is empty (no previous versions)
- Creation timestamp is recorded

### Version Retention and Eviction
- All versions of an item are retained in the lineage chain
- When capacity limits (max_items or max_tokens) are exceeded, eviction occurs
- Eviction considers the entire item (all versions together) as a unit
- If an item is evicted, all its versions are removed
- High-importance items are protected from eviction regardless of version count

### Example Scenario
1. Si learns: "New York is in the United States"
   - Key: `location:new_york`
   - Upsert creates version 1
   - Lineage: empty

2. Si learns: "New York is the largest city in the US"
   - Key: `location:new_york` (same key)
   - Upsert creates version 2
   - Lineage: [1]
   - Version 1 is still accessible via `get_version(item_id, 1)`

3. Si learns: "Alice is a person"
   - Key: `person:alice` (different key)
   - Upsert creates new fact with version 1
   - This is NOT an update to the New York fact

## Operations

### memorize(item, metadata)
Stores or updates a semantic memory item.

**Behavior**:
- If item with same key exists: update (create new version)
- If item is new: insert
- Increments version number
- Records lineage
- Updates access timestamp

**Returns**:
- Confirmation of memorization
- Item ID
- Version number
- Eviction information (if any)

### remember(prompt, metadata)
Retrieves items from the semantic store based on a prompt.

**Behavior**:
- Performs hybrid search (dense + sparse) for relevance to prompt
- Applies filters (tags, source, etc.)
- Returns items ranked by relevance
- Includes version information

**Returns**:
- List of matching items
- Ranked by relevance
- Includes version information

### forget(forget_instructions, mode)
Removes an item from the semantic store based on natural language instructions.

**Supported Instructions**:
- `"key:entity_type:entity_identifier"` - Remove fact by semantic key
- `"oldest"` - Remove least recently used item
- `"least important"` - Remove lowest importance item
- `"before:timestamp"` - Remove items created before timestamp

**Behavior**:
- Interprets natural language instructions
- If mode="soft": Marks item as deleted but retains in store
- If mode="hard": Permanently removes item and all versions
- Updates indexes appropriately

**Returns**:
- Confirmation of forget operation
- Item reference that was forgotten
- Updated store size

### upsert(key, item, metadata)
Explicitly upserts an item by semantic key.

**Parameters**:
- **key**: Semantic identifier in format `{entity_type}:{entity_identifier}`
- **item**: The fact or knowledge to store
- **metadata**: Optional metadata (tags, source, importance, etc.)

**Behavior**:
- If key exists: creates new version, updates lineage
- If key is new: inserts as new fact with version 1
- Records timestamps (creation or update)
- Updates access timestamp
- May trigger eviction if capacity limits exceeded

**Returns**:
- Confirmation of upsert
- Item ID
- Version number
- Eviction information (if any)

### summarize(scope, token_limit)
Generates a summary of semantic memory items.

**Behavior**:
- Summarizes all items or items in scope
- Respects token limit
- Preserves key facts and relationships

**Returns**:
- Summary text
- Token count
- Source items included

### get_capacity_info()
Returns information about semantic store capacity and usage.

**Behavior**:
- Returns current store state

**Returns**:
- Current item count
- Current token usage
- LRU and importance information
- Eviction policy status

### get_version(item_id, version)
Retrieves a specific version of an item.

**Behavior**:
- Retrieves the specified version
- Returns version metadata

**Returns**:
- Item at specified version
- Version metadata

## Design Principles

1. **Upsert-Based**: Updates are natural and efficient
2. **Versioned**: History is maintained
3. **Hybrid Search**: Supports multiple retrieval patterns
4. **Importance-Aware**: Protects important knowledge
5. **Auditable**: All changes are tracked

## Interaction with Other Components

- **Memory Orchestrator**: Calls memorize/remember/forget/summarize/get_capacity_info operations
- **Turn Trace**: Logs all semantic operations
- **Frontal Cortex**: May mark items as important
- **Skill Extraction**: Analyzes semantic memory for patterns

## Configuration

The Semantic Memory Store can be configured with:

- **importance_threshold_high**: Importance at or above this is protected (default: 0.7)
- **importance_threshold_low**: Importance below this is evicted first (default: 0.3)
- **enable_versioning**: Whether to track versions (default: true)
- **enable_soft_delete**: Whether to support soft delete (default: true)

## Example Scenario

1. System learns "New York is in the United States" (version 1)
2. System stores with key="location:new_york"
3. Later, system learns "New York is the largest city in the US" (version 2)
4. Upsert updates the item, creates version 2
5. Lineage links version 2 to version 1
6. User searches "cities in America"
7. Hybrid search finds the item via semantic similarity
8. System can query_version(item_id, 1) to see original fact
9. System can query_version(item_id, 2) to see updated fact
10. LRU eviction removes least-used items when capacity is exceeded

