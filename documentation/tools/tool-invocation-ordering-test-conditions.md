# Tool Invocation Ordering - Test Conditions

**Status:** Specification v1.0  
**Date:** 2025-11-07

---

## Test Suite: Data Flow Between Tools

### Test: Natural Language Data Flow Interpretation
- **Given:** Plan specifies "Search hotels using remaining budget (initial - flights)"
- **When:** Executor executes the plan
- **Then:**
  - Executor extracts flight_price from Step 1 output with 100% accuracy (exact numeric value)
  - Executor calculates remaining_budget = initial - flight_price (verified: calculation error = 0)
  - Executor passes remaining_budget to hotel search tool (verified: parameter value matches calculated value exactly)
  - Hotel search tool receives correct budget value (verified: received_value == remaining_budget within 0.01 tolerance)

### Test: Constraint Propagation
- **Given:** Initial budget=$3000, flights=$2800
- **When:** Executor calculates remaining budget
- **Then:**
  - remaining_budget = 3000 - 2800 = $200 (exact calculation, verified to 0.01 precision)
  - Hotel search tool receives max_budget=$200 (verified: parameter equals 200.00)
  - Hotel search respects the constraint (verified: all returned results have price <= 200.00)

### Test: Multiple Data Dependencies
- **Given:** Plan with 3 sequential steps, each depends on previous output
- **When:** Executor executes all steps
- **Then:**
  - Step 1 output → Step 2 input (verified: 100% of Step 1 output fields required by Step 2 are passed correctly)
  - Step 2 output → Step 3 input (verified: 100% of Step 2 output fields required by Step 3 are passed correctly)
  - All data flows correctly through chain (verified: no data loss, type mismatches, or missing fields)

---

## Test Suite: Adaptive Execution

### Test: Tool Success with Constraint Satisfaction
- **Given:** Hotels found for $150 (within $200 budget)
- **When:** Executor monitors output
- **Then:**
  - Executor recognizes success
  - Executor continues to next step
  - No adaptation needed

### Test: Tool Failure - Constraint Violated
- **Given:** Hotels search fails (no options under $200)
- **When:** Executor monitors output
- **Then:**
  - Executor detects failure
  - Executor assesses: "Budget too tight"
  - Executor queries FC for new plan
  - FC generates new plan (cheaper flights, increase budget, different dates)

### Test: Executor Queries FC with Context
- **Given:** Hotels failed with $200 budget
- **When:** Executor queries FC
- **Then:**
  - Executor provides: situation description (>= 50 chars), constraint that failed (specific field name + value), memories (>= 3 relevant items)
  - FC receives complete context (verified: all 3+ context components present in FC query)
  - FC generates informed new plan (verified: plan references at least 1 context element from executor query)

### Test: FC Generates Adaptive Plan
- **Given:** FC receives context about failed hotels
- **When:** FC generates new plan
- **Then:**
  - Plan includes multiple options (cheaper flights, ask user, different dates)
  - Plan is based on memories and situation
  - Plan is executable by Executor

### Test: Executor Executes New Plan
- **Given:** FC generated new plan after hotels failed
- **When:** Executor executes new plan
- **Then:**
  - Executor follows new plan
  - Executor monitors new execution
  - Executor adapts if needed

---

## Test Suite: Sub-Turn Spawning

### Test: Identify Parallelizable Tasks
- **Given:** Plan specifies "Search flights, hotels, rental cars (all parallel)"
- **When:** Executor analyzes plan
- **Then:**
  - Executor identifies 3 parallelizable tasks
  - Executor decides to spawn 3 sub-turns

### Test: Spawn Sub-Turns
- **Given:** Executor decides to spawn 3 sub-turns
- **When:** Executor recruits sub-turns
- **Then:**
  - Sub-turn 1 created for flights search
  - Sub-turn 2 created for hotels search
  - Sub-turn 3 created for rental cars search
  - All 3 run in parallel

### Test: Sub-Turn Context Access
- **Given:** Sub-turn 1 (flights search) is running
- **When:** Sub-turn 1 needs context
- **Then:**
  - Sub-turn 1 can read parent's conversation history
  - Sub-turn 1 can read parent's working memory
  - Sub-turn 1 cannot modify parent's consciousness

### Test: Sub-Turn Results Collection
- **Given:** All 3 sub-turns completed
- **When:** Parent Executor collects results
- **Then:**
  - Parent receives flights results from sub-turn 1
  - Parent receives hotels results from sub-turn 2
  - Parent receives rental cars results from sub-turn 3
  - Parent continues with comparison step

### Test: Sub-Turn Depth Limit
- **Given:** Sub-turn 1 tries to spawn sub-sub-turn
- **When:** Sub-turn 1 requests sub-turn spawning
- **Then:**
  - System checks depth limit
  - If depth < max: sub-sub-turn is allowed
  - If depth >= max: sub-sub-turn is rejected

### Test: Sub-Turn Budget Isolation
- **Given:** Parent has 10,000 tokens, spawns 3 sub-turns
- **When:** Sub-turns execute
- **Then:**
  - Each sub-turn has its own budget allocation
  - Sub-turn cannot exceed its allocation
  - Parent's remaining budget is tracked

---

## Test Suite: Turn Trace Recording

### Test: Tool Invocation Logged
- **Given:** Executor calls flight search tool
- **When:** Tool invocation completes
- **Then:**
  - Turn Trace records: tool_name, inputs, outputs, latency
  - Turn Trace records: success/failure status
  - Turn Trace records: any errors

### Test: Executor Decision Logged
- **Given:** Executor decides to adapt plan
- **When:** Executor queries FC
- **Then:**
  - Turn Trace records: why adaptation was needed
  - Turn Trace records: what context was provided to FC
  - Turn Trace records: what new plan FC generated

### Test: Data Flow Logged
- **Given:** Executor calculates remaining_budget
- **When:** Executor passes to next tool
- **Then:**
  - Turn Trace records: calculation (initial - flights)
  - Turn Trace records: result value
  - Turn Trace records: which tool received it

### Test: Replay from Turn Trace
- **Given:** Completed turn with complex tool invocations
- **When:** System replays from Turn Trace
- **Then:**
  - Replay follows same execution order
  - Replay produces same results
  - Replay is deterministic

---

## Test Suite: Integration

### Test: End-to-End Budget-Constrained Travel
- **Given:** User asks "Book flight and hotel to Paris, $3000 budget"
- **When:** Turn executes
- **Then:**
  - FC generates plan: flights first, then hotels with remaining budget
  - Executor runs flights search: $2800
  - Executor calculates remaining: $200
  - Executor runs hotels search with $200 budget
  - If hotels fail: Executor queries FC for new plan
  - System adapts and continues

### Test: Parallel Sub-Turns with Data Aggregation
- **Given:** Plan specifies parallel searches then comparison
- **When:** Turn executes
- **Then:**
  - 3 sub-turns spawn and run in parallel
  - All 3 complete and return results
  - Parent Executor receives all results
  - Parent runs comparison tool with aggregated data
  - Comparison produces final recommendation

