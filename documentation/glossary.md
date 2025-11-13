# Si Glossary

**Status:** v1.0 (Stable)
**Last Updated:** 2025-11-07
**Purpose:** Centralized definitions of Si-specific terminology

---

## Core Execution

**Turn** - The atomic cycle of Si's cognition, beginning when a stimulus is received and ending when Si produces a response and commits memory updates. A turn spans from sensory input to response output.

**Stimulus** - Input that triggers a turn (user message, system trigger, external event, time-based trigger).

**Response** - Output from a turn (message to user and/or actions taken).

**Commit** - The process of persisting state to memory systems at turn end. Includes episodic memory, semantic memory, frontal cortex, and scratch page updates.

**Turn Trace** - Ordered, immutable log of all decisions, tool invocations, observations, and events during a turn. Used for audit, debugging, learning, and transparency.

**Turn Context** - Metadata about a turn including turn_id, agent_id, conversation_id, budgets, stimulus details, and environment information.

---

## Memory Systems

**Working Memory** - Short-term context assembled for the current turn from episodic, semantic, conversation history, and frontal cortex. Injected into prompts for decision-making.

**Episodic Memory** - Long-term storage of turn events, observations, tool results, and experiences. Organized by timestamp and episode.

**Semantic Memory** - Factual knowledge, learned skills, and generalizable patterns extracted from episodic memory. Organized by semantic key.

**Conversation History** - Dialogue history between user and agent. Includes user inputs, assistant responses, and turn summaries.

**Memory Item** - Individual unit of memory with core fields (id, type, text, tokens_estimate, timestamp, agent_id, tags, metadata) and store-specific fields.

**Episode** - Grouped set of related episodic memory items, typically representing a coherent sequence of events or a conversation session.

---

## Memory Operations

**Memorize** - Operation to store information in memory (episodic, semantic, or conversation history).

**Remember** - Operation to retrieve information from memory based on query or context.

**Forget** - Operation to remove information from memory (soft delete or hard delete).

**Upsert** - Update if exists, insert if new. Used in semantic memory to maintain single version of facts.

**Versioning** - Creating new versions on mutation. Each change creates a new version with incremented version number.

**Lineage** - Chain of previous versions forming an audit trail. Enables revert and historical analysis.

**Soft Delete** - Mark item as deleted but keep in store for audit and compliance. Item excluded from queries by default.

**Hard Delete** - Permanent removal from store. Used for sensitive data or space reclamation.

**Eviction** - Removal of items when capacity limits exceeded. Follows store-specific policies (FIFO+importance for working memory, LRU+importance for semantic).

**Compaction** - Summarization and archival of old items to reduce storage while preserving key information.

**Redaction** - Removal of sensitive information before persistence to non-working-memory stores.

---

## Daemons

**Daemon** - Self-contained autonomous system with own decision-making ability, instructions, and purpose. Implements Daemon Interface (get_purpose, get_instructions, query).

**Router** - Daemon that receives stimulus, decides urgency level, and determines which daemons to activate.

**Clarifier** - Daemon that handles ambiguous inputs by asking clarifying questions.

**Executor** - Daemon that orchestrates turn execution by querying other daemons and deciding which lifecycle states to execute.

**Error Handler** - Daemon that handles failures by deciding recovery strategy (retry, alternative tool, escalate, fail gracefully).

**Evaluator** - Daemon that assesses whether turn is complete and successful.

**Reflector** - Daemon that learns from turn outcomes and updates consciousness, episodic memory, and skills.

**Consciousness** - Daemon providing Si's self-concept, identity, mandates, capabilities, and governance rules. Implements Daemon Interface.

---

## Consciousness

**Mandate** - What Si believes and values. Organized in 4 layers: Mission (core purpose), Safety (constraints), Collaboration (interaction principles), Operational (execution guidelines).

**Capability** - What Si can do. Describes behavioral requirements and constraints.

