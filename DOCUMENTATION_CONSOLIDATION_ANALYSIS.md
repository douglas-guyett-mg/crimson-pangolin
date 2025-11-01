# Documentation Consolidation Analysis

## Overview
After moving `agent-plans/` to `documentation/`, we now have duplicate/overlapping documentation. This analysis compares old vs. new files to determine consolidation strategy.

---

## 1. WORKING MEMORY ROOT LEVEL

### OLD: `documentation/working-memory.md` (243 lines)
- **Title**: "Working Memory for Agents — Technical Plan (Platform-Independent)"
- **Status**: Draft v0.1
- **Scope**: Detailed technical specification with implementation guidance
- **Content**:
  - Purpose, scope, principles
  - Background & paper-inspired concepts
  - Terminology definitions
  - High-level architecture diagram
  - Component responsibilities (detailed)
  - Data schemas (MemoryItem, MemoryBundle, etc.)
  - Read/write/eviction policies
  - Retrieval algorithms (hybrid search, re-ranking)
  - Compaction & summarization strategies
  - Budget enforcement mechanisms
  - Testing requirements (scattered throughout)

### OLD: `documentation/working-memory/README.md` (124 lines)
- **Title**: "Working Memory System — High-Level Overview"
- **Status**: Draft v0.1
- **Scope**: High-level conceptual overview
- **Content**:
  - Purpose & design tenets
  - Conceptual model (WM, memory types, FC, summarizer, orchestrator)
  - High-level flow per turn (5 steps)
  - Interfaces (behavioral, not vendor-specific)
  - Test requirements (scattered)

### NEW: `documentation/working-memory/overview.md` (146 lines)
- **Title**: "Si Working Memory System — Overview"
- **Status**: Recently created
- **Scope**: Structured overview following documentation guidelines
- **Content**:
  - Purpose & key concepts
  - Core architecture with all 15+ sub-components
  - Key concepts (Memory Item, Budget Allocation, Write Gating, Context Assembly)
  - Major operations (assemble_context, track_tool_invocation, memorize, commit, get_budget_status)
  - Design principles
  - Interaction with other systems
  - Complete example scenario
  - Related documents (cross-links)

### ANALYSIS
- **Overlap**: All three cover the same system but at different levels
- **OLD files**: More detailed, implementation-focused, scattered test requirements
- **NEW file**: Structured, follows guidelines, includes example scenario, cleaner organization
- **Recommendation**: NEW file is better structured but may be missing some technical depth from OLD files

---

## 2. MEMORY ORCHESTRATOR

### OLD: `documentation/working-memory/memory-orchestrator.md` (191 lines)
- **Title**: "Memory Orchestrator — Consciousness-Aware Context Assembly"
- **Status**: Draft v0.1
- **Scope**: Detailed spec with consciousness alignment
- **Content**:
  - Purpose & mental model (mini-agent)
  - Consciousness alignment (charter, mandates, modes, capabilities)
  - Prompt assembly design pattern
  - Detailed interfaces (behavioral)
  - Budget enforcement
  - Tool tracking & Turn Trace integration
  - Commit logic
  - Test requirements (scattered)

### NEW: `documentation/working-memory/memory-orchestrator/overview.md` (179 lines)
- **Title**: "Memory Orchestrator — Overview"
- **Status**: Recently created
- **Scope**: Structured overview following guidelines
- **Content**:
  - Purpose & key responsibilities
  - Core operations (assemble_context, track_tool_invocation, memorize, commit, get_budget_status)
  - Budget allocation & enforcement
  - Write gating algorithm
  - Integration with other components
  - Design principles
  - Related documents (cross-links)

### ANALYSIS
- **Overlap**: Both cover the same component
- **OLD file**: Includes consciousness alignment (unique content)
- **NEW file**: Follows structured guidelines, cleaner organization
- **Recommendation**: NEW file is better structured but MISSING consciousness alignment content

---

## 3. TURN TRACE

