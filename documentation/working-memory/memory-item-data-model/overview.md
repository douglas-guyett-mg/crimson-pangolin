# Memory Item Data Model

## Overview

The Memory Item Data Model defines the structure and semantics of individual memory units that flow through the working memory system. Every piece of information stored in any memory type (Working Memory, Episodic, Semantic, Conversation History) is represented as a MemoryItem.

## Purpose

The data model serves to:

1. **Standardize Representation**: Provide a common structure for all memory types to use
2. **Enable Interoperability**: Allow memory items to be moved between stores or aggregated
3. **Support Metadata**: Carry rich metadata about the memory's origin, importance, and characteristics
4. **Enable Serialization**: Support persistence and transmission of memory items

## Core Fields

All MemoryItem instances must include the following fields:

### id
- **Type**: String (UUID or unique identifier)
- **Required**: Yes
- **Description**: Unique identifier for this memory item
- **Semantics**: Used to reference, update, or delete the item

### type
- **Type**: String (enum)
- **Required**: Yes
- **Valid Values**: "conversation", "action", "observation", "goal", "skill", "fact", "pattern", "error", "custom"
- **Description**: Categorizes the kind of memory
- **Semantics**: Helps route items to appropriate memory stores and affects summarization

### text
- **Type**: String
- **Required**: Yes
- **Description**: The actual content of the memory
- **Semantics**: The primary information being stored

### tokens_estimate
- **Type**: Integer
- **Required**: Yes
- **Description**: Estimated token count of the text field
- **Semantics**: Used for budget enforcement and capacity planning

### timestamp
- **Type**: ISO 8601 datetime string
- **Required**: Yes
- **Description**: When this memory was created
- **Semantics**: Used for chronological ordering and TTL calculations

### agent_id
- **Type**: String
- **Required**: Yes
- **Description**: ID of the agent that created this memory
- **Semantics**: Enables multi-agent scenarios and filtering

### tags
- **Type**: List of strings
- **Required**: No (defaults to empty list)
- **Description**: User-defined labels for categorization
- **Semantics**: Enable flexible filtering and grouping

### metadata
- **Type**: Dictionary/Object
- **Required**: No (defaults to empty)
- **Description**: Additional structured data about the memory
- **Semantics**: Extensible field for custom attributes

---

## Store-Specific Fields

Store-specific fields are organized in the `store_metadata` dictionary to maintain modularity and technology-agnosticism. Each memory store can define its own fields without modifying the core MemoryItem structure.

### store_metadata
- **Type**: Dictionary/Object
- **Required**: No (defaults to empty)
- **Description**: Namespace for store-specific fields
- **Semantics**: Allows stores to add fields without coupling to the core data model

**Store-specific field examples:**

**Working Memory Store:**
- `store_metadata.working_memory.position`: Integer - Position in the bounded queue (0 = oldest, n = newest)
- `store_metadata.working_memory.step_index`: Integer - Which step/turn this item was added
- `store_metadata.working_memory.importance_score`: Float (0-1) - Importance for eviction decisions

**Episodic Memory Store:**
- `store_metadata.episodic.parent_id`: String (optional) - ID of parent item if this is a chunk
- `store_metadata.episodic.chunk_index`: Integer (optional) - Position in chunked sequence
- `store_metadata.episodic.episode_id`: String (optional) - Groups related items into episodes

**Semantic Memory Store:**
- `store_metadata.semantic.version`: Integer - Version number for upserts
- `store_metadata.semantic.lineage`: List of strings - IDs of previous versions
- `store_metadata.semantic.last_updated`: ISO 8601 datetime - When this item was last modified
- `store_metadata.semantic.access_count`: Integer - How many times this item has been retrieved

**Conversation History:**
- `store_metadata.conversation_history.speaker`: String - Who said this (e.g., "user", "assistant", "system")
- `store_metadata.conversation_history.turn_number`: Integer - Which turn in the conversation
- `store_metadata.conversation_history.utterance_index`: Integer - Position in the rolling buffer

---

## Metadata Schema

The metadata field is extensible and can contain any key-value pairs. Common metadata keys include:

- **source**: Where the memory came from (e.g., "tool_output", "user_input", "inference")
- **confidence**: Float (0-1) - Confidence in the accuracy of this memory
- **importance**: Float (0-1) - Importance for prioritization
- **novelty**: Float (0-1) - How novel/surprising this information is
- **recency_weight**: Float - Weight for recency-based ranking
- **redaction_level**: String - Sensitivity level ("public", "internal", "sensitive")
- **related_items**: List of item IDs - Cross-references to related memories

---

## Serialization Requirements

MemoryItem instances must support:

1. **JSON Serialization**: Convert to/from JSON for storage and transmission
2. **Binary Serialization**: Optional support for efficient storage
3. **Schema Validation**: Validate that required fields are present and correct types
4. **Version Compatibility**: Handle schema evolution gracefully

---

## Design Principles

1. **Minimal Core**: Only essential fields are required; everything else is optional
2. **Extensible**: Store-specific and custom fields can be added without breaking compatibility
3. **Immutable Identity**: Once created, an item's id and timestamp should not change
4. **Rich Metadata**: Metadata enables flexible querying and prioritization
5. **Type Safety**: All fields have well-defined types and validation rules

## Interaction with Other Components

- **Memory Interfaces**: Create and manipulate MemoryItem instances
- **Memory Orchestrator**: Passes MemoryItem instances between components
- **Budget Manager**: Uses tokens_estimate for budget calculations
- **Serialization Layer**: Converts items to/from storage formats
- **Turn Trace**: Logs MemoryItem operations for debugging

## Example MemoryItem

```
{
  "id": "mem_abc123",
  "type": "observation",
  "text": "User asked about the weather in New York",
  "tokens_estimate": 12,
  "timestamp": "2025-10-30T14:23:45Z",
  "agent_id": "agent_001",
  "tags": ["user_query", "weather"],
  "metadata": {
    "source": "user_input",
    "confidence": 0.95,
    "importance": 0.7,
    "redaction_level": "public"
  },
  "store_metadata": {
    "working_memory": {
      "position": 5,
      "step_index": 42,
      "importance_score": 0.8
    }
  }
}
```

