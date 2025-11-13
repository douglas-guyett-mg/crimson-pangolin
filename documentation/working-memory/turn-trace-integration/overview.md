# Turn Trace Integration

**Status**: Specification v0.2
**Last Updated**: 2025-10-31
**Mental Model**: Brain as Daemons/Mini-Agents

## Overview

Turn Trace Integration defines how the Working Memory system logs and traces all operations for debugging, auditing, and analysis. It specifies what information is logged, how logs are structured, how they can be queried, and how they support system improvement.

The Turn Trace is an intra-turn tracing system that records intentions, decisions, actions, observations, and policy effects using a "brain as daemons/mini-agents" model. Each functional brain area is represented as a mini-agent that emits messages and spans into the Turn Trace, communicating in agentic style alongside structured fields for programmatic use.

## Purpose

Turn Trace Integration serves to:

1. **Enable Debugging**: Log all operations for debugging
2. **Support Auditing**: Create audit trail of all memory operations
3. **Enable Analysis**: Provide data for system analysis and improvement
4. **Support Transparency**: Make system decisions transparent
5. **Enable Monitoring**: Monitor system health and performance
6. **Enable Real-Time Control**: Support post-turn learning and accountability
7. **Support Integration**: Link with Frontal Cortex, Episodic Memory, and Semantic Memory

## Mental Model: Brain Components as Mini-Agents (Daemons)

Each functional brain area is represented as a mini-agent (daemon) that emits messages and spans into the Turn Trace:

- **WM-Orchestrator (wm)**: Context assembly, budgets, commit
- **Planner / dlPFC (pfc)**: Plans steps, hypotheses
- **Action Selector / Basal Ganglia (bg)**: Selects next action/tool
- **Executor / Motor + Tools Adapter (motor)**: Issues tool calls
- **Observer / Sensory Integration (sense)**: Captures observations
- **Monitor / ACC (acc)**: Detects errors/conflicts/anomalies
- **Predictor / Cerebellum (cbl)**: Predicts outcomes, compares errors
- **Episodic Writer / Hippocampus (hip)**: Curates episodic writes
- **Semantic Distiller (sem)**: Distills reusable facts/skills
- **Frontal Cortex Bridge (fcb)**: Reads/updates FC goals & actions
- **Summarizer (sum)**: Produces micro/meso/macro summaries

All of the above are logical roles; implementers may map them to processes/threads/tasks.

## Communication Style

In addition to structured events, daemons communicate using AgentMessage for readability and explainability:

- **AgentMessage**: { speaker: component_id, utterance: string, audience?: string[], intent?: plan|decide|act|observe|reflect|warn|propose }
- **Style guidance**: Concise, first-person ("pfc: I propose we…", "bg: Selecting tool X (score=0.82)")
- **Messages are optional**: Structured fields MUST be sufficient for behavior

## Data Model (Platform-Agnostic)

### Top-Level Structures
- **Trace**: { trace_id, turn_id, agent_id, conversation_id?, started_at, ended_at?, seed?, budgets{}, components: string[] }
- **Span**: { span_id, trace_id, parent_span_id?, component: wm|pfc|bg|motor|sense|acc|cbl|hip|sem|fcb|sum, name, start_ts, end_ts?, attributes{} }
- **Event (base)**: { event_id, ts, trace_id, span_id?, component, kind, agent_msg?: AgentMessage, attributes{} }

### Event Kinds (Extensible via Registry)
- **PlanStep**: { step_text, rationale?, goal_id?, candidates?: [ActionCandidate] }
- **Decision**: { chosen: ActionRef, considered?: [ActionCandidate], policy_notes?, score?: number }
- **ToolInvocation**: { tool_name, inputs_digest, args_schema_ref?, attempt, status: queued|started|succeeded|failed|cancelled, latency_ms?, cost?, output_digest?, error_class?, error_msg? }
- **Observation**: { summary, output_digest, tokens_out?, artifacts?: EvidenceRef[] }
- **Error**: { class, message, retryable?, backoff_ms?, policy_triggered?: string }
- **Policy**: { rule_id, action: allow|deny|redact|rate_limit, details{} }
- **Summary**: { level: micro|meso|macro, text, tokens? }
- **Proposal**: { proposal_type: registry|policy, payload_ref }

### Supporting Types
- **ActionRef**: { pending_action_id?, type, owner: agent|user, title?, status? }
- **ActionCandidate**: { ref?: ActionRef, type, tool_name?, arguments_digest?, score? }
- **EvidenceRef**: { kind: url|file|artifact_id|hash, value }

### Redaction Policy
Sensitive fields (inputs/outputs) should be hashed (SHA-256) or truncated with thresholds; store small excerpts only. Full payloads are not required for trace.

## Protocol & Ordering

- Within a component, events are strictly ordered (monotonic seq)
- Global order uses (ts, component_seq, component_id) for deterministic replay
- Spans form a DAG over actions (parent→child); cross-links allowed
- End-of-turn flush emits a TurnSummary and prepares commit artifacts

## Logged Operations

### Memory Operations
All memory operations are logged:
- **Memorize Operations**: Item memorized, store, timestamp, budget consumed
- **Remember Operations**: Prompt/query, results returned, search time
- **Forget Operations**: Item forgotten, reason, mode (soft/hard), timestamp
- **Summarize Operations**: Scope, summary generated, compression ratio, time
- **Get Capacity Info Operations**: Capacity queried, current usage, available space
- **Eviction Operations**: Items evicted, reason, timestamp

