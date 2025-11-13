# Decisions Made

This document records architectural and design decisions made during Si development. These decisions are recorded to prevent future reviews from re-flagging the same issues.

---

## Privacy & Data Redaction

**Decision:** Privacy is enforced via **access control**, not data redaction.

**Rationale:**
- Each user's memories and data are only accessible to their own agent instance
- Access control at the data layer (database/storage) prevents unauthorized access
- Data redaction adds complexity and potential for errors
- Access control is simpler, more reliable, and easier to audit

**Implementation:**
- User data is scoped by user_id at the storage layer
- Each agent instance is bound to a specific user
- Cross-user data access is prevented by access control policies, not by redacting data

**Status:** ACCEPTED - No further action needed on privacy/redaction architecture

**Related Documentation:**
- `documentation/scratch-page/integration.md` - Mentions privacy enforcement
- `documentation/working-memory/overview.md` - Describes data scoping

---

## Observability & Monitoring

**Decision:** Turn Trace is the **primary observability mechanism** for Si.

**Rationale:**
- Turn Trace already captures all events, decisions, tool invocations, and errors
- Building monitoring on top of Turn Trace provides a single source of truth
- Eliminates need for separate observability systems
- Enables comprehensive audit trail and debugging

**Implementation:**
- All components emit events to Turn Trace during execution
- Monitoring systems analyze Turn Trace data to generate metrics
- Dashboards and alerting are built on Turn Trace queries
- Real-time monitoring can query Turn Trace while turn is executing

**Status:** ACCEPTED - Turn Trace documentation updated to reflect this

**Related Documentation:**
- `documentation/turn-trace/overview.md` - Observability & Monitoring section
- `documentation/turn-trace/test-conditions.md` - Observability test suite

---

## Daemon Registry Architecture

**Decision:** Daemon Registry is **informational only**. Executor is responsible for daemon lifecycle management.

**Rationale:**
- Keeps registry simple and focused on information storage/retrieval
- Puts orchestration logic in Executor where it belongs
- Executor already has the context to make good decisions about which daemons to create
- Separates concerns: Registry = information, Executor = orchestration

**Implementation:**
- Registry stores daemon type catalog (fixed, code-defined)
- Registry stores daemon instance information (created by Executor)
- Executor queries registry to understand available types
- Executor calls registry to register/update/deregister instances
- Registry enforces turn scoping and executor hierarchy rules

**Status:** ACCEPTED - Daemon Registry documentation created

**Related Documentation:**
- `documentation/daemon-registry/overview.md` - Full specification
- `documentation/daemon-registry/test-conditions.md` - Test requirements
- `documentation/daemon-interface.md` - Updated to reference registry

---

## Turn Scoping & Executor Hierarchy

**Decision:** Daemons are scoped to their turn. Only Executors can communicate across turns, and only with related Executors.

**Rationale:**
- Turn isolation prevents accidental data leaks between parallel turns
- Executor hierarchy enables coordination of parallel work
- Boss/recruit relationships provide clear communication boundaries
- Prevents unrelated turns from interfering with each other

**Implementation:**
- Registry enforces turn scoping: non-executor daemons cannot cross turns
- Executor relationships tracked as boss/recruited pairs
- Executors can communicate with: boss, siblings, recruits
- Executors cannot communicate with unrelated executors

**Status:** ACCEPTED - Daemon Registry specification includes full turn scoping rules

**Related Documentation:**
- `documentation/daemon-registry/overview.md` - Turn Scoping Rules section
- `documentation/daemon-registry/test-conditions.md` - Executor hierarchy tests

---

## Blank Agent Daemon Type

**Decision:** Add a new "Blank Agent" daemon type that Executor can write custom prompts for.

**Rationale:**
- Provides flexibility for Executor to create specialized daemons for specific tasks
- Avoids need to create new daemon types for one-off use cases
- Executor can dynamically generate prompts based on the plan
- Keeps daemon type catalog focused on core, reusable types

**Implementation:**
- Blank Agent is a standard daemon type in the registry
- Executor provides custom prompt when creating Blank Agent instance
- Blank Agent implements standard Daemon Interface (query, get_purpose, get_instructions)
- Blank Agent's purpose/instructions are the custom prompt provided by Executor

**Status:** ACCEPTED - Included in Daemon Type Registry

**Related Documentation:**
- `documentation/daemon-registry/overview.md` - Lists Blank Agent as available type

