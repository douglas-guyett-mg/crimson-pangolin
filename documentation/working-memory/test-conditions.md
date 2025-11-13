# Working Memory System - Test Conditions

## Test Suite: System Initialization

### Test 1.1: System Initializes with Registered Memory Types
**Objective**: Verify that the working memory system initializes with all configured memory types

**Setup**:
- Configure system with 3 memory types (episodic, semantic, working)
- Initialize system

**Steps**:
1. Initialize working memory system
2. Verify all configured memory types are registered
3. Verify each memory type is accessible

**Expected Outcome**:
- All 3 memory types are registered and accessible
- Each memory type is in ready state

**Verification**:
- System.get_registered_types() returns all 3 types
- Each type responds to get_capacity_info()

---

### Test 1.2: System Initializes with Memory Types Configuration
**Objective**: Verify that the system initializes with configured memory types

**Setup**:
- Configure system with episodic, semantic, working, and conversation history memory types
- Initialize system

**Steps**:
1. Initialize system
2. Check memory type status

**Expected Outcome**:
- All configured memory types are initialized
- Each memory type is ready for operations

**Verification**:
- get_memory_types() returns all configured types
- Each type responds to operations

---

## Test Suite: Context Assembly

### Test 2.1: assemble_context Queries All Memory Types
**Objective**: Verify that context assembly queries all registered memory types

**Setup**:
- Initialize system with 3 memory types
- Populate each with items
- Call assemble_context()

**Steps**:
1. Call assemble_context(prompt="What happened?", budget=2000)
2. Verify all memory types were queried

**Expected Outcome**:
- All 3 memory types are queried
- Results are combined

**Verification**:
- Context includes items from all 3 memory types
- Query log shows all 3 types were accessed

---

### Test 2.2: assemble_context Returns Token Count
**Objective**: Verify that context assembly returns accurate token count

**Setup**:
- Initialize system with memory types
- Populate memory types with items

**Steps**:
1. Call assemble_context(prompt="What happened?")
2. Verify token count is returned

**Expected Outcome**:
- Returned context includes token count
- Token count is accurate

**Verification**:
- context.token_count is returned
- Token count matches actual content

**Note**: LLM call budget enforcement is handled by LLM Budget System, not memory stores. See `documentation/llm-budget-system/overview.md`.

---

## Test Suite: Memory Routing

### Test 3.1: memorize Routes Items to Correct Memory Types
**Objective**: Verify that items are routed to appropriate memory types

**Setup**:
- Initialize system with episodic, semantic, and working memory types
- Create items of different types

**Steps**:
1. Memorize an "observation" item
2. Verify it's routed to episodic and working memory
3. Memorize a "fact" item
4. Verify it's routed to semantic memory

**Expected Outcome**:
- Items are routed to appropriate stores based on type

**Verification**:
- Observation is in episodic and working memory
- Fact is in semantic memory

---

### Test 3.2: memorize Respects Budgets
**Objective**: Verify that memorize respects budget constraints

**Setup**:
- Initialize system with limited budget
- Attempt to write items exceeding budget

**Steps**:
1. Write items until budget is exceeded
2. Verify behavior

**Expected Outcome**:
- Write fails or items are evicted to stay within budget

**Verification**:
- Total tokens never exceed budget

---

## Test Suite: Tool Tracking

### Test 4.1: track_tool_invocation Records Tool Call
**Objective**: Verify that tool invocations are recorded

**Setup**:
- Initialize system
- Prepare tool invocation data

**Steps**:
1. Call track_tool_invocation(tool_name="search", parameters={...}, result={...})
2. Verify it's recorded

**Expected Outcome**:
- Tool invocation is recorded in episodic memory
- Item is retrievable

**Verification**:
- Episodic memory contains the tool trace
- Tool name and parameters are recorded

---

### Test 4.2: track_tool_invocation Records Errors
**Objective**: Verify that tool errors are recorded

**Setup**:
- Call track_tool_invocation with error

**Steps**:
1. Track tool invocation with error
2. Verify error is recorded

**Expected Outcome**:
- Tool error is recorded in episodic memory

**Verification**:
- Episodic memory item includes error message

---

## Test Suite: Commit Operation

### Test 5.1: commit Finalizes All Writes
**Objective**: Verify that commit finalizes all pending writes

**Setup**:
- Write multiple items
- Call commit

**Steps**:
1. Write items
2. Call commit()
3. Verify all writes are persisted

**Expected Outcome**:
- All writes are finalized
- Items are persisted

**Verification**:
- All written items are retrievable after commit

---

### Test 5.2: commit Exports Episodic Memory
**Objective**: Verify that commit exports episodic memory

**Setup**:
- Write items to episodic memory
- Call commit

**Steps**:
1. Write items
2. Call commit()
3. Verify episodic export

**Expected Outcome**:
- Episodic memory items are exported

**Verification**:
- Commit returns episodic memory export
- Export includes all episodic items

---

### Test 5.3: commit Exports Semantic Memory
**Objective**: Verify that commit exports semantic memory

**Setup**:
- Write items to semantic memory
- Call commit

**Steps**:
1. Write items
2. Call commit()
3. Verify semantic export

**Expected Outcome**:
- Semantic memory items are exported

**Verification**:
- Commit returns semantic memory export
- Export includes all semantic items

---

## Test Suite: Integration

### Test 6.1: Full Turn Lifecycle
**Objective**: Verify complete turn lifecycle from context assembly to commit

**Setup**:
- Initialize system with all memory types
- Populate with initial memories

**Steps**:
1. Call assemble_context()
2. Track tool invocation
3. Memorize response
4. Call commit()

**Expected Outcome**:
- All operations complete successfully
- Exports are generated

**Verification**:
- Context is assembled
- Tool is tracked
- Response is memorized
- Commit succeeds with exports

---

### Test 6.2: Multiple Concurrent Turns
**Objective**: Verify behavior with multiple concurrent turns

**Setup**:
- Initialize system
- Start multiple turns concurrently

**Steps**:
1. Start turn 1, assemble context
2. Start turn 2, assemble context
3. Both turns memorize items
4. Both turns commit

**Expected Outcome**:
- Operations are handled correctly
- No data corruption

**Verification**:
- All operations complete successfully
- Data is consistent across turns

