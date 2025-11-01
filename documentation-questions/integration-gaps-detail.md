# Integration Gaps: Detailed Analysis

**Date**: 2025-10-31  
**Purpose**: Identify specific integration points that lack documentation

---

## 1. WM Initialization → Router Decision Flow

### Current State
- Memory Orchestrator initializes with budget
- Router receives input and assesses urgency
- **Gap**: No documented sequence showing how WM initialization feeds into Router's decision

### What's Missing
```
Turn Start
  ↓
WM Initialization:
  - Load consciousness (mandates, values, governance)
  - Load Frontal Cortex (active goals, pending actions)
  - Load conversation history
  - Load relevant episodic memory
  - Load relevant semantic memory
  ↓
Router receives initialized WM context
  ↓
Router makes decision using:
  - Consciousness from WM
  - Goals/actions from WM
  - History from WM
  - Input signal
```

### Documentation Needed
- Sequence diagram of initialization
- What each memory type contributes
- How Router accesses initialized context
- Error handling if initialization fails

---

## 2. Router → Frontal Cortex Plan Checking

### Current State
- Router decides urgency and hobgoblin activation
- Frontal Cortex maintains goals and pending actions
- **Gap**: No documented interaction for plan checking/creation

### What's Missing
```
Router Decision Point:
  "Do I need a plan?"
    ↓
  Query FC: "Do I have an active plan for this goal?"
    ↓
  FC Response: Plan exists OR Plan doesn't exist
    ↓
  If exists: Use existing plan
  If not: Request FC create new plan
    ↓
  Router uses plan in hobgoblin activation
```

### Documentation Needed
- Router's criteria for checking FC
- FC query interface for plan lookup
- Plan creation request format
- How Router uses returned plan
- Fallback if plan creation fails

---

## 3. Executor → Orchestrator Parallel Coordination

### Current State
- Executor orchestrates tool execution
- Orchestrator manages memory operations
- **Gap**: No documented coordination for parallel tool execution

### What's Missing
```
Executor receives plan with multiple tools
  ↓
Executor analyzes dependencies:
  - Tool A: independent
  - Tool B: depends on Tool A
  - Tool C: independent
  ↓
Executor requests Orchestrator:
  "Run A and C in parallel, then B"
  ↓
Orchestrator coordinates:
  - Allocates resources
  - Manages tool execution
  - Handles failures
  - Tracks results
  ↓
Executor receives results
```

### Documentation Needed
- Executor-Orchestrator interface for parallel execution
- Dependency analysis algorithm
- Resource allocation strategy
- Failure handling and retry logic
- Result aggregation

---

## 4. Tool → Memory Access Pattern

### Current State
- Tools receive `memory_snippet` in ToolRequest
- Tools can include `memory_writes` in ToolResponse
- **Gap**: No documented API for tools to query memory

### What's Missing
```
Tool Execution:
  ↓
Tool needs context: "What wines does user like?"
  ↓
Tool queries Memory:
  - Search episodic memory for wine preferences
  - Search semantic memory for wine facts
  - Get results
  ↓
Tool uses results in execution
  ↓
Tool adds findings to memory_writes
```

### Documentation Needed
- Tool memory query API
- Query syntax/format
- What memory types are queryable
- Access control/permissions
- Response format
- Error handling

---

## 5. Tool/Hobgoblin → Scratch Page Integration

### Current State
- Scratch page doesn't exist
- **Gap**: Complete missing component

### What's Missing
```
Tool Execution:
  ↓
Tool discovers insight: "User will like wine X"
  ↓
Tool adds to Scratch Page:
  {
    type: "contextual_insight",
    content: "User will like 2020 Marcel La Pierre",
    context: "bistro_vendome_visit",
    confidence: 0.95
  }
  ↓
Future turns can access this observation
```

### Documentation Needed
- Scratch page API for adding items
- Item type registry
- Integration with tool execution
- Integration with hobgoblin decision-making
- Integration with memory systems

---

## 6. Memory Daemon State Export

### Current State
- Memory Orchestrator commits state
- Exports go to episodic/semantic memory and FC
- **Gap**: No documented "memory daemon" concept or state export protocol

### What's Missing
```
Turn End:
  ↓
Orchestrator.commit():
  ↓
Prepare state exports:
  - Episodic memory writes
  - Semantic memory writes
  - FC progress updates
  - Scratch page updates
  ↓
Send to "memory daemons":
  - What are daemons?
  - What format?
  - What protocol?
  ↓
Daemons process state:
  - Store in persistent memory
  - Update indices
  - Trigger learning
```

### Documentation Needed
- Definition of "memory daemon"
- State export format
- Export protocol/API
- What each daemon does with state
- Error handling and recovery

---

## 7. Evaluator → Response Timing Coordination

### Current State
- Evaluator assesses completion
- Response timing is urgency-driven
- **Gap**: No documented interaction between Evaluator and response timing

### What's Missing
```
Evaluator checks:
  - Is work complete?
  - Are results satisfactory?
  - Any errors or issues?
  ↓
Evaluator signals to Responder:
  - "Ready to respond" OR
  - "Need more work" OR
  - "Partial results available"
  ↓
Responder decides:
  - Send response now (based on urgency)
  - Wait for more results
  - Send partial response
```

### Documentation Needed
- Evaluator completion criteria
- Evaluator-Responder communication protocol
- How urgency level affects Evaluator decisions
- Partial completion handling

---

## Summary Table

| Gap | From | To | Type | Priority |
|-----|------|----|----|----------|
| Initialization sequence | WM | Router | Process | HIGH |
| Plan checking/creation | Router | FC | API | HIGH |
| Parallel execution | Executor | Orchestrator | Coordination | HIGH |
| Memory query API | Tools | Memory | API | HIGH |
| Scratch page integration | Tools/Hobgoblins | Scratch Page | API | CRITICAL |
| State export protocol | Orchestrator | Daemons | Protocol | MEDIUM |
| Completion signaling | Evaluator | Responder | Protocol | MEDIUM |

