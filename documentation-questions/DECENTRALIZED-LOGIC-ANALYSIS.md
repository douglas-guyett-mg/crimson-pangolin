# Analysis: Budget/Sampling/Scoring Logic is Decentralized to Memory Types
**Date**: 2025-10-31  
**Finding**: You're RIGHT! The logic IS in the individual memory types.

---

## The Architecture

The system has **moved away from centralized Budget Enforcement/Sampling/Scoring** and instead pushed this logic into each individual memory type plugin.

---

## Evidence: Individual Memory Types Handle Their Own Logic

### 1. Working Memory Store
**File**: `documentation/working-memory/working-memory-store/overview.md`

**Handles its own**:
- ✅ **Capacity Management** (Lines 24-30): Respects item count and token budget limits
- ✅ **Eviction Policy** (Lines 32-38): Evicts low-importance items first, protects high-importance
- ✅ **Importance Scoring** (Line 52): Each item has `importance_score` (0-1)
- ✅ **TTL Management** (Lines 40-45): Evicts old items based on step-count or wall-time

**Example**: When adding an item that exceeds 4,000 token budget:
1. Working Memory checks capacity
2. Evicts low-importance items first
3. Protects high-importance items (score >= 0.7)
4. Returns eviction information

### 2. Semantic Memory Store
**File**: `documentation/working-memory/semantic-memory-store/overview.md`

**Handles its own**:
- ✅ **Capacity Limits** (Line 85-86): Enforces max_items and max_tokens
- ✅ **Eviction** (Line 85-88): Evicts when capacity exceeded
- ✅ **Importance Protection** (Line 88): High-importance items protected from eviction
- ✅ **Versioning** (Line 84): Manages version retention and lineage

**Example**: When upsert would exceed capacity:
1. Semantic Memory checks capacity
2. Evicts low-importance items
3. Protects high-importance items
4. Returns eviction information

### 3. Episodic Memory Store
**File**: `documentation/working-memory/episodic-memory-store/overview.md`

**Handles its own**:
- ✅ **Append-only semantics**: Manages its own storage model
- ✅ **Soft-delete support**: Manages deletion internally
- ✅ **Chronological ordering**: Manages its own query logic

### 4. Conversation History Plugin
**File**: `documentation/working-memory/conversation-history-plugin/overview.md`

**Handles its own**:
- ✅ **Buffer Management** (Line 143): max_tokens budget per plugin
- ✅ **Eviction** (Line 145): Summarizes evicted utterances
- ✅ **Summarization** (Line 146): Generates summaries within token limits

---

## What This Means

Each memory type is **self-contained and autonomous**:

```
Memory Type Plugin
├── memorize(item) → handles capacity, eviction, importance
├── remember(prompt) → handles search and ranking
├── forget(instructions) → handles deletion
├── summarize(scope, token_limit) → handles summarization
└── get_capacity_info() → reports current state
```

Each plugin:
- Manages its own budget/capacity
- Decides its own eviction policy
- Scores items according to its own logic
- Handles sampling/reduction internally

---

## What Can Be Removed

Since each memory type handles its own logic, the **centralized** components can be removed:

### ❌ Can Remove:
- `documentation/working-memory/budget-enforcement/overview.md` - Centralized budget enforcement
- `documentation/working-memory/write-gating-algorithm/overview.md` - Centralized write gating
- References to "Sampling Engine" as a centralized component
- References to "Scoring Engine" as a centralized component
- References to "Budget Allocation Strategies" (equal, weighted, priority-based)

### ✅ Keep:
- Individual memory type implementations (they have their own logic)
- Memory Orchestrator (coordinates the plugins)
- Memory Plugin Interface (defines the contract)
- Memory Type Registry (discovers plugins)

---

## What Needs Updating

### Test Conditions
**Remove**:
- `documentation/working-memory/test-conditions.md` - Test 2.3 (budget allocation strategy)
- `documentation/working-memory/memory-orchestrator/test-conditions.md` - Test 8.3 (budget allocation strategy)

**Keep**:
- Individual memory type test conditions (they test their own logic)

### Documentation
**Remove**:
- Budget Enforcement overview (centralized component)
- Write Gating Algorithm overview (centralized component)
- References to centralized Sampling/Scoring engines

**Keep**:
- Individual memory type documentation (they document their own logic)
- Memory Orchestrator (it just calls the plugins)

---

## The Cleaner Architecture

```
Memory Orchestrator
├── Calls: Working Memory.remember()
│   └── WM handles: capacity, eviction, importance
├── Calls: Episodic Memory.remember()
│   └── EM handles: chronological search
├── Calls: Semantic Memory.remember()
│   └── SM handles: hybrid search, importance
└── Calls: Conversation History.remember()
    └── CH handles: buffer management
```

No centralized Budget Enforcement, Sampling Engine, or Scoring Engine needed!

---

## Summary

**You were right!** The logic has been decentralized to the individual memory types. Each plugin handles:
- Its own capacity management
- Its own eviction policies
- Its own importance scoring
- Its own sampling/reduction

So we can safely remove:
- Centralized Budget Enforcement
- Centralized Write Gating Algorithm
- References to centralized Sampling/Scoring engines
- References to Budget Allocation Strategies

And keep the individual memory type implementations that already have this logic built in.

