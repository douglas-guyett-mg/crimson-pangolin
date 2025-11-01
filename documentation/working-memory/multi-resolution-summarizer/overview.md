# Multi-Resolution Summarizer

## Overview

The Multi-Resolution Summarizer is a system that generates summaries of memory items at different levels of detail and compression. It produces micro-summaries for immediate context injection, meso-summaries for episode-level understanding, and macro-summaries for distilled patterns and skills.

## Purpose

The Multi-Resolution Summarizer serves to:

1. **Compress Information**: Reduce token count while preserving key information
2. **Support Multiple Resolutions**: Generate summaries at different detail levels
3. **Preserve Answerability**: Ensure summaries retain enough information to answer questions
4. **Enable Skill Extraction**: Provide material for extracting reusable skills
5. **Optimize Context**: Generate summaries optimized for context injection

## Summary Resolutions

### Micro-Summaries
- **Target**: 1-3 sentences
- **Token Budget**: 50-100 tokens
- **Use Case**: Inject into working memory for immediate context
- **Content**: Key facts, recent actions, current state
- **Example**: "User asked about weather. System checked forecast. Sunny tomorrow."

### Meso-Summaries
- **Target**: Paragraph-length
- **Token Budget**: 200-500 tokens
- **Use Case**: Summarize episodes or conversations
- **Content**: Sequence of events, key decisions, outcomes
- **Example**: "User asked about weather in New York. System retrieved forecast data. Provided 5-day forecast. User asked about weekend. System highlighted Saturday as best day."

### Macro-Summaries
- **Target**: Distilled patterns and skills
- **Token Budget**: 500-1000 tokens
- **Use Case**: Extract generalizable patterns and skills
- **Content**: Reusable procedures, patterns, lessons learned
- **Example**: "Pattern: When user asks about weather, retrieve forecast data, provide multi-day forecast, highlight best days. Skill: Weather query handling - retrieve data, format response, highlight key days."

## Summarization Process

### Stage 1: Content Selection
Select the most important content to include:
1. Identify key events/facts
2. Filter out redundant information
3. Preserve causal relationships
4. Maintain temporal order

### Stage 2: Abstraction
Abstract content to higher level:
1. Generalize specific details
2. Extract patterns
3. Identify key concepts
4. Remove implementation details

### Stage 3: Compression
Compress content to fit token budget:
1. Remove less important details
2. Combine related information
3. Use concise language
4. Abbreviate where possible

### Stage 4: Validation
Validate summary quality:
1. Check answerability (can questions be answered?)
2. Check completeness (are key points included?)
3. Check coherence (does it read naturally?)
4. Check token count (within budget?)

## Operations

### summarize_micro(items, token_limit)
Generates a micro-summary.

**Parameters**:
- `items`: List of MemoryItem objects to summarize
- `token_limit`: Maximum tokens (default: 100)

**Returns**:
- Summary text
- Actual token count
- Source items included
- Confidence score

### summarize_meso(items, token_limit)
Generates a meso-summary.

**Parameters**:
- `items`: List of MemoryItem objects to summarize
- `token_limit`: Maximum tokens (default: 500)

**Returns**:
- Summary text
- Actual token count
- Source items included
- Confidence score

### summarize_macro(items, token_limit)
Generates a macro-summary.

**Parameters**:
- `items`: List of MemoryItem objects to summarize
- `token_limit`: Maximum tokens (default: 1000)

**Returns**:
- Summary text
- Actual token count
- Source items included
- Extracted patterns/skills
- Confidence score

### validate_summary(summary, original_items)
Validates a summary.

**Parameters**:
- `summary`: Summary text
- `original_items`: Original items

**Returns**:
- Answerability score
- Completeness score
- Coherence score
- Overall quality score

## Design Principles

1. **Multi-Resolution**: Supports multiple detail levels
2. **Token-Aware**: Respects token budgets
3. **Answerability-Focused**: Preserves ability to answer questions
4. **Deterministic**: Same inputs produce same summaries
5. **Auditable**: Tracks which items contributed to summary

## Interaction with Other Components

- **Memory Plugins**: Call summarize operations
- **Memory Orchestrator**: Coordinates summarization
- **Skill Extractor**: Uses macro-summaries for skill extraction
- **Turn Trace**: Logs summarization operations

## Configuration

The Multi-Resolution Summarizer can be configured with:

- **micro_token_limit**: Token limit for micro-summaries (default: 100)
- **meso_token_limit**: Token limit for meso-summaries (default: 500)
- **macro_token_limit**: Token limit for macro-summaries (default: 1000)
- **summarization_model**: Model to use for summarization (default: "abstractive")
- **enable_validation**: Whether to validate summaries (default: true)
- **min_quality_score**: Minimum quality score to accept (default: 0.7)

## Example Scenario

1. Working Memory receives 10 items over 5 turns
2. Items are about to be evicted due to capacity
3. **Micro-Summary Generation**:
   - Select 3 most important items
   - Generate 1-sentence summary: "User asked about weather, system provided forecast."
   - Token count: 12
4. **Meso-Summary Generation**:
   - Select all 10 items
   - Generate paragraph: "User asked about weather in New York. System retrieved 5-day forecast. Provided detailed forecast. User asked about weekend. System highlighted Saturday as best day. User thanked system."
   - Token count: 45
5. **Macro-Summary Generation**:
   - Analyze all 10 items
   - Extract pattern: "Weather query handling"
   - Extract skill: "Retrieve forecast, format response, highlight key days"
   - Generate summary with pattern and skill
   - Token count: 120
6. **Validation**:
   - Check if summary can answer: "What did the user ask about?" - Yes
   - Check if summary can answer: "What was the forecast?" - Partially
   - Quality score: 0.85
7. **Storage**:
   - Micro-summary injected into working memory
   - Meso-summary stored with episode
   - Macro-summary sent to skill extractor

