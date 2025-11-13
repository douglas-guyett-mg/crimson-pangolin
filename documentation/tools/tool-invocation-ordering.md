# Tool Invocation Ordering and Data Flow

**Status:** v1.0 (Stable)
**Last Updated:** 2025-11-07
**Purpose:** Define how tools are ordered, how data flows between them, and how Executor adapts execution based on actual results

---

## Overview

Tool invocation is not just about running tools in sequence or parallel. It's about:

1. **Static Planning (FC)**: Plan specifies execution strategy and data flow in natural language
2. **Dynamic Execution (Executor)**: Executor runs tools and adapts based on actual outputs
3. **Adaptive Decision-Making**: When constraints are violated, Executor loops back to FC for new plan
4. **Observability**: All decisions and adaptations are logged to Turn Trace

---

## Tool Invocation Model

### Phase 1: Plan Generation (FC)

Frontal Cortex generates a natural language plan that specifies:

1. **Execution Order**: Which tools run first, second, etc.
2. **Data Flow**: How outputs from one tool become inputs to another (in natural language)
3. **Constraints**: What constraints apply (budgets, limits, etc.)
4. **Parallelization**: Which tools can run in parallel
5. **Fallback Options**: What to do if constraints are violated

**Example Plan:**
```
Step 1: Search for flights to Paris
  - Input: destination=Paris, dates=Nov 15-20
  - Output: flight_price (will be used downstream)
  - Success criteria: Found flights under $3000

Step 2: Search for hotels in Paris
  - Input: destination=Paris, dates=Nov 15-20, max_budget=(initial_budget - flight_price)
  - Output: hotel_price
  - Success criteria: Found hotels within remaining budget
  - Depends on: Step 1 (needs flight_price)

Step 3: Book both
  - Depends on: Step 1 and Step 2 (both succeeded)
```

### Phase 2: Executor Execution

Executor executes the plan with monitoring and adaptation:

```
Executor runs Step 1: Search Flights
  ↓
Flights tool returns: price=$2800
  ↓
Executor monitors: "Flights used $2800, leaving $200 for hotels"
  ↓
Executor calculates: remaining_budget = 3000 - 2800 = $200
  ↓
Executor prepares Step 2 with: max_budget=$200
  ↓
Executor runs Step 2: Search Hotels with max_budget=$200
  ↓
Hotels tool returns: "No hotels found under $200"
  ↓
Executor monitors: "Hotels failed - budget too tight"
  ↓
Executor queries FC: "Hotels failed with $200 budget. What should we do?"
  ↓
FC generates new plan based on memories and situation
```

---

## Data Flow Between Tools

### Natural Language Data Flow

Plans specify data flow in natural language, not formal expressions:

**Good:**
```
"Search for hotels using the remaining budget
 (initial budget - flight cost)"
```

**Not:**
```
max_budget = initial_budget - flights.price
```

### Executor Interpretation

Executor interprets natural language data flow by:

1. **Parsing the description**: Understanding what data is needed
2. **Extracting from previous results**: Getting the actual values
3. **Calculating new inputs**: Computing derived values (e.g., remaining budget)
4. **Passing to next tool**: Providing inputs to downstream tools

### Example: Budget Constraint Propagation

```
Initial budget: $3000

Step 1 output: flights = $2800
  ↓
Executor calculates: remaining = 3000 - 2800 = $200
  ↓
Step 2 input: max_budget = $200
  ↓
Step 2 output: hotels = $150
  ↓
Executor calculates: remaining = 200 - 150 = $50
  ↓
Step 3 input: remaining_budget = $50
```

---

## Adaptive Execution

### Monitoring and Decision-Making

Executor monitors tool outputs and makes decisions based on:

1. **Success/Failure**: Did the tool succeed?
2. **Output Quality**: Is the output usable?
3. **Constraint Satisfaction**: Do results satisfy constraints?
4. **Memories**: What do we know from past similar situations?

### When Constraints Are Violated

If a tool fails or violates constraints, Executor doesn't just fail. It:

1. **Assesses the situation**: What went wrong? Why?
2. **Consults memories**: Have we seen this before? What worked?
3. **Queries FC**: "What should we do now?"
4. **Executes new plan**: FC generates new plan based on situation

**Example:**
```
Hotels failed (no options under $200)
  ↓
Executor assesses: "Budget too tight for hotels"
  ↓
Executor queries FC: "Hotels failed with $200 budget. 
                      What should we do?"
  ↓
FC looks at memories:
  - User's past behavior
  - User's preferences
  - Similar situations
  ↓
FC generates new plan:
  Option A: "Search for cheaper flights"
  Option B: "Ask user to increase budget"
  Option C: "Look at different dates"
  ↓
Executor executes new plan
```

---

## Sub-Turn Spawning for Parallelization

### When to Spawn Sub-Turns

FC identifies tasks that can be independent turns:

```
Plan says:
  "Search flights (can be parallel)
   Search hotels (can be parallel)
   Search rental cars (can be parallel)
   Then compare all three"

Executor recruits three sub-turns:
  ├─ Sub-turn 1: "Search flights to Paris"
  ├─ Sub-turn 2: "Search hotels in Paris"
  └─ Sub-turn 3: "Search rental cars in Paris"
```

### Sub-Turn Communication

Sub-turns communicate through:

1. **Parent context**: Sub-turns can read parent's context (conversation history, working memory)
2. **Results queue**: Sub-turns write results back to parent
3. **Executor hierarchy**: Sub-turns' Executors communicate with parent's Executor

### Sub-Turn Limitations

Sub-turns have intentional limitations:

- ❌ Cannot modify parent's consciousness mandates
- ❌ Cannot modify parent's long-term goals
- ❌ Cannot spawn sub-sub-turns beyond depth limit
- ❌ Cannot exceed parent's remaining budget
- ✅ Can read parent's context
- ✅ Can write results back to parent
- ✅ Can communicate with parent's Executor

See `documentation/daemon-registry/overview.md` for full turn scoping rules.

---

## Turn Trace Recording

All tool invocations and decisions are logged to Turn Trace:

1. **Tool invocation**: Which tool, with what inputs
2. **Tool output**: What the tool returned
3. **Executor decision**: What Executor decided to do next
4. **Adaptation**: If Executor adapted the plan, why and how
5. **FC loop-back**: If Executor queried FC, what context was provided

This enables:
- **Debugging**: Understand why a tool invocation failed
- **Learning**: Learn which execution strategies work well
- **Optimization**: Identify bottlenecks and parallelization opportunities

---

## Future Learning (v2)

Over time, the system learns:

1. **Optimal context for FC**: What context does FC actually need when looping back?
2. **Execution patterns**: Which execution strategies work best for different situations?
3. **Tool combinations**: Which tools work well together?
4. **Constraint handling**: How to handle constraint violations more effectively?

All learning is based on Turn Trace data and memories.

