# Frontal Cortex (FC) — Planning Daemon and Long-Horizon Goals

**Status:** v0.2 (Draft)
**Last Updated:** 2025-11-07
**Author:** Augment Agent

## Purpose
FC is a daemon that serves two roles:
1. **Planning Daemon**: Builds execution plans that Executor executes (intra-turn planning)
   - Uses LLM to generate natural language plans
   - Decides which daemons to call
   - Decides execution order and parallelization
   - Decides if sub-turns should be spawned
2. **Long-Horizon Goals System**: Maintains long-horizon goals and actionable next steps across turns (inter-turn planning)

## Role in the Agent

### As a Daemon
- Implements the Daemon Interface (query(), get_purpose(), get_instructions())
- Queried by Executor to create/update plans for the current turn
- Receives context from Executor and returns execution plan
- Can be queried by Gut to update plan if concerns are raised

### As a Long-Horizon System
- Durable across turns/conversations; shared per agent (and optionally per org/tenant)
- Provides two primary constructs:
  1) Goals: long-horizon objectives, milestones, constraints
  2) Pending Actions: concrete next steps assigned to the agent or the user
- Integrates a Question/Story framework to steer big goals via guiding questions and evolving narratives

## Plan Generation Algorithm (v1 - LLM-Based)

### Overview

FC generates execution plans using an LLM. The LLM receives context about available daemons, tools, current state, consciousness mandates, and relevant memories, then generates a natural language plan with steps, success criteria, and parallelization information.

### Inputs to Plan Generation

When Executor queries FC for a plan, FC gathers:

1. **Daemon List**: Available daemons that can be called
   - Router, Clarifier, Gut, Executor, Reflector, Responder, Evaluator, etc.
   - Each daemon's purpose and instructions

2. **Tool List**: Available tools that can be invoked
   - Web search, expert archive search, contact bank, send message, etc.
   - Each tool's description and parameters

3. **User Input**: The current query or request

