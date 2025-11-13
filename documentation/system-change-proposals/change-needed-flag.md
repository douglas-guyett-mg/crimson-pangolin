# Change Needed Flag

**Purpose:** Define the structure of flags placed in turn traces when SI detects that a system change is needed.

---

## Overview

During turn execution, the Change Flagging Daemon monitors for situations where SI needs system improvements. When detected, a flag is added to the turn trace. These flags are later processed by the System Analyzer Daemon to generate proposals.

---

## Flag Structure

```
{
  flag_id: string,                    // Unique identifier for this flag
  flag_type: string,                  // Type of change needed (see Flag Types below)
  severity: "info" | "warning" | "critical",
  source_daemon: string,              // Which daemon flagged this (e.g., "frontal_cortex", "executor")
  timestamp: ISO8601,                 // When flag was created
  
  // Core details - varies by flag_type
  details: {
    // Specific to flag_type (see examples below)
  },
  
  // Context for understanding why this change is needed
  context: {
    turn_id: string,
    user_id?: string,
    conversation_id?: string,
    action_attempted?: string,        // What was SI trying to do when it needed this?
    error_message?: string,           // If applicable
    rationale?: string                // Why is this change needed?
  },
  
  // Optional: suggested documentation structure
  suggested_documentation?: {
    // Hints for what documentation should look like
    // (filled in by Change Flagging Daemon if it has suggestions)
  }
}
```

---

## Flag Types

### 1. `new_action_type`
**Source:** Frontal Cortex daemon  
**When:** FC suggests a new action type that doesn't exist in the registry

```
details: {
  action_type_name: string,           // e.g., "parallel_task"
  description: string,                // What this action type does
  reason: string,                     // Why it's needed
  suggested_policies?: {
    requires_confirmation?: boolean,
    auto_complete?: boolean,
    blocking?: boolean
  }
}
```

### 2. `new_tool`
**Source:** Any daemon  
**When:** A daemon needs a tool that doesn't exist

```
details: {
  tool_name: string,                  // e.g., "pdf_analyzer"
  tool_purpose: string,               // What it should do
  use_case: string,                   // Why it's needed
  suggested_inputs?: string[],        // What parameters it should accept
  suggested_outputs?: string[]        // What it should return
}
```

### 3. `tool_enhancement`
**Source:** Any daemon  
**When:** An existing tool needs new functionality

```
details: {
  tool_name: string,                  // Which tool to enhance
  enhancement: string,                // What capability to add
  reason: string,                     // Why it's needed
  current_limitation?: string         // What's missing now
}
```

### 4. `capability_gap`
**Source:** Any daemon  
**When:** SI needs a capability that doesn't exist (broader than just tools)

```
details: {
  capability_name: string,            // e.g., "image_generation"
  description: string,                // What capability is needed
  use_case: string,                   // Why it's needed
  affected_daemons?: string[]         // Which daemons would use this
}
```

---

## Examples

### Example 1: FC Suggests New Action Type
```json
{
  "flag_id": "flag_20251104_001",
  "flag_type": "new_action_type",
  "severity": "info",
  "source_daemon": "frontal_cortex",
  "timestamp": "2025-11-04T14:32:00Z",
  "details": {
    "action_type_name": "parallel_task",
    "description": "Execute multiple tasks in parallel and wait for all to complete",
    "reason": "User asked to run 5 research tasks simultaneously",
    "suggested_policies": {
      "requires_confirmation": false,
      "auto_complete": true,
      "blocking": false
    }
  },
  "context": {
    "turn_id": "turn_20251104_042",
    "user_id": "user_123",
    "action_attempted": "Create 5 parallel research tasks",
    "rationale": "Current action types don't support parallel execution"
  }
}
```

### Example 2: Executor Suggests New Tool
```json
{
  "flag_id": "flag_20251104_002",
  "flag_type": "new_tool",
  "severity": "warning",
  "source_daemon": "executor",
  "timestamp": "2025-11-04T15:45:00Z",
  "details": {
    "tool_name": "pdf_analyzer",
    "tool_purpose": "Extract text, tables, and metadata from PDF files",
    "use_case": "User uploaded a PDF and asked for analysis",
    "suggested_inputs": ["pdf_file_path", "extraction_type"],
    "suggested_outputs": ["text_content", "tables", "metadata"]
  },
  "context": {
    "turn_id": "turn_20251104_043",
    "user_id": "user_456",
    "action_attempted": "Analyze PDF document",
    "error_message": "Tool 'pdf_analyzer' not found",
    "rationale": "User needs PDF analysis capability"
  }
}
```

---

## Storage in Turn Trace

Flags are stored in the turn trace under a `system_change_flags` array:

```
{
  turn_id: "...",
  stimulus: { ... },
  lifecycle_states: [ ... ],
  system_change_flags: [
    { flag_id: "flag_20251104_001", ... },
    { flag_id: "flag_20251104_002", ... }
  ],
  response: { ... },
  ...
}
```

---

## Severity Levels

- **info**: Nice-to-have improvement, not blocking
- **warning**: Useful improvement, would help with current task
- **critical**: Blocking issue, prevents task completion

---

## Next Steps

1. Define Proposal data structure (what System Analyzer generates)
2. Specify Change Flagging Daemon behavior
3. Specify System Analyzer Daemon behavior

