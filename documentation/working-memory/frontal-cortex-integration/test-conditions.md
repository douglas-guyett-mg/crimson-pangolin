# Frontal Cortex Integration - Test Conditions

## Test Suite: Context Assembly

### Test 15.1: Assemble Context for Planning
**Objective**: Verify that context is assembled for planning

**Setup**:
- Create memory with relevant items
- Request context assembly

**Steps**:
1. Call assemble_context(prompt="weather_query", budget=2000, constraints={})
2. Verify context is assembled

**Expected Outcome**:
- Context is returned
- Context includes relevant items

**Verification**:
- Context contains >= 3 items (memory objects)
- At least 80% of returned items have semantic relevance >= 0.7 to "weather" query
- Context.token_count is reported and non-zero

---

### Test 15.2: Context Respects Token Limit
**Objective**: Verify that context respects token limit

**Setup**:
- Create memory with many items
- Set budget=1000

**Steps**:
1. Call assemble_context(prompt, budget=1000, constraints={})
2. Verify token count

**Expected Outcome**:
- Context token count <= 1000

**Verification**:
- `context.token_count` <= 1000

---

### Test 15.3: Context Includes Goal Context
**Objective**: Verify that context includes goals

**Setup**:
- Create goals in memory
- Request context

**Steps**:
1. Write goals to memory
2. Call assemble_context(task)
3. Verify goals are included

**Expected Outcome**:
- Goals are included in context

**Verification**:
- Context includes goal items

---

### Test 15.4: Context Includes State Context
**Objective**: Verify that context includes current state

**Setup**:
- Create state items in memory
- Request context

**Steps**:
1. Write state items to memory
2. Call assemble_context(task)
3. Verify state is included

**Expected Outcome**:
- State is included in context

**Verification**:
- Context includes state items

---

### Test 15.5: Context Includes Skill Context
**Objective**: Verify that context includes applicable skills

**Setup**:
- Create skills in memory
- Request context for task

**Steps**:
1. Write skills to memory
2. Call assemble_context(task="weather_query")
3. Verify skills are included

**Expected Outcome**:
- Applicable skills are included

**Verification**:
- Context includes weather-related skills

---

## Test Suite: Tool Invocation Tracking

### Test 15.6: Track Tool Invocation
**Objective**: Verify that tool invocations are tracked

**Setup**:
- Execute tool invocation
- Track it

**Steps**:
1. Call track_tool_invocation(tool="weather_api", parameters={location: "Paris"}, result={forecast: "sunny"})
2. Verify invocation is tracked

**Expected Outcome**:
- Invocation is recorded

**Verification**:
- Invocation record is created

---

### Test 15.7: Track Multiple Invocations
**Objective**: Verify that multiple invocations are tracked

**Setup**:
- Execute multiple tool invocations
- Track each

**Steps**:
1. Call track_tool_invocation() 5 times
2. Verify all invocations are tracked

**Expected Outcome**:
- All invocations are recorded

**Verification**:
- 5 invocation records are created

---

### Test 15.8: Invocation Tracking Updates Budget
**Objective**: Verify that invocation tracking updates budget

**Setup**:
- Track invocation
- Check budget

**Steps**:
1. Get initial budget
2. Call track_tool_invocation()
3. Get updated budget

**Expected Outcome**:
- Budget is updated

**Verification**:
- Updated budget < initial budget

---

### Test 15.9: Invocation Tracking Records Parameters
**Objective**: Verify that invocation parameters are recorded

**Setup**:
- Track invocation with parameters

**Steps**:
1. Call track_tool_invocation(tool="api", parameters={key: "value"})
2. Verify parameters are recorded

**Expected Outcome**:
- Parameters are recorded

**Verification**:
- Invocation record includes parameters

---

### Test 15.10: Invocation Tracking Records Result
**Objective**: Verify that invocation result is recorded

**Setup**:
- Track invocation with result

**Steps**:
1. Call track_tool_invocation(tool="api", result={data: "result"})
2. Verify result is recorded

**Expected Outcome**:
- Result is recorded

**Verification**:
- Invocation record includes result

---

## Test Suite: Outcome Recording

### Test 15.11: Commit Successful Outcome
**Objective**: Verify that successful outcomes are committed

**Setup**:
- Execute plan successfully
- Commit outcome

**Steps**:
1. Call commit(outcome="Plan succeeded", success=true)
2. Verify outcome is recorded

**Expected Outcome**:
- Outcome is recorded

**Verification**:
- Outcome record is created

---

### Test 15.12: Commit Failed Outcome
**Objective**: Verify that failed outcomes are committed

