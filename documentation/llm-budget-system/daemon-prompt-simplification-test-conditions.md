# Daemon Prompt Simplification - Test Conditions

**Status**: Specification v1.0  
**Last Updated**: 2025-11-04  
**Scope**: Tests for daemon-specific simplification strategies and quality preservation

---

## Test Suite: Simplification Levels

### Test 1.1: Light Simplification (Level 1)
**Objective**: Verify light simplification removes verbose explanations but keeps facts

**Setup**:
- Prepare prompt with 5000 tokens
- Target size: 4000 tokens
- Simplification level: 1 (light)

**Steps**:
1. Call `daemon.simplify_prompt(prompt, 5000, 4000, level=1)`
2. Verify result

**Expected Outcome**:
- Simplified prompt: ~4000 tokens
- All facts preserved
- Verbose explanations removed

**Verification**:
- Result token count <= 4000
- All key facts present in simplified prompt

---

### Test 1.2: Medium Simplification (Level 2)
**Objective**: Verify medium simplification summarizes context but keeps critical facts

**Setup**:
- Prepare prompt with 6000 tokens
- Target size: 3000 tokens
- Simplification level: 2 (medium)

**Steps**:
1. Call `daemon.simplify_prompt(prompt, 6000, 3000, level=2)`
2. Verify result

**Expected Outcome**:
- Simplified prompt: ~3000 tokens
- Context summarized
- Critical facts preserved

**Verification**:
- Result token count <= 3000
- Critical facts still present

---

### Test 1.3: Heavy Simplification (Level 3)
**Objective**: Verify heavy simplification keeps only essential facts

**Setup**:
- Prepare prompt with 8000 tokens
- Target size: 2000 tokens
- Simplification level: 3 (heavy)

**Steps**:
1. Call `daemon.simplify_prompt(prompt, 8000, 2000, level=3)`
2. Verify result

**Expected Outcome**:
- Simplified prompt: ~2000 tokens
- Only essential facts kept
- Minimal context

**Verification**:
- Result token count <= 2000
- Essential facts preserved

---

## Test Suite: Router Daemon Simplification

### Test 2.1: Router Preserves Urgency Signals
**Objective**: Verify Router preserves urgency keywords during simplification

**Setup**:
- Prepare prompt with urgency keywords: "urgent", "critical", "asap"
- Prompt size: 5000 tokens
- Target: 3000 tokens

**Steps**:
1. Call `router.simplify_prompt(prompt, 5000, 3000, level=2)`
2. Verify urgency keywords preserved

**Expected Outcome**:
- Simplified prompt contains urgency keywords
- Context history removed
- Recent turns preserved

**Verification**:
- "urgent" or "critical" or "asap" present in result
- Result token count <= 3000

---

### Test 2.2: Router Removes Verbose Context History
**Objective**: Verify Router removes verbose context but keeps recent turns

**Setup**:
- Prompt with 20 turns of conversation history
- Prompt size: 6000 tokens
- Target: 3000 tokens

**Steps**:
1. Call `router.simplify_prompt(prompt, 6000, 3000, level=2)`
2. Verify context history reduced

**Expected Outcome**:
- Only recent 3-5 turns kept
- Old turns removed
- Recent context preserved

**Verification**:
- Result contains only recent turns
- Result token count <= 3000

---

## Test Suite: Frontal Cortex Daemon Simplification

### Test 3.1: Frontal Cortex Preserves Goal and Success Criteria
**Objective**: Verify Frontal Cortex preserves goal and success criteria

**Setup**:
- Prompt with goal, constraints, and detailed context
- Prompt size: 7000 tokens
- Target: 4000 tokens

**Steps**:
1. Call `plan_generator.simplify_prompt(prompt, 7000, 4000, level=2)`
2. Verify goal and criteria preserved

**Expected Outcome**:
- Goal statement preserved
- Success criteria preserved
- Detailed context summarized

**Verification**:
- Goal present in result
- Success criteria present in result
- Result token count <= 4000

---

### Test 3.2: Frontal Cortex Summarizes Tool Descriptions
**Objective**: Verify Frontal Cortex summarizes tool descriptions to essential capabilities

**Setup**:
- Prompt with detailed tool descriptions (verbose)
- Prompt size: 6000 tokens
- Target: 3000 tokens

**Steps**:
1. Call `plan_generator.simplify_prompt(prompt, 6000, 3000, level=2)`
2. Verify tool descriptions summarized

**Expected Outcome**:
- Tool descriptions condensed to essential capabilities
- Tool names and key functions preserved
- Verbose documentation removed

**Verification**:
- Tool names present in result
- Tool capabilities summarized
- Result token count <= 3000