**Mode** - Operational state or context (e.g., "research mode", "execution mode"). Affects decision-making and behavior.

**Governance Rule** - Constraint or change control that governs Si's behavior and evolution.

**Core Identity** - Si's fundamental self-concept and values.

---

## Frontal Cortex

**Goal** - Long-horizon objective or milestone. Has priority, horizon (days/steps), status, progress metrics, and constraints.

**Pending Action** - Concrete next step assigned to agent or user. Has type, owner, priority, status, and due date.

**Question** - Guiding question for steering goals. Linked to goals and stories.

**Story** - Narrative capturing observations and insights. Linked to questions and goals. Has acceptance criteria and evidence references.

**Action Type** - Category of action (agent_task, user_task, approval_request, reminder, follow_up, research, decision, background_job, blocker). Defined in registry.

**Registry Proposal** - Proposal for new action type, policy change, or deprecation. Submitted mid-turn, reviewed by humans, merged via PR process.

---

## Scratch Page

**Observation** - Contextual insight, alert, or finding captured during turn. Low-commitment, may expire.

**Vignette** - Short narrative of an observation with context and implications.

**Pending Observation** - Observation awaiting resolution or action.

**Resolved Observation** - Observation that has been addressed or archived.

---

## Skills & Learning

**Skill** - Reusable procedure or decision-making pattern. Has name, description, preconditions, steps, postconditions, parameters, examples, and confidence.

**Pattern** - Repeated sequence of actions identified in episodic memory.

**Skill Extraction** - Process of identifying patterns in memory and extracting them as reusable skills.

**Procedural Skill** - Step-by-step procedure for accomplishing a task.

**Decision Skill** - Decision-making procedure for choosing among options.

**Interaction Skill** - How to interact with tools, APIs, or external systems.

**Communication Skill** - How to communicate with user or other agents.

---

## Tools

**Tool** - Self-contained executable unit with manifest, inputs, outputs, and side effects.

**Tool Invocation** - Execution of a tool with specific parameters.

**Tool Request** - Request sent to tool containing request_id, tool_id, inputs, and policies.

**Tool Response** - Response from tool containing status, outputs, observations, memory_writes, and error information.

**Tool Manifest** - Declarative specification of tool capabilities, inputs, outputs, policies, and resources.

**Tool Error** - Failure during tool execution.

**Transient Error** - Temporary error likely to succeed on retry (network timeout, rate limit, temporary unavailability).

**Permanent Error** - Fundamental error unlikely to resolve on retry (invalid input, auth failure, not found).

**Retry** - Attempt to execute tool again after failure.

**Escalate** - Raise issue to higher level (user, human, system).

**Cascading Failure** - Failure that propagates to dependent operations.

---

## Budgets & Resources

**Budget** - Resource limit controlling cost and latency (tokens, tool calls, time).

**Token Budget** - Limit on LLM prompt size (maximum tokens in a single LLM call).

**Tool-Call Budget** - Limit on number of tool invocations in a turn.

**Time Budget** - Limit on turn execution time.

**LLM Budget System** - System controlling token usage for LLM calls. Enforces global and daemon-specific budgets.

---

## Data Model

**Metadata** - Data about data (tags, source, confidence, timestamps, etc.).

**Approval Status** - Whether item is approved for use (draft, approved, deprecated).

**Effective Date** - When item becomes active or expires.

---

## Related Documentation

- **Core Engineering Tenants**: `documentation/si_core_engineering_tennants.md`
- **Turn Architecture**: `documentation/turn/overview.md`
- **Working Memory**: `documentation/working-memory/overview.md`
- **Consciousness**: `documentation/consciousness/README.md`
- **Frontal Cortex**: `documentation/frontal-cortex/README.md`
- **Scratch Page**: `documentation/scratch-page/overview.md`
- **Tools**: `documentation/tools/si_tools_architecture.md`

