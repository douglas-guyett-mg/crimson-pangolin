# Turn — Definition and Lifecycle (Platform-Independent)

Author: Augment Agent
Status: Draft v0.1
Last updated: 2025-10-29

## What is a Turn?
A turn is the atomic cycle of present cognition for Si, beginning when a stimulus is received (e.g., user message, time-based trigger, external event) and ending when Si produces a response and/or completes actions and commits memory updates.

## Inputs and Outputs
- Inputs: Stimulus {type, payload, source, timestamps}, current conversation_id, agent_id, config budgets.
- Outputs: Response (message and/or actions), Turn Trace (ordered log of thoughts/tools/observations), Memory commits (EM/SM/CH/FC updates), Metrics.

## Lifecycle States
1) Stimulus Received
2) Pre-Retrieval: build ReadContext (budgets, filters, objectives)
3) WM Assembly: Orchestrator assembles Working Memory bundle (CH recent + summary, FC goals and pending actions, retrieved EM/SM/others)
4) Thought/Planning: transient reasoning captured in WM
5) Action(s): tool invocations executed; observations captured
6) Reflection/Summarization: micro/meso summaries generated as needed
7) Response: produce reply/action decision
8) Commit/Flush: persist selected artifacts; update FC; evict/compact WM

## Turn Context (data sketch)
- turn_id, agent_id, conversation_id, started_at, deadline?, budgets {tokens/context, tools, time}, stimulus {type, payload, source}, env {tenant, permissions}, trace_id.

## Invariants
- Single-threaded ordering within a turn (actions occur in logged order).
- Respect budgets: token, tool-call count, and time.
- All tool invocations are recorded in Turn Trace with timestamps.
- Privacy guard runs before persistence to non-WM stores.

## Policies and Budgets
- Section budgets for context assembly (WM/CH/EM/SM/FC).
- Importance/novelty gating for persistence at commit.
- Timeout/deadline handling: if deadline approaches, force summarization and commit minimal viable output.


## Registry Proposals (type changes)
- During Thought/Planning or Reflection, the planner may suggest new action types or policy edits.
- At Commit/Flush, WM submits these as FC RegistryProposals; they do not alter current-turn behavior.
- If an unregistered type is attempted, capture a proposal and coerce to a safe default or prompt for a known type.

## Testing Requirements
- Lifecycle transitions occur in order; missing any step is flagged.
- Deterministic assembly under fixed seeds and configs.
- Turn Trace completeness: every action captured with start/end and status.
- Commit produces expected writes given fixtures; no unredacted PII persists.

## Acceptance Criteria
- A coding agent can implement a Turn runner that executes this lifecycle and passes the tests.
- The Turn integrates WM, CH, EM, SM, and FC per the budgets and policies defined here.

