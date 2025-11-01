# Gap Analysis: User Flow vs Current Documentation

**Date**: 2025-10-31  
**Reviewer**: Augment Agent  
**Purpose**: Identify documentation gaps between the user's description of how the system should work and current Si documentation

---

## User's Described Flow

```
1. User sends message
2. Create WM object that loads relevant information based on current user state and stored WM state
3. Router checks plan or has FC create one, decides: immediate answer, quick tools, or background turns
4. Orchestrator kicks off tool work and background turns
5. When done, evaluates response and sends to user
6. WM stores relevant state and sends to memory daemons
7. Some steps happen in parallel
8. Tools can access existing memories (e.g., wine bottle recommender accessing wine preferences)
9. Missing piece: "scratch page" for pending items/thoughts/observations
```

---

## Critical Gaps Identified

### 🔴 GAP 1: Working Memory Initialization Flow (HIGH PRIORITY)

**What's Missing**: Clear documentation of how WM is initialized at turn start with relevant information loaded from stored state.

**Current State**:
- Memory Orchestrator overview mentions "initialize with configured budget" but doesn't detail the initialization sequence
- No clear description of "load relevant information based on current user state and working memory state"

**What Needs to Be Documented**:
- Step-by-step WM initialization process
- How stored WM state is retrieved and loaded
- How user state influences what gets loaded
- Integration with Consciousness, Frontal Cortex, and Episodic Memory at initialization

**Related Files**: `documentation/working-memory/memory-orchestrator/overview.md`, `documentation/turn/overview.md`

---

### 🔴 GAP 2: Router-Frontal Cortex Integration (HIGH PRIORITY)

**What's Missing**: How Router "checks plan or has FC create one" - the interaction between Router and Frontal Cortex for plan management.

**Current State**:
- Router.md describes urgency assessment and hobgoblin activation
- Frontal Cortex.md describes goals and pending actions
- No documentation of how Router queries FC for existing plans or requests plan creation

**What Needs to Be Documented**:
- Router's decision process for checking existing plans in FC
- How Router requests FC to create a new plan
- What information Router passes to FC for plan creation
- How Router uses FC's goals and pending actions in decision-making

**Related Files**: `documentation/turn/hobgoblins/router.md`, `documentation/frontal-cortex/README.md`

---

### 🔴 GAP 3: Orchestrator Parallel Execution Coordination (HIGH PRIORITY)

**What's Missing**: How the Orchestrator "kicks off tool work and background turns" - the mechanics of parallel execution coordination.

**Current State**:
- Parallelism.md describes parallel execution patterns
- Response-timing.md describes when to respond
- No clear specification of how Orchestrator coordinates parallel tool execution and background turns
- No specification of how Orchestrator decides which tools run in parallel vs sequentially

**What Needs to Be Documented**:
- Orchestrator's role in scheduling parallel tool execution
- How Orchestrator manages background turns
- Synchronization points and coordination mechanisms
- Resource allocation across parallel tasks
- How Orchestrator handles tool dependencies

**Related Files**: `documentation/turn/parallelism.md`, `documentation/turn/response-timing.md`

---

### 🔴 GAP 4: Tool Access to Memory (HIGH PRIORITY)

**What's Missing**: How tools access existing memories to understand context (e.g., wine bottle recommender accessing wine preferences).

**Current State**:
- Tool architecture specifies `memory_snippet` in ToolRequest (optional distilled context)
- No specification of how tools query or access memory
- No specification of what memory access patterns are available to tools
- No specification of memory access permissions/policies

**What Needs to Be Documented**:
- Tool memory access API/interface
- What memory types tools can access
- How tools query memory (search, filter, etc.)
- Memory access permissions and security model
- Examples of tools using memory context

**Related Files**: `documentation/tools/si_tools_architecture.md`, `documentation/working-memory/overview.md`

---

### 🔴 GAP 5: Scratch Page / Pending Observations System (CRITICAL - MISSING COMPONENT)

**What's Missing**: A system for tracking pending items/thoughts/observations that tools and memories can add to.

**Current State**:
- Frontal Cortex tracks goals and pending actions
- Working Memory tracks current context
- No system for "scratch page" of pending observations like:
  - "Waiting for email confirmation"
  - "They're going to bistro vendome - checked wine list, they'll like 2020 Marcel La Pierre"
  - "Fraud ring targeting people like them - high alert"

**What Needs to Be Documented**:
- Purpose and scope of scratch page system
- Data model for pending observations/thoughts
- Who can write to scratch page (tools, memories, hobgoblins)
- How scratch page items are prioritized and surfaced
- Lifecycle of scratch page items (creation, review, resolution, archival)
- Integration with other systems (WM, FC, tools)
- Test requirements for scratch page behavior

**Related Files**: NEW - needs complete specification

---

### 🟡 GAP 6: Memory Daemon Integration (MEDIUM PRIORITY)

**What's Missing**: Clear specification of what "memory daemons" are and how they receive state updates.

**Current State**:
- Memory Orchestrator mentions "memory daemons" in context of state export
- No specification of what daemons are or their responsibilities
- No specification of state export format or protocol

**What Needs to Be Documented**:
- Definition of memory daemons and their role
- What state updates are sent to daemons
- Format and protocol for state export
- How daemons use exported state
- Examples of daemon operations

**Related Files**: `documentation/working-memory/memory-orchestrator/overview.md`

---

### 🟡 GAP 7: Commit Flow and State Export (MEDIUM PRIORITY)

**What's Missing**: Detailed specification of how WM "stores relevant state and sends to memory daemons" at turn end.

**Current State**:
- Memory Orchestrator describes commit() operation at high level
- No detailed specification of what state is exported
- No specification of how state is formatted for export
- No specification of how exported state is used by downstream systems

**What Needs to Be Documented**:
- Detailed commit flow with all steps
- What state is selected for export (criteria)
- Export format and structure
- How exported state is consumed by episodic/semantic memory
- How exported state is consumed by Frontal Cortex
- Error handling and recovery

**Related Files**: `documentation/working-memory/memory-orchestrator/overview.md`, `documentation/working-memory/overview.md`

---

## Summary

| Gap | Priority | Type | Impact |
|-----|----------|------|--------|
| WM Initialization Flow | HIGH | Process | Blocks implementation of turn startup |
| Router-FC Integration | HIGH | Integration | Blocks plan management implementation |
| Orchestrator Parallel Execution | HIGH | Process | Blocks parallel execution implementation |
| Tool Memory Access | HIGH | API | Blocks tool context access implementation |
| Scratch Page System | CRITICAL | Missing Component | Blocks pending observations feature |
| Memory Daemon Integration | MEDIUM | Clarification | Affects state export design |
| Commit Flow & State Export | MEDIUM | Process | Affects turn finalization design |

---

## Recommended Next Steps

1. **IMMEDIATE**: Create specification for Scratch Page / Pending Observations system
2. **HIGH**: Document WM initialization flow with detailed sequence
3. **HIGH**: Document Router-FC integration for plan management
4. **HIGH**: Document Orchestrator parallel execution coordination
5. **HIGH**: Document tool memory access API and patterns
6. **MEDIUM**: Clarify memory daemon integration
7. **MEDIUM**: Detail commit flow and state export process

