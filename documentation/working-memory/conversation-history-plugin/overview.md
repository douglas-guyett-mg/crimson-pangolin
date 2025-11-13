# Conversation History Daemon

## Overview

The Conversation History Daemon maintains a rolling buffer of recent utterances (user messages, assistant responses, system messages) and generates summaries of older conversations. It provides a specialized memory type optimized for maintaining conversational context while respecting token budgets. It implements the Daemon Interface (see `documentation/daemon-interface.md`).

## Purpose

The Conversation History Daemon serves to:

1. **Maintain Recent Utterances**: Keep the last N utterances immediately accessible
2. **Summarize Older Conversations**: Compress older utterances into summaries
3. **Support Swappable Backends**: Allow different storage implementations
4. **Optimize for Conversation**: Provide conversation-specific operations
5. **Respect Token Budgets**: Enforce token limits on conversation history

## Core Characteristics

### Rolling Buffer
- **Last N Utterances**: Maintains the most recent N utterances (default: 20)
- **FIFO Eviction**: Oldest utterances are evicted when buffer is full
- **Utterance Grouping**: Groups related messages (e.g., user query + assistant response)
- **Turn Tracking**: Records which turn each utterance belongs to

### Summarization
- **Automatic Summarization**: Older utterances are summarized when evicted
- **Summary Injection**: Summaries are injected into context
- **Incremental Summaries**: New summaries are merged with existing summaries
- **Token-Aware**: Summaries respect token budgets

### Swappable Backends
- **In-Memory Backend**: Fast, ephemeral storage (default)
- **Durable Backend**: Persistent storage (optional)
- **Custom Backends**: Support for custom implementations
- **Transparent Switching**: Backend can be changed without affecting interface

### Speaker Tracking
Each utterance includes:

- **speaker**: Who said it ("user", "assistant", "system")
- **turn_number**: Which turn in the conversation
- **utterance_index**: Position in the rolling buffer
- **timestamp**: When the utterance was created

## Daemon Interface Implementation

The Conversation History Daemon implements the standard Daemon Interface:

### query(natural_language_query, metadata?, data?)
Handles all conversation history operations through natural language queries.

**Query Types**:
- **Store**: `"Add utterance: [text]"` - Appends utterance to buffer
- **Retrieve**: `"Find utterances about [topic]"` - Searches buffer for relevance
- **Remove**: `"Remove oldest utterance"` or `"Remove before turn N"` - Removes utterances
- **Summarize**: `"Summarize conversation"` - Generates summary of buffer

**Behavior**:
- Interprets natural language query to determine operation
- For store operations: Appends utterance, records metadata, checks capacity, evicts if needed
- For retrieve operations: Searches buffer, applies filters, ranks by relevance/recency
- For remove operations: Interprets instructions, removes matching utterances
- For summarize operations: Generates compressed summary within token limits

**Returns**:
- Operation result (confirmation, list of items, summary, etc.)
- Metadata about operation (items affected, tokens used, etc.)

### get_purpose()
Returns: `"Maintain rolling buffer of recent conversation utterances with automatic summarization of older conversations"`

### get_instructions()
Returns guidance on how to use this daemon for conversation context management

## Query Examples

**Store Operation**:
```
query("Add user utterance: 'What's the weather today?'",
      metadata={speaker: "user", turn_number: 5})
```

**Retrieve Operation**:
```
query("Find utterances about weather",
      metadata={speaker: "user"})
```

**Remove Operation**:
```
query("Remove oldest utterance")
```

**Summarize Operation**:
```
query("Summarize conversation",
      data={token_limit: 500})
```

## Capacity Management

The daemon manages buffer capacity automatically:

- **Max Utterances**: 20 (configurable)
- **Max Tokens**: 8000 (configurable)
- **Eviction Policy**: FIFO (oldest utterances evicted first)
- **Summarization**: Evicted utterances are automatically summarized
- **Summary Injection**: Summaries are injected into context for retrieval

## Design Principles

1. **Conversation-Optimized**: Designed specifically for conversational context
2. **Rolling Buffer**: Maintains recent utterances efficiently
3. **Automatic Summarization**: Older utterances are compressed automatically
4. **Swappable Backends**: Storage implementation can be changed
5. **Token-Aware**: Respects token budgets

## Interaction with Other Components

- **Memory Orchestrator**: Calls query() to store/retrieve/remove/summarize utterances
- **Budget Manager**: Enforces token budgets during operations
- **Turn Trace**: Logs all conversation operations
- **Frontal Cortex**: May mark utterances as important

## Configuration

The Conversation History Daemon can be configured with:

- **buffer_size**: Maximum utterances in buffer (configurable)
- **backend**: Storage backend ("in-memory" or "durable", default: "in-memory")
- **summarization_enabled**: Whether to summarize evicted utterances (default: true)
- **eviction_policy**: Policy for managing buffer capacity (e.g., FIFO, importance-based)

**Note**: Specific capacity numbers are technology-dependent and will be determined during implementation based on the chosen storage backend.

## Example Scenario

1. User starts conversation
2. User: "What's the weather?" (utterance 1)
3. Assistant: "I'll check the weather..." (utterance 2)
4. User: "How about tomorrow?" (utterance 3)
5. Assistant: "Tomorrow will be..." (utterance 4)
6. ... (conversation continues for 20 utterances)
7. User: "Tell me about Paris" (utterance 21)
8. Buffer is full, utterances 1-5 are evicted
9. Summary is generated: "User asked about weather today and tomorrow. Assistant provided forecasts."
10. Summary is injected into context
11. Assistant responds to Paris query with full context
12. Later, get_last_n(5) returns utterances 17-21
13. get_summary() returns the generated summary

