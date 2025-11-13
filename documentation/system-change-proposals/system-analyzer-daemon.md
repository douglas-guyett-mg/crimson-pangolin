# System Analyzer Daemon

**Purpose:** Analyze flagged changes from turn traces, deduplicate proposals, and generate rich proposals for human review.

---

## Overview

The System Analyzer Daemon runs separately from turn execution (can query historic turns). It:
1. Reads flagged changes from turn traces
2. Deduplicates against existing proposals
3. Generates rich proposals with evidence and rationale
4. Stores proposals for human review

---

## Daemon Interface

The System Analyzer Daemon implements the Daemon Interface:

```
get_purpose() -> string
  Returns: "Analyze flagged system changes and generate proposals for human review"

get_instructions() -> string
  Returns: Specifications for analysis and proposal generation

query(prompt: string, metadata: object) -> Future<Result>
  Enables other daemons to query for proposal information
```

---

## Core Responsibilities

### 1. Read Flagged Changes
**What:** Query turn traces for flagged changes  
**When:** On demand (can run separately from turn execution)  
**How:** Query turn traces with `system_change_flags` array

### 2. Deduplicate Proposals
**What:** Check if proposal already exists for this change  
**When:** Before generating a new proposal  
**How:** Compare against existing proposals, identify similar ones

### 3. Aggregate Evidence
**What:** Collect evidence across multiple turns  
**When:** When generating a proposal  
**How:** Count flags, track severity, identify patterns

### 4. Generate Proposals
**What:** Create rich proposals with decision-support information  
**When:** After deduplication  
**How:** Follow proposal structure, include evidence and rationale

### 5. Store Proposals
**What:** Persist proposals for human review  
**When:** After generation  
**How:** Store in accessible location (DB/files/etc - TBD)

---

## Analysis Process

### Step 1: Query Flagged Changes
```
Query turn traces for:
  - All turns with system_change_flags
  - Filter by flag_type (optional)
  - Filter by date range (optional)
  - Filter by severity (optional)
```

### Step 2: Group by Change Type
```
Group flags by:
  - flag_type (new_action_type, new_tool, etc.)
  - Specific change (e.g., "parallel_task" action type)
  - Source daemon
```

### Step 3: Deduplicate
```
For each group:
  1. Check if proposal already exists for this exact change
  2. Check if similar proposal exists
  3. If similar proposal exists:
     - Link as related_proposals
     - Update existing proposal with new evidence
  4. If no proposal exists:
     - Generate new proposal
```

### Step 4: Aggregate Evidence
```
For each proposal:
  - Count total flags
  - Track severity distribution
  - Identify source daemons
  - List affected turns
  - Extract use cases
  - Identify affected users
```

### Step 5: Generate Proposal
```
Create proposal with:
  - Title and description
  - Rationale based on evidence
  - Evidence section (counts, severity, turns, users)
  - Details section (type-specific information)
  - Suggested documentation structure
  - Related proposals (if any)
```

### Step 6: Store Proposal
```
Store proposal in accessible location:
  - Assign proposal_id
  - Set status to "pending"
  - Record created_at timestamp
  - Make available for human review
```

---

## Deduplication Logic

### Exact Match
```
If proposal exists with same:
  - proposal_type
  - title
  - key details (e.g., action_type_name, tool_name)
Then:
  - Update existing proposal with new evidence
  - Don't create new proposal
```

### Similar Match
```
If proposal exists with similar:
  - proposal_type
  - purpose (even if different name)
Then:
  - Create new proposal
  - Link as related_proposals
  - Include note about similarity
```

### No Match
```
If no existing proposal:
  - Create new proposal
  - Set status to "pending"
```

---

## Initial Mandate (v1)

**Scope:** Process all flagged changes, generate proposals for each  
**Frequency:** Can run on demand or on schedule  
**Deduplication:** Check for existing proposals before creating new ones

```
1. Query turn traces for flagged changes
   - Get all turns with system_change_flags
   - Group by flag_type and specific change

2. For each group:
   - Check if proposal already exists
   - If yes: update with new evidence
   - If no: generate new proposal

3. Aggregate evidence:
   - Count flags
   - Track severity
   - Identify patterns
   - Extract use cases

4. Generate proposal:
   - Include all evidence
   - Include suggested documentation
   - Include decision-support information

5. Store proposal:
   - Make available for human review
   - Track status (pending, approved, rejected, implemented)
```

---

## Future Enhancements (v2+)

As we learn what makes sense:

1. **Smarter Deduplication**
   - Use semantic similarity to find related proposals
   - Suggest merging similar proposals
   - Track proposal lineage

2. **Prioritization**
   - Rank proposals by frequency, severity, impact
   - Suggest which proposals to review first

3. **Trend Analysis**
   - Identify emerging patterns
   - Suggest proactive improvements
   - Track system evolution

4. **Integration with Code Generation**
   - Generate code stubs from proposals
   - Track implementation status
   - Link PRs to proposals

---

## API

### Query Flagged Changes
```
query_flagged_changes(
  flag_type?: string,               // Optional filter
  severity?: string,                // Optional filter
  since?: ISO8601,                  // Optional date filter
  limit?: number                    // Optional limit
) -> Future<Flag[]>
```

### Generate Proposal
```
generate_proposal(
  flags: Flag[]                     // Flags to aggregate
) -> Future<Proposal>
```

### Store Proposal
```
store_proposal(
  proposal: Proposal
) -> Future<string>                 // Returns proposal_id
```

### Check for Existing Proposal
```
find_existing_proposal(
  proposal_type: string,
  key_details: object
) -> Future<Proposal | null>
```

---

## Test Conditions

See `documentation/system-change-proposals/test-conditions.md` for detailed test conditions.

Key tests:
1. Reads flagged changes from turn traces
2. Deduplicates against existing proposals
3. Aggregates evidence correctly
4. Generates proposals with all required fields
5. Stores proposals persistently
6. Handles multiple flags in single turn
7. Handles multiple turns with same flag type

---

## Next Steps

1. Define test conditions for System Analyzer Daemon
2. Define test conditions for Change Flagging Daemon
3. Create integration documentation

