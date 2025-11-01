# Cleanup Analysis: Budget Allocation, Sampling, and Scoring References
**Date**: 2025-10-31  
**Purpose**: Understand how these concepts are currently used in documentation

---

## Summary

These three concepts are **deeply integrated** into the working memory system design. They're not just mentioned in passing—they're core to how the system works.

---

## 1. Budget Allocation Strategies

### Where It's Used

**File**: `documentation/working-memory/test-conditions.md` (Line 91-110)
- **Test 2.3**: "assemble_context Allocates Budget According to Strategy"
- Tests that budget is divided across memory types using a configured strategy
- References: `budget_allocation_strategy` parameter with examples (equal, weighted, priority-based)

**File**: `documentation/working-memory/memory-orchestrator/test-conditions.md` (Line 45-65)
- **Test 8.3**: Same test as above
- Verifies budget allocation matches configured strategy

**File**: `documentation/working-memory/overview.md` (Line 66)
- Lists "Memory Orchestrator" as managing "budget allocation"

**File**: `documentation/working-memory/initialization-flow.md` (Line 282)
- Records "budget allocation" as part of initialization

**File**: `documentation/working-memory/budget-enforcement/overview.md` (Line 160-175)
- **Example Scenario**: Shows orchestrator allocating 3000 to working memory, 4000 to episodic, 3000 to semantic
- This is the allocation strategy in action

### What It Does

The Memory Orchestrator needs to divide a total budget (e.g., 10,000 tokens) across multiple memory types. The strategy determines HOW it divides:
- **Equal**: Each type gets 10,000 / 3 = 3,333 tokens
- **Weighted**: Types get different percentages based on importance
- **Priority-based**: High-priority types get more budget

### Impact of Removing

If you remove this, the Memory Orchestrator can't be tested or implemented because:
- No way to specify how budget should be divided
- Test conditions become incomplete
- Implementation would be ambiguous

---

## 2. Sampling Engine

### Where It's Used

**File**: `documentation/working-memory/budget-enforcement/overview.md`
- **Lines 50-56**: "Sampling Policies" section
  - Random Sampling
  - Importance-Based Sampling
  - Recency-Based Sampling
  - Deterministic Sampling
- **Lines 141**: References "Sampling Engine" as component that "Performs sampling when needed"
- **Lines 169-170**: Example scenario shows sampling applied when budget exceeded

### What It Does

When a memory operation would exceed budget, the Sampling Engine decides what to keep and what to discard:
- Random: Keep 85% of item randomly
- Importance-Based: Keep high-importance parts
- Recency-Based: Keep recent parts
- Deterministic: Use fixed seed for reproducibility

### Impact of Removing

If you remove this, Budget Enforcement can't work because:
- No mechanism to handle items that exceed budget
- Soft limits become meaningless
- Example scenario (lines 169-170) breaks

---

## 3. Scoring Engine

### Where It's Used

**File**: `documentation/working-memory/write-gating-algorithm/overview.md`
- **Stage 2: Scoring** (Lines 36-49)
  - Scores items for importance, novelty, recency, relevance, source
  - Uses formula: `score = (importance * 0.4) + (novelty * 0.2) + (recency * 0.2) + (relevance * 0.1) + (source_weight * 0.1)`
- **Stage 3: Routing** (Lines 65-68)
  - Uses scores to decide routing:
    - High Score (>0.7): Route to additional stores
    - Low Score (<0.3): Route only to Episodic Memory

### What It Does

The Scoring Engine evaluates each memory item to determine its value:
- Combines multiple factors (importance, novelty, recency, relevance, source)
- Produces a score (0-1) that influences routing and eviction decisions
- High-score items get preserved; low-score items get evicted first

### Impact of Removing

If you remove this, Write Gating can't work because:
- No way to decide which items are important
- Routing decisions become arbitrary
- Eviction cascade (Stage 4) can't prioritize what to keep
- Test conditions for write-gating break

---

## The Interconnection

These three concepts form a chain:

```
Budget Allocation Strategy
    ↓
    Divides total budget across memory types
    ↓
Sampling Engine
    ↓
    When items exceed allocated budget, samples them
    ↓
Scoring Engine
    ↓
    Scores items to decide what to keep/evict
```

---

## Decision Points

### Option 1: Keep All Three
- System works as designed
- All test conditions are valid
- Implementation is clear

### Option 2: Remove All Three
- Budget Enforcement becomes incomplete
- Write Gating becomes incomplete
- Multiple test conditions become invalid
- System design is unclear

### Option 3: Simplify (Keep Scoring, Remove Allocation Strategies & Sampling)
- Scoring still works (items get scored)
- But: How does budget get divided? (allocation strategy needed)
- But: What happens when items exceed budget? (sampling needed)
- Partial removal creates gaps

---

## Recommendation

**These three concepts are not optional extras—they're core to how the system works.**

If you want to remove them, you need to:
1. Redesign Budget Enforcement (how does it work without sampling?)
2. Redesign Write Gating (how does it work without scoring?)
3. Redesign Memory Orchestrator (how does it allocate budget?)

**Better approach**: Keep them as-is, or explicitly redesign the system to work differently.

---

## Files That Would Need Updates If Removed

**Would Break**:
- `documentation/working-memory/test-conditions.md` (Test 2.3)
- `documentation/working-memory/memory-orchestrator/test-conditions.md` (Test 8.3)
- `documentation/working-memory/budget-enforcement/overview.md` (entire example scenario)
- `documentation/working-memory/write-gating-algorithm/overview.md` (Stage 2 & 3)

**Would Need Redesign**:
- `documentation/working-memory/budget-enforcement/` (entire component)
- `documentation/working-memory/write-gating-algorithm/` (entire component)
- `documentation/working-memory/memory-orchestrator/` (budget allocation logic)