4. **Current State** (from FC's persistent storage):
   - Active goals and their progress
   - Pending actions and their status
   - Relevant questions and stories

5. **Consciousness Context** (from Consciousness daemon):
   - Mandates and capabilities
   - Current mode and governance rules
   - Constraints and priorities

6. **Relevant Memories** (from Working Memory):
   - Recent observations and context
   - Episodic memory of similar situations
   - Semantic memory of learned patterns

### Plan Output Format

FC returns a **natural language plan** with clear structure:

```
[Natural language narrative describing the execution strategy]

## Step 1: [Step Name]
Success Criteria: [What success looks like]
Actions: [What to do]
Parallelizable with: [Step 2, Step 4]

## Step 2: [Step Name]
Success Criteria: [What success looks like]
Actions: [What to do]
Parallelizable with: [Step 1]

## Step 3: [Step Name]
Success Criteria: [What success looks like]
Actions: [What to do]
Parallelizable with: [None - must wait for Step 2]
```

Each step includes:
- **Description**: Natural language explanation of what this step does
- **Success Criteria**: How to know the step succeeded
- **Actions**: What to do (daemons to call, tools to invoke, etc.)
- **Parallelization Info**: Which other steps can run simultaneously

### Executor Interpretation

The Executor interprets each plan step using an LLM:

1. **Receives step description** from the plan
2. **Provides context to LLM**:
   - Step description and success criteria
   - Summary of current state (what's happened so far in this turn)
   - Available daemons and tools
3. **LLM decides what to do**:
   - Which daemons to call
   - Which tools to invoke
   - How to handle results
4. **Executor executes the decision**
5. **Executor checks success criteria**
   - If success → move to next step
   - If failure → handle error and potentially regenerate plan

### Learning and Optimization (v2 - Future)

All context and decisions are logged to Turn Trace:
- What plan was generated
- What context was provided
- What the Executor decided to do
- Whether it succeeded

This enables future learning:
- Learn optimal context patterns for different types of queries
- Learn which daemons/tools work best for different situations
- Learn to generate better plans over time
- Adapt plan generation strategy based on outcomes

## Relationship to Scratch Page

**Frontal Cortex** represents formal commitments and plans that result from observations captured in the **Scratch Page**:

- **Scratch Page**: Stories and vignettes that capture observations, insights, and alerts (low-commitment, may expire)
- **Frontal Cortex**: Formal goals and actionable next steps that result from those stories (high-commitment, durable)

**Flow**:
1. Tools/daemons observe something → Add story to Scratch Page
2. Router/Executor review Scratch Page stories
3. If story warrants action → Create Frontal Cortex action/goal
4. If not → Story remains in Scratch, expires, or gets archived

**Key Distinction**:
- Scratch Page is the "working notes" where observations accumulate
- Frontal Cortex is the "official record" of commitments and plans
- Not all Scratch Page stories become Frontal Cortex actions
- All Frontal Cortex actions should be traceable to the observations that motivated them

## Data Model (platform-agnostic)
- Goal: { id, title, description, priority, horizon (days/steps), status, progress, metrics{}, constraints[], parent_goal_id?, lineage_ids[], created_at, updated_at }
- PendingAction: { id, type, owner: agent|user, title, description?, priority, status, due_at?, goal_id?, blocking?, requires_confirmation?, created_by, created_from_turn_id?, evidence_refs[], metadata{}, created_at, updated_at }
- Question: { id, text, goal_id?, priority, status, owner?, tags[], created_at, updated_at }
- Story: { id, question_id, narrative, acceptance_criteria[], evidence_refs[], status, last_updated }

- ActionType: { id, display_name, description?, allowed_owners: (agent|user)[], default_policies: { requires_confirmation?: bool, auto_complete?: bool, escalation?: { after?: duration, policy?: string }, blocking?: bool }, allowed_statuses: string[], allowed_transitions: { [from_status: string]: string[] }, policy_handlers?: { escalation?: string, completion_guard?: string }, deprecation_status: active|deprecated }


### Action Types (initial)
- agent_task: executable by agent; may auto-complete
- user_task: requires user acknowledgment to complete
- approval_request: requires_confirmation before proceeding
- reminder: time-based nudge, can escalate
- follow_up: check back after an event or time window
- research: gather information before execution
- decision: requires explicit choice among options

- background_job: long-running or async process to monitor
- blocker: indicates a blocking condition that halts progress

### Type Registry and Extensibility
- The list of Action Types is defined in a declarative registry (e.g., JSON/YAML or a table) managed by FC.
- Change process:
  1) Propose: add/update/remove type with rationale and policy spec.
  2) Define policies: default_policies, allowed_statuses, transitions, owner constraints.
  3) Tests: add fixtures covering policy semantics and transitions.
  4) Migrate: if renaming/removing, supply a mapping/migration plan.
  5) Rollout: stage behind a feature flag; monitor metrics per type; then graduate.
  6) Deprecation: mark deprecated; keep id stable and block new creation if necessary.
- Registry operations: list_action_types(), register_action_type(t: ActionType), deprecate_action_type(id)
- Backward compatibility: avoid deleting type ids; instead, deprecate and provide migration guidance.



### No-code Type Onboarding
- The orchestrator and FC are type-agnostic and policy-driven; adding a new Action Type does not require code changes.
- To add a type: update the registry entry (repo file or approved runtime override), pass schema validation/tests, then publish.
- Behavior comes from default_policies, allowed_statuses, allowed_transitions, and optional policy_handlers.
- If a behavior cannot be expressed via policies/handlers, propose a new generic policy or handler key rather than hardcoding a type.


### Planner Suggestion Pathway (pre-defined types + suggestions)
- Policy: Only pre-registered Action Types are executable at runtime. The planner cannot activate new types mid-turn.
- When the planner believes a new type or policy change is warranted, it submits a RegistryProposal for review.

Data model:
- RegistryProposal: { id, proposal_type: add|modify|deprecate, type_id?, display_name?, description?, diff?, suggested_policies?, allowed_statuses?, allowed_transitions?, rationale, examples[], risks[], created_by, created_from_turn_id, status: open|approved|rejected|merged, reviewers[], created_at, updated_at }

Operations:
- submit_registry_proposal(p: RegistryProposal) -> Ack
- list_registry_proposals(filters) -> RegistryProposal[]
- review_registry_proposal(id, decision: approve|reject, comments) -> Ack
- export_proposal_as_pr(id) -> { repo, branch, pr_url } (docs-as-code flow)

