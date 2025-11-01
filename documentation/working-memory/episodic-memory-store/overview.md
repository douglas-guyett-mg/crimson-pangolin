# Episodic Memory Store — Append-Only with Soft-Delete (v1)

## Overview

The Episodic Memory Store is an append-only log that records experiences, events, and interactions in chronological order. This is **Version 1** of the episodic memory implementation, characterized by:
- Append-only semantics (items cannot be modified after creation)
- Soft-delete support for normal operations (items marked as deleted but retained)
- Hard-delete capability for compliance requirements (e.g., GDPR right to be forgotten)
- Chronological ordering and queryable history

It provides a durable, queryable history of what has happened, enabling the system to learn from past experiences and retrieve relevant historical context.

**Note**: Other variants of episodic memory may implement different forgetting mechanisms (e.g., time-decay, importance-based pruning, or active forgetting). This document describes the append-only variant with soft-delete semantics.

## Purpose

The Episodic Memory Store serves to:

1. **Record Experiences**: Maintain a complete, chronological log of events and interactions
2. **Support Historical Queries**: Enable retrieval of events within time ranges or by episode
3. **Enable Learning**: Provide raw material for skill extraction and pattern discovery
4. **Maintain Audit Trail**: Create an auditable record of all significant events

## Core Characteristics

### Append-Only Log
- **Immutable History**: Once written, items cannot be modified
- **Chronological Order**: Items are stored in the order they occurred
- **Timestamped**: Every item includes precise timestamp
- **Indexed**: Multiple indexes enable efficient querying
- **Deletion Semantics**:
  - **Soft Delete (default)**: Items marked as deleted but retained in log for audit trail
  - **Hard Delete (compliance exception)**: Items permanently removed for legal/compliance requirements (e.g., GDPR)

### Chunking for Large Entries
Large events (e.g., long tool outputs) are automatically chunked:

- **Chunk Size**: Configurable, typically 2000 tokens per chunk
- **Chunk Linking**: Chunks are linked via parent_id and chunk_index
- **Semantic Preservation**: Chunking preserves semantic meaning
- **Transparent Retrieval**: Chunks are reassembled when retrieved

### Episode Grouping
Related events can be grouped into episodes:

- **Episode ID**: Groups related items together
- **Episode Boundaries**: Episodes can span multiple turns or steps
- **Episode Queries**: Retrieve all items in an episode
- **Episode Summaries**: Generate summaries per episode

### Deletion Semantics
This version supports two deletion modes:

**Soft Delete (Default)**:
- Marks item as deleted but retains it in the log
- Soft-deleted items are excluded from normal queries and context assembly
- Maintains audit trail for compliance and debugging
- Reversible (can be undeleted if needed)
- Used for normal operational deletions

**Hard Delete (Compliance Exception)**:
- Permanently removes item from the log
- Used only for legal/compliance requirements (e.g., GDPR right to be forgotten)
- Irreversible
- Should be logged and audited separately
- Not the default behavior

## Operations

### memorize(item, metadata)
Appends a new item to the episodic log.

**Behavior**:
- Appends item to the end of the log
- Assigns sequential ID if not provided
- Chunks large items automatically
- Records timestamp

**Returns**:
- Confirmation of memorization
- Item ID (or chunk IDs if chunked)
- Chunk information if applicable

### remember(prompt, metadata)
Retrieves items from the episodic log based on a prompt.

**Behavior**:
- Searches all items in the log for relevance to prompt
- Applies filters (time range, episode_id, tags, etc.)
- Returns items ranked by relevance and recency
- Reassembles chunks if needed

**Returns**:
- List of matching items
- Ranked by relevance and recency
- Chunks reassembled

### forget(forget_instructions, mode)
Removes an item from the episodic log based on natural language instructions.

**Supported Instructions**:
- `"memory id:N"` - Remove item with ID N
- `"before:timestamp"` - Remove all items before timestamp
- `"episode:episode_id"` - Remove all items in episode
- `"oldest"` - Remove oldest item

**Behavior**:
- Interprets natural language instructions
- If mode="soft": Marks item as deleted but retains in log for audit trail. Item is excluded from queries but remains for compliance.
- If mode="hard": Permanently removes item from log. Used only for compliance requirements. Should be logged separately.
- Updates indexes appropriately

**Returns**:
- Confirmation of forget operation
- Item reference that was forgotten
- Updated log size

### summarize(scope, token_limit)
Generates a summary of episodic items.

**Behavior**:
- Summarizes all items or items in scope
- Respects token limit
- Preserves key events

**Returns**:
- Summary text
- Token count
- Source items included

### get_capacity_info()
Returns information about episodic log capacity and usage.

**Behavior**:
- Returns current log state

**Returns**:
- Current item count
- Current token usage
- Total log size
- Retention information
- Soft-delete vs. hard-delete counts

**Important**: Hard delete should only be used for legal/compliance requirements and must be audited separately from soft deletes.

## Design Principles

1. **Append-Only**: History is immutable (items cannot be modified after creation)
2. **Chronological**: Items are ordered by time
3. **Queryable**: Multiple query patterns supported
4. **Chunked**: Large items are handled transparently
5. **Auditable**: All operations are logged
6. **Compliance-Aware**: Supports both soft-delete (default) and hard-delete (compliance exception)
7. **Variant-Aware**: This is v1 (append-only with soft-delete); other variants may implement different forgetting mechanisms

## Interaction with Other Components

- **Memory Orchestrator**: Calls memorize/remember/forget/summarize/get_capacity_info operations
- **Skill Extractor**: Queries episodes to extract reusable skills
- **Turn Trace**: Logs all episodic operations
- **Frontal Cortex**: May mark episodes as important

## Configuration

The Episodic Memory Store can be configured with:

- **chunk_size**: Maximum tokens per chunk (default: 2000)
- **max_log_size**: Maximum total items in log (default: unlimited)
- **retention_period**: How long to keep items (default: unlimited)
- **enable_soft_delete**: Whether to support soft delete (default: true)

## Example Scenario

1. User starts conversation (timestamp: 10:00)
2. System receives user query, writes to episodic log
3. System calls tool, receives large output (5000 tokens)
4. Output is chunked into 3 chunks, all linked with parent_id
5. System generates response, writes to episodic log
6. User provides feedback, writes to episodic log
7. All items are grouped under episode_id="ep_001"
8. Later, query_episode("ep_001") retrieves all 6 items in order
9. Skill extractor analyzes episode to extract reusable patterns
10. System can query_range(10:00, 10:30) to get all items in that time window

