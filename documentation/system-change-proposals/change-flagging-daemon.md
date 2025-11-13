# Change Flagging Daemon

**Purpose:** Monitor turn execution and flag changes needed when SI detects system improvements are required.

---

## Overview

The Change Flagging Daemon runs during turn execution and monitors for situations where SI needs system improvements. When detected, it creates a flag in the turn trace. These flags are later processed by the System Analyzer Daemon to generate proposals.

---

## Daemon Interface

The Change Flagging Daemon implements the Daemon Interface:

```
get_purpose() -> string
  Returns: "Monitor turn execution and flag system changes needed"

get_instructions() -> string
  Returns: Specifications for what to flag and how

query(prompt: string, metadata: object) -> Future<Result>
  Enables other daemons to query for flagging information
```

---

## Core Responsibilities

### 1. Monitor Daemon Suggestions
**What:** Listen for suggestions from other daemons (especially Frontal Cortex)  
**When:** During turn execution  
**How:** Other daemons call `flag_change_needed()` method

### 2. Create Flags
**What:** Create a flag in the turn trace  
**When:** A daemon suggests a change is needed  
**How:** Add to `turn_trace.system_change_flags` array

### 3. Collect Context
**What:** Gather context about why the change is needed  
**When:** When creating a flag  
**How:** Include turn_id, user_id, action_attempted, error_message, rationale

---

## Initial Mandate (v1)

**Scope:** Capture explicit suggestions from daemons  
**Focus:** Frontal Cortex action type suggestions, other daemon tool/capability suggestions  
**Approach:** Reactive - flag when daemons explicitly suggest changes

```
1. Monitor Frontal Cortex for new action type suggestions
   - When FC suggests a new action type that doesn't exist
   - Create flag with type: "new_action_type"
   - Include FC's suggested policies and description

2. Monitor other daemons for tool/capability suggestions
   - When any daemon suggests a missing tool
   - Create flag with type: "new_tool"
   - Include tool name, purpose, suggested inputs/outputs

3. Monitor for tool enhancement suggestions
   - When a daemon suggests enhancing an existing tool
   - Create flag with type: "tool_enhancement"
   - Include tool name, enhancement description, reason

4. Collect context for all flags
   - Turn ID, user ID, conversation ID
   - What action was being attempted
   - Error messages (if applicable)
   - Rationale from the suggesting daemon
```

---

## Future Enhancements (v2+)

As we learn what makes sense, the daemon can expand to:

1. **Proactive Analysis**
   - Run LLM analysis on turn traces
   - Identify patterns (e.g., "user asked for X three times")
   - Generate proactive suggestions

2. **Pattern Detection**
   - Detect repeated failures (e.g., "tool not found" 5 times)
   - Suggest new tools based on patterns
   - Identify capability gaps

3. **Performance Analysis**
   - Detect slow operations that could be optimized
   - Suggest new action types for common patterns
   - Identify bottlenecks

---

## API for Other Daemons

Other daemons call this method to flag changes:

```
flag_change_needed(
  flag_type: string,              // "new_action_type", "new_tool", etc.
  details: object,                // Type-specific details
  severity: string,               // "info", "warning", "critical"
  rationale: string               // Why this change is needed
) -> void
```

### Example: Frontal Cortex Suggests New Action Type

```
// In Frontal Cortex daemon
change_flagging_daemon.flag_change_needed(
  flag_type: "new_action_type",
  details: {
    action_type_name: "parallel_task",
    description: "Execute multiple tasks in parallel",
    reason: "User asked to run 5 research tasks simultaneously",
    suggested_policies: {
      requires_confirmation: false,
      auto_complete: true
    }
  },
  severity: "info",
  rationale: "Current action types don't support parallel execution"
)
```

---

## Storage

Flags are stored in the turn trace:

```
turn_trace: {
  turn_id: "...",
  stimulus: { ... },
  lifecycle_states: [ ... ],
  system_change_flags: [
    {
      flag_id: "flag_20251104_001",
      flag_type: "new_action_type",
      ...
    }
  ],
  response: { ... }
}
```

---

## Behavior

### During Turn Execution
1. Change Flagging Daemon is initialized at turn start
2. Other daemons call `flag_change_needed()` when they detect changes needed
3. Flags are accumulated in `turn_trace.system_change_flags`
4. At turn end, flags are persisted with the turn trace

### After Turn Execution
1. System Analyzer Daemon reads turn traces
2. Processes flagged changes
3. Generates proposals for human review

---

## Test Conditions

See `documentation/system-change-proposals/test-conditions.md` for detailed test conditions.

Key tests:
1. Flags are created when daemons suggest changes
2. Flags include all required context
3. Flags are stored in turn trace
4. Multiple flags can be created in a single turn
5. Flags persist across turn boundaries

---

## Next Steps

1. Specify System Analyzer Daemon (processes flags, generates proposals)
2. Define test conditions for Change Flagging Daemon
3. Define test conditions for System Analyzer Daemon

