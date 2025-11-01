# Critical Gap: Scratch Page / Pending Observations System

**Date**: 2025-10-31  
**Status**: MISSING COMPONENT - Needs Complete Specification  
**Priority**: CRITICAL

---

## The Gap

The user's description includes a "scratch page" concept that is **not currently documented anywhere** in Si's architecture:

> "one piece that does seem to be missing is sort of the scratch page of pending items/actions or thoughts that the agent has. Thinking things like 'oh i am waiting for them to confirm they email' or 'I know they are going to bistro vendome and I check the wine list and they will like the 2020 marcel la pierre' or 'there is a pending fraud ring that is targeting people like them so I should be on high alert' and different tools/memories can add to the scratch page"

---

## What This System Should Do

### Purpose
Maintain a dynamic, shared "working notes" space where:
- **Hobgoblins** can record pending thoughts and observations
- **Tools** can add findings and recommendations
- **Memory systems** can surface relevant observations
- **Future turns** can access and act on pending items

### Examples from User Description

1. **Waiting State**: "I am waiting for them to confirm their email"
   - Type: pending_confirmation
   - Owner: agent
   - Status: waiting
   - Related to: user email verification

2. **Contextual Insight**: "They're going to bistro vendome and I checked the wine list - they'll like the 2020 Marcel La Pierre"
   - Type: contextual_insight
   - Owner: agent
   - Status: ready_to_use
   - Related to: upcoming event, wine recommendation

3. **Alert/Risk**: "There is a pending fraud ring targeting people like them - high alert"
   - Type: risk_alert
   - Owner: agent
   - Status: active
   - Related to: security, user profile

---

## How It Differs from Existing Systems

| System | Purpose | Scope | Lifecycle |
|--------|---------|-------|-----------|
| **Frontal Cortex** | Long-horizon goals & pending actions | Durable, cross-turn | Explicit completion |
| **Working Memory** | Current turn context | In-context, current turn | Evicted at turn end |
| **Scratch Page** | Pending observations & thoughts | Semi-durable, cross-turn | Auto-review, manual archival |

**Key Difference**: Scratch Page is for **observations and thoughts** that don't fit the formal structure of goals/actions but are important for future decision-making.

---

## What Needs to Be Specified

### 1. Data Model
- Item structure (type, content, owner, status, timestamps, etc.)
- Item types (pending_confirmation, contextual_insight, risk_alert, etc.)
- Metadata and tagging system
- Lifecycle states

### 2. Operations
- `add_observation(item)` - Add to scratch page
- `list_observations(filters)` - Query scratch page
- `update_observation(id, updates)` - Modify existing item
- `archive_observation(id)` - Move to archive
- `clear_resolved(ids)` - Remove resolved items

### 3. Integration Points
- **Tools**: How tools add observations
- **Hobgoblins**: How hobgoblins add/read observations
- **Memory Systems**: How memories surface relevant observations
- **Router**: How Router uses observations in decision-making
- **Evaluator**: How Evaluator reviews observations for completion

### 4. Policies
- Who can add observations (tools, hobgoblins, memories)
- Who can modify/delete observations
- Retention policies (how long items stay)
- Priority/urgency levels
- Auto-archival rules

### 5. Test Requirements
- Adding observations from different sources
- Querying with various filters
- Lifecycle transitions
- Integration with other systems
- Concurrent access patterns

---

## Why This Matters

Without this system, the agent cannot:
- Track pending confirmations or waiting states
- Store contextual insights for future use
- Maintain active alerts or risk awareness
- Share observations between tools and hobgoblins
- Build up situational awareness across turns

This is a **foundational component** for the agent's ability to maintain context and make informed decisions.

---

## Recommended Approach

1. **Create specification** in `documentation/scratch-page/` with:
   - `overview.md` - Purpose, scope, data model, operations
   - `test-conditions.md` - Detailed test requirements
   - `integration.md` - How it connects to other systems

2. **Define item types** as extensible registry (similar to Frontal Cortex action types)

3. **Specify tool integration** - How tools add observations

4. **Specify memory integration** - How memories surface observations

5. **Specify hobgoblin integration** - How hobgoblins use observations

---

## Questions for Clarification

Before implementing, we should clarify:

1. Should scratch page items be persistent across sessions or just within a conversation?
2. Should there be automatic cleanup/archival of old observations?
3. Should tools be able to modify observations added by other tools?
4. How should priority/urgency be determined for observations?
5. Should observations be searchable by memory systems?
6. What's the maximum size/lifetime of the scratch page?