### Gating Operations
All gating decisions are logged:
- **Redaction**: What was redacted, redaction level, tokens saved
- **Scoring**: Scores assigned, scoring factors, combined score
- **Routing**: Routing decisions, target stores, rationale
- **Eviction Cascade**: Evictions triggered, items evicted, reason

### Orchestration Operations
All orchestration operations are logged:
- **Context Assembly**: Context assembled, items included, token count
- **Tool Invocation**: Tool called, parameters, result, time
- **Commit**: Outcome committed, success/failure, feedback
- **Budget Updates**: Budget consumed, remaining, status

### Integration Operations
All integration operations are logged:
- **Frontal Cortex Calls**: Context requests, plan execution, outcomes
- **Skill Operations**: Skills extracted, skills applied, confidence updates
- **Summarization**: Summaries generated, compression ratio, quality

## Log Structure

### Log Entry Format
Each log entry contains:
- **timestamp**: When operation occurred
- **operation_type**: Type of operation (memorize, remember, forget, summarize, get_capacity_info)
- **agent_id**: Which agent performed operation
- **turn_id**: Which turn this occurred in
- **operation_id**: Unique ID for this operation
- **details**: Operation-specific details
- **duration**: How long operation took
- **budget_consumed**: Budget consumed by operation
- **status**: Success or failure
- **error**: Error message if failed

### Log Levels
- **DEBUG**: Detailed information for debugging
- **INFO**: General information about operations
- **WARN**: Warnings about potential issues
- **ERROR**: Errors that occurred
- **CRITICAL**: Critical errors that need attention

## Operations

### log_operation(operation_type, details)
Logs an operation.

**Parameters**:
- `operation_type`: Type of operation
- `details`: Operation details

**Returns**:
- Log entry ID
- Confirmation

### query_logs(filters)
Queries logs with filters.

**Parameters**:
- `filters`: Filter criteria (operation_type, agent_id, time_range, etc.)

**Returns**:
- List of matching log entries
- Total count

### get_operation_trace(operation_id)
Gets full trace for an operation.

**Parameters**:
- `operation_id`: Operation ID

**Returns**:
- Full operation trace
- Related operations

### analyze_logs(analysis_type)
Analyzes logs for insights.

**Parameters**:
- `analysis_type`: Type of analysis (performance, errors, patterns, etc.)

**Returns**:
- Analysis results
- Insights and recommendations

## Interfaces (Behavioral)

### TurnTrace API (Logical)
- **start_trace(turn_ctx)** → { trace_id }
- **start_span(component, name, attrs?)** → span_id
- **log_event(kind, component, payload, agent_msg?, span_id?)** → event_id
- **end_span(span_id, attrs?)**
- **link_pending_action(event_id, pending_action_id)**
- **propose_registry_change(proposal)** → Ack
- **subscribe(consumer_id, filters)** → stream<Event>
- **export_for_commit()** → { episodic_items[], semantic_items[], fc_updates[], evidence_refs[] }
- **end_trace(outcome, attrs?)**

### Queries (Read-Side)
- **get_trace(trace_id)** → Trace with all spans and events
- **list_events(trace_id, filters)** → Event[]
- **list_tool_invocations(trace_id, status?, tool_name?)** → ToolInvocation[]
- **summarize(trace_id, level)** → Summary

## Design Principles

1. **Comprehensive**: All operations are logged
2. **Structured**: Logs are structured for easy querying
3. **Efficient**: Logging has minimal performance impact
4. **Secure**: Sensitive data is redacted from logs
5. **Queryable**: Logs can be queried and analyzed

## Interaction with Other Components

- **Memory Orchestrator**: Calls log_operation for all operations
- **Memory Interfaces**: Log their operations
- **Write Gating Algorithm**: Logs gating decisions
- **Frontal Cortex**: Logs planning operations
- **Skill Extraction**: Logs skill operations

## Configuration

Turn Trace Integration can be configured with:

- **log_level**: Minimum log level (default: INFO)
- **enable_detailed_logging**: Whether to log detailed information (default: false)
- **log_retention**: How long to keep logs (default: 30 days)
- **enable_log_compression**: Whether to compress old logs (default: true)
- **redact_sensitive_data**: Whether to redact sensitive data (default: true)

## Example Scenario

1. Agent memorizes item to memory
2. **Log Entry Created**:
   - timestamp: 2024-01-15T10:30:45Z
   - operation_type: "memorize"
   - agent_id: "agent_001"
   - turn_id: "turn_123"
   - operation_id: "op_456"
   - details: {item_id: "item_789", store: "working_memory", tokens: 50}
   - duration: 10ms
   - budget_consumed: 50
   - status: "success"
3. **Gating Operations Logged**:
   - Redaction: "No redaction needed"
   - Scoring: "score=0.75"
   - Routing: "Route to working_memory"
4. **Query Logs**:
   - User queries: "Show all memorize operations in last hour"
   - System returns: 150 memorize operations
5. **Analyze Logs**:
   - User requests: "Analyze performance"
   - System returns: "Average memorize time: 12ms, 95th percentile: 45ms"

