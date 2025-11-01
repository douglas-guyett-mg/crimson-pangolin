# Conversation History Plugin

## Overview

The Conversation History Plugin maintains a rolling buffer of recent utterances (user messages, assistant responses, system messages) and generates summaries of older conversations. It provides a specialized memory type optimized for maintaining conversational context while respecting token budgets.

## Purpose

The Conversation History Plugin serves to:

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

## Operations

### memorize(item, metadata)
Adds a new utterance to the conversation history.

**Behavior**:
- Appends utterance to the buffer
- Records speaker, turn_number, timestamp
- Checks buffer capacity
- Evicts oldest utterances if necessary
- Summarizes evicted utterances

**Returns**:
- Confirmation of memorization
- Utterance reference (position in buffer)
- List of evicted utterances (if any)
- Summary information (if generated)

### remember(prompt, metadata)
Retrieves utterances from the conversation history based on a prompt.

**Behavior**:
- Searches all utterances in buffer for relevance to prompt
- Applies filters (speaker, turn_number, etc.)
- Returns utterances ranked by relevance and recency
- Most recent matching utterances first

**Returns**:
- List of matching utterances
- Ranked by relevance and recency

### forget(forget_instructions, mode)
Removes an utterance from the conversation history based on natural language instructions.

**Supported Instructions**:
- `"oldest"` - Remove the oldest utterance
- `"before:turn_N"` - Remove all utterances before turn N
- `"speaker:user"` or `"speaker:assistant"` - Remove utterances from specific speaker
- `"position:N"` - Remove utterance at position N

**Behavior**:
- Interprets natural language instructions
- If mode="soft": marks as deleted but keeps in buffer
- If mode="hard": removes from buffer and updates indexes
- Updates indexes

**Returns**:
- Confirmation of forget operation
- Utterance reference that was forgotten
- Updated buffer size

### summarize(scope, token_limit)
Generates a summary of conversation history.

**Behavior**:
- Summarizes all utterances or recent utterances based on scope
- Respects token limit
- Preserves key information from conversation

**Returns**:
- Summary text
- Token count
- Source utterances included

### get_capacity_info()
Returns information about buffer capacity and usage.

**Behavior**:
- Returns current buffer state

**Returns**:
- Current utterance count
- Current token usage
- Max utterances limit (20)
- Max tokens limit (8000)
- Available space
- Summary information

## Design Principles

1. **Conversation-Optimized**: Designed specifically for conversational context
2. **Rolling Buffer**: Maintains recent utterances efficiently
3. **Automatic Summarization**: Older utterances are compressed automatically
4. **Swappable Backends**: Storage implementation can be changed
5. **Token-Aware**: Respects token budgets

## Interaction with Other Components

- **Memory Orchestrator**: Calls memorize/remember/forget/summarize/get_capacity_info operations
- **Budget Manager**: Enforces token budgets
- **Turn Trace**: Logs all conversation operations
- **Frontal Cortex**: May mark utterances as important

## Configuration

The Conversation History Plugin can be configured with:

- **buffer_size**: Maximum utterances in buffer (default: 20)
- **max_tokens**: Maximum token budget (default: 8000)
- **backend**: Storage backend ("in-memory" or "durable", default: "in-memory")
- **summarization_enabled**: Whether to summarize evicted utterances (default: true)
- **summary_token_limit**: Maximum tokens for summary (default: 1000)

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

