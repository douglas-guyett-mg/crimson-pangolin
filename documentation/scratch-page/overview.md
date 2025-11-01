# Scratch Page System

**Status**: Specification v0.1  
**Last Updated**: 2025-10-31  
**Alignment**: Si Core Engineering Tenants (documentation-as-code, tests-first, modularity, technology-agnosticism)

## Overview

The Scratch Page is a semi-durable, shared working notes system where hobgoblins, tools, and memory systems record pending observations, contextual insights, and alerts that don't fit the formal structure of Frontal Cortex goals and pending actions.

The Scratch Page enables Si to maintain situational awareness across turns by capturing:
- **Pending states**: "Waiting for email confirmation"
- **Contextual insights**: "They're going to bistro vendome - they'll like the 2020 Marcel La Pierre"
- **Risk alerts**: "Fraud ring targeting people like them - high alert"
- **Observations**: Findings from tools and hobgoblins that inform future decisions

## Purpose

The Scratch Page serves to:

1. **Capture Transient Observations**: Record findings that are important but not formal goals/actions
2. **Enable Cross-Turn Awareness**: Make observations available to future turns
3. **Support Tool Integration**: Allow tools to add findings and recommendations
4. **Support Hobgoblin Coordination**: Enable hobgoblins to share pending thoughts
5. **Inform Decision-Making**: Provide context for Router, Executor, and other hobgoblins
6. **Track Pending States**: Maintain awareness of waiting conditions and pending confirmations

## Scope

**In Scope**: Data model, operations, lifecycle, integration points, policies, and test requirements.

**Out of Scope**: Specific storage technology, UI/visualization, or deployment details.

## Key Concepts

### Observation Item

An atomic unit in the Scratch Page containing:
- **id**: Unique identifier (ULID)
- **type**: Classification (pending_confirmation, contextual_insight, risk_alert, observation, etc.)
- **content**: The actual observation text
- **source**: Who created it (tool_name, hobgoblin_name, system)
- **owner**: Who should act on it (agent, user, or specific tool)
- **status**: Current state (active, pending_review, resolved, archived)
- **confidence**: Float 0-1 indicating certainty
- **context**: Related goal_id, task_id, or other contextual references
- **tags**: String array for categorization
- **created_at**: ISO-8601 timestamp
- **updated_at**: ISO-8601 timestamp
- **expires_at**: Optional expiration time
- **metadata**: Arbitrary key-value pairs

### Observation Types

Extensible registry of observation types:

- **pending_confirmation**: Waiting for user/system confirmation
- **contextual_insight**: Relevant finding for future use
- **risk_alert**: Security or safety concern
- **observation**: General finding or note
- **action_suggestion**: Recommended action
- **pattern_detected**: Learned pattern or behavior
- **dependency**: Blocking or dependent condition
- **reminder**: Time-based or event-based reminder

New types can be added via registry without code changes.

### Lifecycle States

- **active**: Currently relevant and being monitored
- **pending_review**: Needs evaluation before use
- **resolved**: Condition has been addressed
- **archived**: No longer relevant, kept for history

## Core Architecture

### Central Coordination
- **Scratch Page Store**: Persistent storage of observations
- **Observation Registry**: Registry of available observation types
- **Lifecycle Manager**: Handles state transitions and expiration

### Integration Points
- **Tools**: Add observations during execution
- **Hobgoblins**: Read/write observations during decision-making
- **Memory Systems**: Surface relevant observations during context assembly
- **Router**: Uses observations in urgency assessment
- **Executor**: Uses observations in tool selection
- **Evaluator**: Reviews observations for completion

## Operations

### add_observation(item: ObservationItem) → ObservationItem
Adds a new observation to the Scratch Page.

**Parameters**:
- `item`: ObservationItem with required fields

**Returns**:
- Created ObservationItem with id and timestamps

**Validation**:
- Type must be registered
- Content must be non-empty
- Source must be valid

### list_observations(filters: ObservationFilter) → ObservationItem[]
Queries observations with optional filtering.

**Parameters**:
- `filters`: Optional filters (type, status, owner, tags, source, etc.)

**Returns**:
- Array of matching observations, ordered by recency

### get_observation(id: string) → ObservationItem
Retrieves a specific observation by id.

**Parameters**:
- `id`: Observation id

**Returns**:
- ObservationItem or null if not found

### update_observation(id: string, updates: Partial<ObservationItem>) → ObservationItem
Updates an existing observation.

**Parameters**:
- `id`: Observation id
- `updates`: Fields to update (status, content, metadata, etc.)

**Returns**:
- Updated ObservationItem

**Validation**:
- Only certain fields can be updated (not id, created_at, source)
- Status transitions must be valid

### archive_observation(id: string) → void
Moves observation to archived state.

**Parameters**:
- `id`: Observation id

**Returns**:
- Confirmation

### clear_resolved(ids: string[]) → void
Removes resolved observations.

**Parameters**:
- `ids`: Array of observation ids to remove

**Returns**:
- Confirmation

### get_active_observations() → ObservationItem[]
Returns all active observations, ordered by priority.

**Returns**:
- Array of active observations

## Design Principles

1. **Lightweight**: Observations are simple, unstructured notes
2. **Extensible**: New observation types can be added without code changes
3. **Transparent**: All operations are logged for auditability
4. **Deterministic**: Same inputs produce same outputs
5. **Modular**: Scratch Page is independent of other systems
6. **Accessible**: All hobgoblins and tools can read/write

## Interaction with Other Systems

- **Tools**: Add observations during execution; read observations for context
- **Hobgoblins**: Read observations during decision-making; write pending thoughts
- **Memory Systems**: Surface relevant observations during context assembly
- **Router**: Uses observations in urgency assessment
- **Executor**: Uses observations in tool selection and planning
- **Evaluator**: Reviews observations for completion and resolution
- **Frontal Cortex**: Observations can reference FC goals and pending actions

## Configuration

The Scratch Page can be configured with:

- **max_observations**: Maximum number of active observations (default: 100)
- **default_expiration**: Default TTL for observations (default: 7 days)
- **auto_archive_resolved**: Whether to auto-archive resolved items (default: true)
- **observation_types**: Registry of available observation types

## Example Scenarios

### Scenario 1: Pending Confirmation
```
Tool: email_verification
  → add_observation({
      type: "pending_confirmation",
      content: "Waiting for user to confirm email address",
      owner: "user",
      status: "active",
      context: { task_id: "verify_email_123" }
    })

Future turn:
  → Router checks observations
  → Sees pending confirmation
  → Prioritizes follow-up
```

### Scenario 2: Contextual Insight
```
Tool: wine_recommender
  → add_observation({
      type: "contextual_insight",
      content: "User going to bistro vendome - they'll like 2020 Marcel La Pierre",
      owner: "agent",
      confidence: 0.95,
      context: { event: "bistro_vendome_visit" }
    })

Future turn:
  → Memory system surfaces observation
  → Agent uses insight in recommendation
```

### Scenario 3: Risk Alert
```
Tool: fraud_detector
  → add_observation({
      type: "risk_alert",
      content: "Fraud ring targeting users in this demographic - high alert",
      owner: "agent",
      status: "active",
      tags: ["security", "fraud"]
    })

Future turns:
  → Router sees alert
  → Increases scrutiny on financial transactions
  → Executor prioritizes verification steps
```

## Next Steps

See related documentation:
- `test-conditions.md` - Detailed test requirements
- `integration.md` - Integration with other systems

