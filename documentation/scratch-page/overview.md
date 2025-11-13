# Scratch Page System

**Status:** v0.1 (Draft)
**Last Updated:** 2025-11-07
**Alignment:** Si Core Engineering Tenants (documentation-as-code, tests-first, modularity, technology-agnosticism)

## Overview

The Scratch Page is a semi-durable, shared working notes system where daemons, tools, and memory systems record stories and vignettes—observations, contextual insights, and alerts that may or may not result in Frontal Cortex actions and plans.

The Scratch Page captures the "what we've learned" stories that inform decision-making:
- **Pending states**: "Waiting for email confirmation"
- **Contextual insights**: "They're going to bistro vendome - they'll like the 2020 Marcel La Pierre"
- **Risk alerts**: "Fraud ring targeting people like them - high alert"
- **Observations**: Findings from tools and daemons that inform future decisions

These stories may lead to Frontal Cortex actions (e.g., "Send wine recommendation"), or they may remain as contextual awareness without formal commitment.

## Purpose

The Scratch Page serves to:

1. **Capture Stories and Vignettes**: Record observations, insights, and alerts as stories that may or may not lead to formal actions
2. **Enable Cross-Turn Awareness**: Make observations available to future turns for context and decision-making
3. **Support Tool Integration**: Allow tools to add findings and recommendations without formal commitment
4. **Support Daemon Coordination**: Enable daemons to share pending thoughts and contextual awareness
5. **Inform Decision-Making**: Provide context for Router, Executor, and other daemons to decide what becomes a Frontal Cortex action
6. **Track Pending States**: Maintain awareness of waiting conditions and pending confirmations that may or may not require formal action

## Relationship to Frontal Cortex

**Scratch Page** and **Frontal Cortex** serve complementary purposes:

- **Scratch Page**: "What we've learned/observed" — Stories and vignettes that capture contextual awareness
- **Frontal Cortex**: "What we're committing to do" — Formal goals and actionable next steps

**Flow**:
1. Tools/daemons observe something → Add story to Scratch Page
2. Router/Executor review Scratch Page stories
3. If story warrants action → Create Frontal Cortex action/goal
4. If not → Story remains in Scratch, expires, or gets archived

**Key Distinction**:
- Scratch Page stories are **low-commitment** observations that inform decision-making
- Frontal Cortex actions are **high-commitment** formal plans with ownership and accountability
- A story may never become an action (it just provides context)
- An action should be traceable back to the story that motivated it

## Scope

**In Scope**: Data model, operations, lifecycle, integration points, policies, and test requirements.

**Out of Scope**: Specific storage technology, UI/visualization, or deployment details.

## Key Concepts

### Observation Item

An atomic unit in the Scratch Page containing:
- **id**: Unique identifier (ULID)
- **type**: Classification (pending_confirmation, contextual_insight, risk_alert, observation, etc.)
- **content**: The actual observation text
- **source**: Who created it (tool_name, daemon_name, system)
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
- **Daemons**: Read/write observations during decision-making
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
6. **Accessible**: All daemons and tools can read/write

## Interaction with Other Systems

- **Tools**: Add observations during execution; read observations for context
- **Daemons**: Read observations during decision-making; write pending thoughts
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

