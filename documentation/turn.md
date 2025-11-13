# Turn — Definition and Lifecycle (Platform-Independent)

Author: Augment Agent
Status: Draft v0.2
Last updated: 2025-11-01

**Related Documentation**: See `turn/overview.md` for conceptual overview, examples, and daemon architecture introduction.

## What is a Turn?
A turn is the atomic cycle of present cognition for Si, beginning when a stimulus is received (e.g., user message, time-based trigger, external event) and ending when Si produces a response and/or completes actions and commits memory updates.

## Inputs and Outputs
- Inputs: Stimulus {type, payload, source, timestamps}, current conversation_id, agent_id, config budgets.
- Outputs: Response (message and/or actions), Turn Trace (ordered log of thoughts/tools/observations), Memory commits (EM/SM/CH/FC updates), Metrics.

## Lifecycle States (Possible Steps, Not Mandatory)

The following states represent **possible steps** that may occur during a turn. The **Executor daemon** (an LLM-based autonomous agent) adaptively decides which steps to execute based on the current situation. Not all steps occur in every turn.

1) **Stimulus Received**: Input arrives and is logged
2) **Pre-Retrieval**: Build ReadContext (budgets, filters, objectives) — may be skipped for simple queries
3) **WM Assembly**: Orchestrator assembles Working Memory bundle (consciousness, CH recent + summary, FC goals and pending actions, retrieved EM/SM/others) — may be skipped if context is already available
4) **Thought/Planning**: Transient reasoning captured in WM — may be skipped for straightforward decisions
5) **Action(s)**: Tool invocations executed; observations captured — may be skipped if no tools needed
6) **Reflection/Summarization**: Micro/meso summaries generated as needed — may be skipped for simple turns
7) **Response**: Produce reply/action decision — always occurs (at minimum, a response is sent)
8) **Commit/Flush**: Persist selected artifacts; update FC; evict/compact WM — always occurs

**Note:** Executor doesn't execute a pre-determined sequence. Instead, it uses LLM reasoning to adaptively decide what to do next based on the current situation, available information, and constraints.

### Turn Flow and Daemon Orchestration

The turn follows this flexible, data-driven flow:

1. **Router** receives stimulus and decides:
   - Can this be answered quickly using Scratch Page or without additional tools?
   - If YES → Responder generates answer and sends to user immediately
   - If NO → Continue to Executor

2. **Executor ALWAYS spawned** (regardless of Router's decision):
   - Queries Working Memory: "What's the current situation?"
   - Receives comprehensive context:
     - Consciousness (mandates, capabilities, modes, governance)
     - Frontal Cortex (goals, pending actions, constraints)
     - Conversation history and observations
     - Episodic and semantic memory (relevant to task)
     - Available tools and daemons
     - Budget status
   - Uses LLM to decide what to do next:
     - Call a tool to get information or take action
     - Query a daemon (FC, Consciousness, Memory, etc.)
     - Spawn a sub-turn for parallel work
     - Respond to the user
     - End the turn
   - Executes the decision and collects results
   - Loops with results until turn is complete
   - Handles errors and constraints during execution
   - Updates Working Memory at commit

3. **Frontal Cortex** (queried by Executor when needed):
   - Provides planning assistance based on goals and context
   - Can be queried multiple times as situation evolves
   - Helps Executor understand long-horizon implications

4. **Logging**: All decisions and reasoning recorded to Turn Trace for auditability and learning

## Turn Context (data sketch)
- turn_id, agent_id, conversation_id, started_at, deadline?, budgets {tokens/context, tools, time}, stimulus {type, payload, source}, env {tenant, permissions}, trace_id.

## Invariants
- Single-threaded ordering within a turn (actions occur in logged order).
- Respect budgets: token, tool-call count, and time.
- All tool invocations are recorded in Turn Trace with timestamps.
- All daemon decisions are recorded in Turn Trace for auditability and learning.
- Privacy guard runs before persistence to non-WM stores.

## Turn-Level Budgets

Each turn has three types of budgets that control the cost and latency of that turn:

1. **LLM Call Budget**: Maximum tokens allowed in a single LLM call prompt during this turn (enforced per daemon, see `documentation/llm-budget-system/overview.md`)
2. **Tool-Call Budget**: Maximum number of tool calls allowed in this turn
3. **Time Budget**: Maximum time allowed for this turn to complete (deadline)

These are **per-turn constraints** that control the cost and latency of a single turn execution. They are separate from:
- **Per-User Cost Tracking**: Lifetime and period-based token limits across all turns (see `documentation/llm-budget-system/per-user-cost-tracking.md`)
- **Memory Store Capacity**: Storage limits for episodic, semantic, and working memory (technology-dependent, managed per-store)

## Policies and Gating
- Importance/novelty gating for persistence at commit.
- Timeout/deadline handling: if deadline approaches, force summarization and commit minimal viable output.

## Daemon Decision-Making During Turn

### Router's Decision
Router decides:
- **Quick Answer**: Can this be answered quickly using Scratch Page or without tools?
- **If YES**: Responder generates answer and sends to user immediately
- **If NO**: Continue to Executor (which will run regardless)

### Executor's Decision
Executor decides:
- **Context**: What context is needed? (queries Working Memory)
- **Plan**: What should happen next? (queries FC for plan)
- **Execution**: Execute plan flexibly:
  - Which daemons to call?
  - In what order?
  - Which can run in parallel?
  - Should sub-turns be spawned?
- **Error Handling**: Handle errors and constraints during execution
- **Response**: Send response to user (if not already sent by Router)

### FC's Planning Decision
FC decides:
- **What's next**: Based on goals, pending actions, and context
- **Which daemons**: What daemons should be called? (Reflector, Gut, Evaluator, etc.)
- **Execution strategy**: Sequential? Parallel? Sub-turns?
- **Plan output**: Structured plan that Executor executes

### Daemon Flexibility
The daemon flow is **not fixed** - it's determined by FC's plan. Examples:
- **Learning turn**: FC plans to call Reflector (understand outcome, update memory)
- **Execution turn**: FC plans to call multiple daemons in parallel (fulfill goal)
- **Complex turn**: FC plans to spawn sub-turns for parallel work

This enables the system to be **learnable and flexible**: as FC's planning improves, the system will make better decisions about which daemons to call and how to orchestrate them.

See `documentation/turn/decision-making-model.md` for details on how daemons make decisions.

## Registry Proposals (type changes)
- During Thought/Planning or Reflection, the planner may suggest new action types or policy edits.
- At Commit/Flush, WM submits these as FC RegistryProposals; they do not alter current-turn behavior.
- If an unregistered type is attempted, capture a proposal and coerce to a safe default or prompt for a known type.

## Testing Requirements
- Executor correctly decides which lifecycle states to execute based on daemon outputs.
- Deterministic execution under fixed seeds and configs.
- Turn Trace completeness: every daemon decision and action captured with start/end and status.
- Commit produces expected writes given fixtures; no unredacted PII persists.
- System can skip states appropriately and still produce correct results.

## Acceptance Criteria
- A coding agent can implement a Turn runner that executes this lifecycle and passes the tests.
- The Turn integrates WM, CH, EM, SM, and FC per the budgets and policies defined here.
- Executor daemon correctly orchestrates lifecycle state execution based on daemon decisions.
- Turn Trace provides full auditability of which states were executed and why.

