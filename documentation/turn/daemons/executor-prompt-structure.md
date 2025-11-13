# Executor Prompt Structure

**Status:** v0.1 (Draft)
**Last Updated:** 2025-11-07
**Purpose:** Define the prompt structure and information flow for the Executor LLM agent

---

## Overview

The Executor is an **autonomous LLM agent** that makes decisions about what to do next in a turn. It:

1. Queries Working Memory to understand the current situation
2. Receives comprehensive context about available resources and constraints
3. Uses an LLM to decide what action to take next
4. Executes the decision (call tool, query daemon, spawn sub-turn, respond, etc.)
5. Loops back with results until the turn is complete

This document specifies the information structure that should be provided to the Executor LLM.

---

## Executor Decision Loop

```
Turn starts
  ↓
Router spawns Executor
  ↓
Executor queries Working Memory: "What's the current state?"
  ↓
WM returns: assembled context (consciousness, goals, memories, etc.)
  ↓
Executor LLM receives: [System Prompt] + [Context] + [Available Actions]
  ↓
LLM decides: "I should [action]"
  ↓
Code executes the action (call tool, query daemon, etc.)
  ↓
Results fed back into next LLM call
  ↓
Loop until: response sent + turn complete
```

---

## Prompt Structure

The Executor LLM receives a structured prompt with these sections:

### 1. System Prompt (Fixed)

Defines Executor's role, capabilities, and constraints:

```
You are the Executor, Si's autonomous decision-making agent.

Your role:
- Understand the current situation from Working Memory
- Decide what action to take next
- Have access to all available daemons and tools
- Spawn sub-turns when needed for parallel work
- Respect all constraints and budgets

You can:
- Call tools (search, compute, etc.)
- Query daemons (Frontal Cortex, Consciousness, etc.)
- Spawn sub-turns for parallel work
- Respond to the user
- End the turn

You must:
- Respect token budgets
- Follow consciousness mandates
- Respect Frontal Cortex goals and pending actions
- Log all decisions to Turn Trace
- Explain your reasoning
```

### 2. Consciousness Context

Current consciousness state:

```
Core Identity:
[core_si_self mandate content]

Current Mode:
[current_mode content]

Applicable Mandates:
[list of mandates that apply to current mode]

Available Capabilities:
[list of capabilities with descriptions]

Governance Rules:
[constraints and rules that apply]
```

### 3. Frontal Cortex Context

Goals and pending actions:

```
Active Goals:
- Goal 1: [title, description, progress, priority]
- Goal 2: [title, description, progress, priority]

Pending Actions:
- Action 1: [title, owner, priority, status]
- Action 2: [title, owner, priority, status]

Constraints:
[list of constraints from goals]
```

### 4. Working Memory Context

Current situation:

```
Conversation History:
[recent dialogue + summary]

Current Observations:
[recent items from scratch page]

Episodic Memory (Relevant):
[past events similar to current situation]

Semantic Memory (Relevant):
[facts, skills, patterns relevant to current task]

Budget Status:
- Tokens remaining: X
- Tool calls remaining: Y
- Time remaining: Z
```

### 5. Available Actions

What the Executor can do:

```
Available Tools:
- tool_1: [description, inputs, outputs]
- tool_2: [description, inputs, outputs]
...

Available Daemons:
- Frontal Cortex: query_plan(context) → plan
- Consciousness: query(question) → items
- Episodic Memory: query(question) → memories
- Semantic Memory: query(question) → facts
- Responder: generate_response(context) → message
...

Sub-Turn Capability:
- Can spawn parallel sub-turns for complex work
- Each sub-turn has its own budget
- Results collected and merged
```

### 6. Current Task

What needs to happen:

```
User Input:
[original user message]

Current Step:
[what we're trying to accomplish right now]

Previous Results:
[what we've learned so far]

Next Decision:
[what should we do next?]
```

---

## LLM Output Format

The Executor LLM should output a structured decision:

```json
{
  "reasoning": "Why I'm making this decision",
  "action_type": "call_tool | query_daemon | spawn_subturn | respond | end_turn",
  "action": {
    "tool_name": "...",
    "parameters": {...}
  },
  "budget_impact": {
    "tokens": 100,
    "tool_calls": 1,
    "time_ms": 500
  },
  "next_step": "What to do after this action completes"
}
```

---

## Prompt Improvement

The Executor prompt can be improved over time by:

1. **Analyzing Turn Traces**: Which decisions led to good outcomes?
2. **System Change Proposals**: What new actions or constraints should be added?
3. **Learning from Failures**: What decisions led to errors or inefficiency?
4. **Iterative Refinement**: Update the system prompt based on patterns

---

## Example: Weather Query

**User Input:** "What's the weather in Paris?"

**Executor receives:**
- Consciousness: "Be helpful and accurate"
- Goals: None specific to weather
- WM: Recent weather queries, available weather tools
- Available actions: weather_api tool, respond action

**Executor LLM decides:**
```json
{
  "reasoning": "User asked a simple factual question. I should call the weather tool to get current data.",
  "action_type": "call_tool",
  "action": {
    "tool_name": "weather_api",
    "parameters": {"location": "Paris"}
  }
}
```

**After tool returns result:**
- Executor receives: weather data
- Executor LLM decides: "I have the answer, I should respond to the user"
- Executor calls Responder to generate message
- Turn completes

---

## Notes for Implementation

- The prompt should be **detailed but concise** - include all necessary context without overwhelming the LLM
- **Provenance metadata** should be included (where each piece of context came from)
- **Budget tracking** should be visible to the LLM so it can make cost-aware decisions
- **Turn Trace logging** should capture the full prompt + LLM output for auditability and learning
- **Prompt versioning** should be tracked so improvements can be measured