### OLD: `documentation/working-memory/turn-trace.md` (169 lines)
- **Title**: "Turn Trace — Daemonized Brain Process"
- **Status**: Draft v0.1
- **Scope**: Detailed spec with brain-as-daemons model
- **Content**:
  - Purpose & mental model (mini-agents/daemons)
  - 11 brain components (WM-Orchestrator, Planner, Action Selector, etc.)
  - Communication style (AgentMessage)
  - Data model (Trace, Span, Event, Event kinds)
  - Protocol & ordering
  - Interfaces (behavioral)
  - Queries (read-side)
  - Test requirements (scattered)

### NEW: `documentation/working-memory/turn-trace-integration/overview.md` (162 lines)
- **Title**: "Turn Trace Integration"
- **Status**: Recently created
- **Scope**: Structured overview of logging/tracing
- **Content**:
  - Purpose (debugging, auditing, analysis, transparency, monitoring)
  - Logged operations (memory, gating, orchestration, integration)
  - Log structure & format
  - Log levels (DEBUG, INFO, WARN, ERROR, CRITICAL)
  - Operations (log_operation, query_logs, etc.)
  - Integration with other systems

### ANALYSIS
- **Overlap**: Both cover tracing/logging but from different angles
- **OLD file**: Focuses on brain-as-daemons model, event types, protocol
- **NEW file**: Focuses on what gets logged and how to query logs
- **Recommendation**: These are COMPLEMENTARY, not duplicates. OLD has architectural model, NEW has operational logging details

---

## 4. OTHER MOVED FILES (No Conflicts)

### Consciousness System
- `documentation/consciousness/` (6 files) - No existing documentation to conflict with
- Files: README, capabilities_catalog, core_identity, governance_and_evolution, mandates_and_objectives, mode_framework

### Frontal Cortex System
- `documentation/frontal-cortex/` (3 files) - No existing documentation to conflict with
- Files: README, action_types.yaml, question-story-framework.md

### Tools System
- `documentation/tools/` (3 files) - No existing documentation to conflict with
- Files: si_tools_architecture.md, tool_io_contract.yaml, tool_manifest_schema.yaml

### Root-Level Files
- `documentation/si_core_engineering_tennants.md` - Core tenants (no conflict)
- `documentation/si_chat_assistant_coordination_plan.md` - Chat coordination plan (no conflict)
- `documentation/turn.md` - Turn specification (no conflict)

---

## CONSOLIDATION STRATEGY OPTIONS

### Option A: Keep Both (No Consolidation)
- **Pros**: No information loss, old detailed specs available for reference
- **Cons**: Confusing duplication, maintenance burden, violates DRY principle

### Option B: Replace Old with New
- **Pros**: Clean structure, follows guidelines, single source of truth
- **Cons**: Lose some technical depth and consciousness alignment details

### Option C: Merge (Recommended)
- **Pros**: Preserve all valuable content, follow guidelines, single source of truth
- **Cons**: More work upfront
- **Approach**:
  1. Keep NEW structured files as primary documentation
  2. Extract unique content from OLD files (consciousness alignment, brain-as-daemons model, detailed protocols)
  3. Integrate extracted content into NEW files or create supplementary docs
  4. Delete OLD files once content is migrated

### Option D: Archive Old Files
- **Pros**: Preserve history, clean current docs
- **Cons**: Requires creating archive structure
- **Approach**: Move old files to `documentation-archive/` for reference

---

## RECOMMENDATION

**Option C (Merge)** is recommended because:
1. NEW files follow the documentation guidelines (overview.md, test-conditions.md structure)
2. OLD files contain valuable technical depth and architectural insights
3. Consciousness alignment in old Memory Orchestrator spec is important
4. Brain-as-daemons model in old Turn Trace spec is important
5. Merging preserves all information while maintaining clean structure

**Next Steps**:
1. Review each merge point with user
2. Extract unique content from OLD files
3. Integrate into NEW files or create supplementary documentation
4. Delete OLD files once migration is complete
5. Update rules file to reference `documentation/` only