---

## Test Suite: Executor Daemon Simplification

### Test 4.1: Executor Preserves Current Step
**Objective**: Verify Executor preserves current step and next steps

**Setup**:
- Prompt with full plan (10 steps) and current step (step 5)
- Prompt size: 5000 tokens
- Target: 3000 tokens

**Steps**:
1. Call `executor.simplify_prompt(prompt, 5000, 3000, level=2)`
2. Verify current and next steps preserved

**Expected Outcome**:
- Current step (step 5) preserved
- Next 2 steps preserved
- Previous steps removed

**Verification**:
- Current step present in result
- Next 2 steps present in result
- Result token count <= 3000

---

### Test 4.2: Executor Preserves Error Context
**Objective**: Verify Executor preserves error messages and constraints

**Setup**:
- Prompt with error message and constraints
- Prompt size: 4000 tokens
- Target: 2500 tokens

**Steps**:
1. Call `executor.simplify_prompt(prompt, 4000, 2500, level=2)`
2. Verify error and constraints preserved

**Expected Outcome**:
- Error message preserved
- Constraints preserved
- Execution trace removed

**Verification**:
- Error message present in result
- Constraints present in result
- Result token count <= 2500

---

## Test Suite: Evaluator Daemon Simplification

### Test 5.1: Evaluator Preserves Expected vs Actual Outcome
**Objective**: Verify Evaluator preserves expected and actual outcomes

**Setup**:
- Prompt with expected outcome, actual outcome, and detailed trace
- Prompt size: 5000 tokens
- Target: 3000 tokens

**Steps**:
1. Call `evaluator.simplify_prompt(prompt, 5000, 3000, level=2)`
2. Verify outcomes preserved

**Expected Outcome**:
- Expected outcome preserved
- Actual outcome preserved
- Detailed trace removed

**Verification**:
- Expected outcome present in result
- Actual outcome present in result
- Result token count <= 3000

---

### Test 5.2: Evaluator Preserves Success Criteria
**Objective**: Verify Evaluator preserves success criteria and user feedback

**Setup**:
- Prompt with success criteria, user feedback, and execution details
- Prompt size: 4500 tokens
- Target: 2500 tokens

**Steps**:
1. Call `evaluator.simplify_prompt(prompt, 4500, 2500, level=2)`
2. Verify criteria and feedback preserved

**Expected Outcome**:
- Success criteria preserved
- User feedback preserved
- Execution details removed

**Verification**:
- Success criteria present in result
- User feedback present in result
- Result token count <= 2500

---

## Test Suite: Determinism and Auditability

### Test 6.1: Same Prompt and Level Produces Same Simplification
**Objective**: Verify simplification is deterministic

**Setup**:
- Prepare prompt
- Simplify twice with same parameters

**Steps**:
1. Call `daemon.simplify_prompt(prompt, 5000, 3000, level=2)` → result1
2. Call `daemon.simplify_prompt(prompt, 5000, 3000, level=2)` → result2
3. Compare results

**Expected Outcome**:
- result1 == result2
- Simplification is deterministic

**Verification**:
- result1 token count == result2 token count
- result1 content == result2 content

---

### Test 6.2: Log All Simplifications
**Objective**: Verify all simplifications are logged for learning

**Setup**:
- Configure daemon with log_simplifications=true
- Make multiple simplifications

**Steps**:
1. Simplify 3 prompts
2. Query simplification log

**Expected Outcome**:
- All 3 simplifications logged
- Log includes: daemon, original_tokens, target_tokens, level, result_tokens

**Verification**:
- `get_simplification_log(daemon)` returns 3 entries
- Each entry has required fields

---

## Test Suite: Edge Cases

### Test 7.1: Prompt Already Within Budget
**Objective**: Verify no simplification needed if prompt within budget

**Setup**:
- Prepare prompt with 2000 tokens
- Target: 3000 tokens

**Steps**:
1. Call `daemon.simplify_prompt(prompt, 2000, 3000, level=1)`
2. Verify result

**Expected Outcome**:
- Prompt returned unchanged
- No simplification applied

**Verification**:
- Result == original prompt
- Result token count == 2000

---

### Test 7.2: Impossible Target (Too Small)
**Objective**: Verify behavior when target is too small

**Setup**:
- Prepare prompt with 5000 tokens
- Target: 100 tokens (impossible)

**Steps**:
1. Call `daemon.simplify_prompt(prompt, 5000, 100, level=3)`
2. Verify result

**Expected Outcome**:
- Best effort simplification
- Result as close to 100 as possible
- Essential information preserved

**Verification**:
- Result token count <= 100 or close to it
- Essential information preserved

