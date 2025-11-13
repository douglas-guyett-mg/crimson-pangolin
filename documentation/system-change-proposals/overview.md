# System Change Proposals

**Status:** v0.1 (Draft)
**Last Updated:** 2025-11-07
**Purpose:** Enable SI to identify, propose, and track system improvements based on patterns observed during turn execution.

---

## Overview

SI operates within a defined system (action types, tools, capabilities, consciousness mandates, etc.). As SI executes turns, it may encounter situations where the system needs to evolve:
- A tool doesn't exist but would be useful
- An action type is needed that isn't defined
- A capability is missing
- An existing tool needs enhancement

**System Change Proposals** is a meta-level improvement system that enables SI to:
1. **Flag** changes needed during turn execution
2. **Analyze** flagged changes to identify patterns
3. **Propose** system improvements with rich evidence
4. **Surface** proposals for human review and decision-making

---

## Architecture

### Three Key Components

**1. Change Flagging Daemon**
- Runs during turn execution
- Monitors daemons (especially FC) for suggestions of missing capabilities
- Creates flags in turn trace when changes are needed
- Initially: Captures FC suggestions for new action types, other daemon suggestions for missing tools/capabilities
- Future: May run LLM analysis on traces to generate proactive suggestions

**2. System Analyzer Daemon**
- Runs separately from turn execution (can query historic turns)
- Reads flagged changes from turn traces
- Deduplicates: checks if proposal already exists for this change or similar changes
- Generates rich proposals with evidence, rationale, and decision-support information
- Stores proposals for human review

**3. Human Review & Implementation**
- Human reviews proposals
- Decides whether to approve and how to document the change
- Generates code based on approved documentation
- (Code generation is out of scope for this system)

---

## Key Design Principles

1. **Documentation-First**: Proposals include suggested documentation structure that humans can refine
2. **Evidence-Based**: Proposals include evidence (which turns flagged this, how many times, severity)
3. **Deduplication**: System Analyzer checks for existing proposals to avoid redundant work
4. **Extensible**: Initially simple (FC action type suggestions), can expand to other proposal types
5. **Auditable**: All proposals tracked with creation date, evidence, and rationale
6. **Technology-Agnostic**: Storage mechanism (DB/files/etc) is a separate decision

---

## Data Flow

```
Turn Execution
    ↓
Change Flagging Daemon detects missing capability
    ↓
Flag stored in Turn Trace
    ↓
System Analyzer Daemon (runs separately)
    ↓
Reads flagged changes from historic turns
    ↓
Deduplicates against existing proposals
    ↓
Generates rich proposal with evidence
    ↓
Proposal stored for human review
    ↓
Human reviews, decides, writes documentation
    ↓
Code generated from documentation
```

---

## What Gets Proposed?

Initially:
- New action types (FC suggesting types that don't exist)
- New tools (daemons suggesting tools that would help)
- Tool enhancements (suggestions to improve existing tools)

Future (as we learn what makes sense):
- Consciousness mandate changes
- Memory policy changes
- Daemon architecture changes
- Other system improvements

---

## Next Steps

1. Define Change Needed Flag structure (what goes in turn trace)
2. Define Proposal data structure (what gets stored for human review)
3. Specify Change Flagging Daemon (how it detects changes needed)
4. Specify System Analyzer Daemon (how it processes flags and generates proposals)
5. Define test conditions for both daemons