**Setup**:
- Execute plan that fails
- Commit outcome

**Steps**:
1. Call commit(outcome="Plan failed", success=false)
2. Verify outcome is recorded

**Expected Outcome**:
- Outcome is recorded

**Verification**:
- Outcome record is created with success=false

---

### Test 15.13: Commit Outcome with Feedback
**Objective**: Verify that feedback is recorded with outcome

**Setup**:
- Commit outcome with feedback

**Steps**:
1. Call commit(outcome, success=true, feedback="Good result")
2. Verify feedback is recorded

**Expected Outcome**:
- Feedback is recorded

**Verification**:
- Outcome record includes feedback

---

### Test 15.14: Commit Updates Episodic Memory
**Objective**: Verify that commit updates episodic memory

**Setup**:
- Commit outcome
- Check episodic memory

**Steps**:
1. Call commit(outcome)
2. Search episodic memory for outcome
3. Verify outcome is found

**Expected Outcome**:
- Outcome is in episodic memory

**Verification**:
- Outcome is searchable in episodic memory

---

### Test 15.15: Commit Updates Skills
**Objective**: Verify that commit updates skills

**Setup**:
- Commit outcome with skill update
- Check skill

**Steps**:
1. Call commit(outcome, success=true)
2. Check skill confidence
3. Verify confidence increased

**Expected Outcome**:
- Skill confidence increased

**Verification**:
- Skill confidence increases by >= 0.05 (on 0-1 scale)
- New confidence = old_confidence + (success_weight * learning_rate) where success_weight >= 0.05
- Confidence value remains in valid range [0.0, 1.0]

---

## Test Suite: Skill Retrieval

### Test 15.16: Get Applicable Skills
**Objective**: Verify that applicable skills are retrieved

**Setup**:
- Create skills
- Request applicable skills

**Steps**:
1. Call get_applicable_skills(context="weather_query")
2. Verify skills are returned

**Expected Outcome**:
- Applicable skills are returned

**Verification**:
- Returned skills are relevant to context

---

### Test 15.17: Skills Are Ranked by Relevance
**Objective**: Verify that skills are ranked by relevance

**Setup**:
- Create multiple applicable skills
- Request skills

**Steps**:
1. Call get_applicable_skills(context)
2. Verify skills are ranked

**Expected Outcome**:
- Skills are ranked by relevance

**Verification**:
- Most relevant skill is first

---

### Test 15.18: Skills Include Confidence Scores
**Objective**: Verify that skills include confidence scores

**Setup**:
- Get applicable skills

**Steps**:
1. Call get_applicable_skills(context)
2. Verify confidence scores

**Expected Outcome**:
- Skills include confidence scores

**Verification**:
- Each skill has confidence score in [0, 1]

---

## Test Suite: Integration Flow

### Test 15.19: Full Planning Flow
**Objective**: Verify full planning flow from context to outcome

**Setup**:
- Execute full planning flow

**Steps**:
1. Call assemble_context(task)
2. Call track_tool_invocation()
3. Call commit(outcome)
4. Verify all steps complete

**Expected Outcome**:
- All steps complete successfully

**Verification**:
- Context is assembled
- Invocation is tracked
- Outcome is committed

---

### Test 15.20: Multiple Planning Cycles
**Objective**: Verify multiple planning cycles work correctly

**Setup**:
- Execute multiple planning cycles

**Steps**:
1. Execute planning cycle 1
2. Execute planning cycle 2
3. Execute planning cycle 3
4. Verify all cycles complete

**Expected Outcome**:
- All cycles complete successfully

**Verification**:
- Each cycle produces outcome

---

## Test Suite: Edge Cases

### Test 15.21: Empty Context
**Objective**: Verify behavior with empty context

**Setup**:
- Request context with no relevant items

**Steps**:
1. Call assemble_context(task) with empty memory
2. Verify behavior

**Expected Outcome**:
- Empty context is returned or error is handled

**Verification**:
- Result is empty context or error

---

### Test 15.22: Tool Invocation with Error
**Objective**: Verify handling of tool invocation errors

**Setup**:
- Track tool invocation that errors

**Steps**:
1. Call track_tool_invocation(tool, result=error)
2. Verify error is recorded

**Expected Outcome**:
- Error is recorded

**Verification**:
- Invocation record includes error

---

### Test 15.23: Commit with No Feedback
**Objective**: Verify commit works without feedback

**Setup**:
- Commit outcome without feedback

**Steps**:
1. Call commit(outcome, success=true)
2. Verify commit succeeds

**Expected Outcome**:
- Commit succeeds

**Verification**:
- Outcome is recorded

