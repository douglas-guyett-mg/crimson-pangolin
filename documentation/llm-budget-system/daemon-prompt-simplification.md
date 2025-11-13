# Daemon-Specific Prompt Simplification

**Status**: Specification v1.0  
**Last Updated**: 2025-11-04  
**Alignment**: Si Core Engineering Tenants (documentation-as-code, tests-first, modularity, technology-agnosticism)

## Overview

When a daemon's LLM call prompt exceeds its budget, the daemon applies its own simplification strategy to reduce the prompt size while preserving essential information. Each daemon knows best how to simplify its own prompts based on its specific purpose and decision-making needs.

## Purpose

Daemon-Specific Prompt Simplification serves to:

1. **Preserve Daemon Autonomy**: Each daemon decides how to simplify its own prompts
2. **Maintain Decision Quality**: Simplification preserves information critical to the daemon's decisions
3. **Enable Budget Compliance**: Reduce prompt size to fit within budget constraints
4. **Support Graceful Degradation**: Degrade quality rather than fail completely
5. **Enable Learning**: Track which simplifications work best for each daemon

## Scope

**In Scope**: Daemon-specific simplification strategies, prompt reduction techniques, quality preservation

**Out of Scope**: Global prompt compression algorithms, storage optimization, memory management

## Key Concepts

### Simplification Strategy
A daemon-specific algorithm for reducing prompt size while preserving essential information. Each daemon implements its own strategy based on its purpose.

### Simplification Levels
Daemons may implement multiple levels of simplification, progressively reducing prompt size:

- **Level 1 (Light)**: Remove verbose explanations, keep all facts
- **Level 2 (Medium)**: Summarize context, keep critical facts
- **Level 3 (Heavy)**: Keep only essential facts, minimal context

### Essential Information
Information that is critical to the daemon's decision-making and should be preserved during simplification.

## Daemon Simplification Strategies

### Router Daemon
**Purpose**: Assess urgency and activate appropriate daemons

**Essential Information**:
- User input (the actual message/request)
- Urgency signals (keywords, patterns)
- Current mode/context
- Recent similar situations

**Simplification Strategy**:
1. Remove verbose context history
2. Keep only recent 3-5 turns
3. Summarize user preferences to key facts
4. Preserve urgency keywords and patterns

### Executor Daemon
**Purpose**: Orchestrate tool execution and lifecycle state management

**Essential Information**:
- Task description and goals
- Constraints and mandates
- Available tools and skills
- Recent successful executions

**Simplification Strategy**:
1. Summarize tool descriptions to essential capabilities
2. Keep only relevant constraints
3. Remove detailed historical context
4. Preserve goal and success criteria

### Executor Daemon
**Purpose**: Execute plans and handle tool invocations

**Essential Information**:
- Current plan step
- Tool specifications
- Recent tool results
- Error context

**Simplification Strategy**:
1. Remove historical plan steps
2. Keep only current step and next 2 steps
3. Summarize tool results to key outcomes
4. Preserve error messages and constraints

### Evaluator Daemon
**Purpose**: Assess whether turn achieved its goals

**Essential Information**:
- Expected outcome
- Actual outcome
- User feedback
- Success criteria

**Simplification Strategy**:
1. Remove detailed execution trace
2. Keep only key decisions and outcomes
3. Summarize context to relevant facts
4. Preserve success criteria and user feedback

### Reflector Daemon
**Purpose**: Learn from turn outcomes and update consciousness

**Essential Information**:
- What was attempted
- What was the outcome
- Why it succeeded or failed
- Patterns and generalizations

**Simplification Strategy**:
1. Summarize detailed traces to key events
2. Keep only relevant context
3. Remove redundant information
4. Preserve outcome and reasoning

## Implementation Pattern

Each daemon implements a `simplify_prompt()` method:

```
daemon.simplify_prompt(
  prompt: string,
  current_size_tokens: integer,
  target_size_tokens: integer,
  simplification_level: 1|2|3
) -> simplified_prompt: string
```

**Parameters**:
- `prompt`: The full prompt to simplify
- `current_size_tokens`: Current token count
- `target_size_tokens`: Target token count to achieve
- `simplification_level`: How aggressively to simplify (1=light, 3=heavy)

**Returns**:
- `simplified_prompt`: Reduced prompt that fits within target

## Design Principles

1. **Daemon Autonomy**: Each daemon controls its own simplification
2. **Quality Preservation**: Simplification preserves decision-critical information
3. **Progressive Degradation**: Multiple simplification levels allow graceful degradation
4. **Auditability**: All simplifications logged for learning
5. **Determinism**: Same prompt + level produces same simplification

## Interaction with Other Systems

- **LLM Budget System**: Triggered when prompt exceeds daemon budget
- **Turn Trace**: All simplifications logged with rationale
- **Consciousness**: May inform what information is essential
- **Learning Loop**: Tracks which simplifications improve outcomes

## Configuration

Each daemon can be configured with:

- **simplification_enabled**: Whether to simplify when over budget (default: true)
- **max_simplification_level**: Maximum aggressiveness (default: 3)
- **preserve_keywords**: Keywords that must be preserved (daemon-specific)
- **log_simplifications**: Whether to log all simplifications (default: true)

## Example Scenario

1. Router daemon prepares prompt: 2500 tokens, budget=2000
2. Calls `simplify_prompt(prompt, 2500, 2000, level=1)`
3. Router removes verbose context history, keeps recent turns
4. Result: 1900 tokens → Within budget → Proceeds with LLM call
5. Simplification logged: "Removed context history, preserved urgency signals"