Governance flow:
1) Planner submits proposal during Thought/Planning or Reflection.
2) FC stores proposal, notifies maintainers, and links the originating turn.
3) Maintainer reviews; if accepted, a PR updates the registry (e.g., action_types.yaml).
4) CI validates schema, policy semantics, transitions, and fixtures; staged rollout via flags.
5) Merge/deploy; FC loads the updated registry; proposal -> merged.
6) Observability monitors new type behavior; deprecation guidance if needed.

Safety & behavior:
- Unknown types referenced mid-turn are not executed; create a proposal and coerce to a safe default (e.g., user_task with requires_confirmation) or prompt the planner to choose an existing type.
- Proposals are visible in the Turn Trace and never alter current-turn semantics.

- Observability: track creation/completion rates, overdue counts, and error rates per action type.

- Unknown or missing types: fall back to safe defaults (treated as user_task requiring confirmation; no auto-complete) until registry syncs.


## Interfaces
- FrontalCortex
  - get_active_goals(agent_id, org_id?, limit?): Goal[]
  - list_pending_actions(filters): PendingAction[]
  - upsert_goal(goal: Goal): Ack
  - upsert_pending_actions(actions: PendingAction[]): Ack
  - update_progress(deltas): Ack
  - get_questions(goal_id?): Question[]
  - upsert_question_story(q: Question, story: Story): Ack
- WM/Turn Runner integration
  - assess_at_turn_start(turn_ctx) -> {goals, pending_actions, questions/stories}
  - commit_at_turn_end(turn_ctx, updates: {progress, new_actions, completed_actions, story_updates}) -> Ack
- Type semantics: policies may vary by PendingAction.type (e.g., approval_request requires_confirmation; user_task cannot be auto-completed by the agent; reminder escalates after due_at).

## Turn-level Behavior
- At start of turn: WM pulls FC goals, top pending actions (by priority/due date/goal relevance), and relevant questions/stories.
- During turn: actions executed can create/complete pending actions; reflections may update progress and stories.
- At commit: WM posts deltas: progress updates, mark-done or new pending actions, story updates (with evidence links).

- Registry proposals: submission, review, and PR export are covered by tests; proposals never affect runtime until merged and loaded.

## Policies
- Prioritization: order pending actions by utility density (priority/effort), due_at, blocking, and goal alignment.
- Ownership: owner=agent vs owner=user determines who is expected to act; agent should not auto-complete user-owned tasks.
- Versioning: all mutations create new versions; support audit trail and revert.
- Evidence: link tool results, files, or references to actions/progress and story updates.
- Access control: scope by agent_id/org_id; optional cross-agent sharing by ACL.

## Persistence Layer

See `documentation/frontal-cortex/persistence-layer.md` for detailed specification of:
- **Conflict Resolution**: Optimistic locking with field-level versioning for concurrent updates
- **Audit Trail**: Immutable mutation history with lineage tracking and revert capability
- **Access Control**: ACL-based sharing model for multi-agent scenarios
- **Query Patterns**: Turn-start retrieval, filtering, and batch operations
- **Integration**: Turn-start and turn-end commit phases with error handling

See `documentation/frontal-cortex/persistence-layer-test-conditions.md` for comprehensive test scenarios.

## Summarization & Compaction
- Periodic goal summaries (meso) and macro narratives for major milestones.
- Story updates at natural breakpoints; archive deprecated questions/stories with lineage.
- Compact old action logs into summaries while preserving key evidence pointers.

## Retrieval Guidance
- For WM assembly: select
  - Top-K goals by relevance to the stimulus/query.
  - Top-K pending actions (mix of urgent + strategic) for the current context.
  - Top relevant questions/stories to guide planning and explanation.

## Testing Requirements
- Deterministic selection of relevant goals/actions under fixed seeds.
- Owner semantics enforced: agent cannot mark user-owned items complete in tests.
- Commit applies deltas idempotently; replays produce same FC state.
- Story updates preserve acceptance_criteria and add evidence links.

## Acceptance Criteria
- A coding agent can implement FC interfaces and integrate with WM/Turn runner per this spec.
- FC persists across turns; WM both reads at start and updates at commit.
- Pending actions are partitioned by owner and prioritized; question/story pairs are retrievable and updateable.

