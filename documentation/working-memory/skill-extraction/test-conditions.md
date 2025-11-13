# Skill Extraction - Test Conditions

## Test Suite: Confidence Scoring

### Test 0.1: Phase 1 - Initial Confidence Calculation
**Objective**: Verify initial confidence is calculated correctly at extraction time

**Setup**:
- Pattern appeared 15 times in 100 items (min_frequency=3, max_frequency=50)
- Validation checks: all 4 passed (generalizability, reusability, well-defined, parameters)

**Steps**:
1. Calculate pattern_frequency_score = (15 - 3) / (50 - 3) = 0.255
2. Calculate validation_quality_score = (1.0 + 1.0 + 1.0 + 1.0) / 4 = 1.0
3. Calculate initial_confidence = (0.255 × 0.4) + (1.0 × 0.6) = 0.702

**Expected Outcome**:
- initial_confidence = 0.702
- Skill is created (confidence > 0.7)

**Verification**:
- Skill exists in Semantic Memory
- Skill.confidence = 0.702

---

### Test 0.2: Phase 1 - Confidence Below Threshold
**Objective**: Verify skills with confidence ≤ 0.7 are not created

**Setup**:
- Pattern appeared 5 times in 100 items
- Validation checks: 3/4 passed (one check failed)

**Steps**:
1. Calculate pattern_frequency_score = (5 - 3) / (50 - 3) = 0.043
2. Calculate validation_quality_score = (1.0 + 1.0 + 0.5 + 1.0) / 4 = 0.875
3. Calculate initial_confidence = (0.043 × 0.4) + (0.875 × 0.6) = 0.542

**Expected Outcome**:
- initial_confidence = 0.542
- Skill is NOT created (confidence ≤ 0.7)

**Verification**:
- Skill does not exist in Semantic Memory
- Extraction logs indicate skill was rejected

---

### Test 0.3: Phase 2 - Update Confidence with Success Rate
**Objective**: Verify confidence is updated when skill is used successfully

**Setup**:
- Skill created with initial_confidence = 0.90
- Skill applied 5 times, succeeded 4 times

**Steps**:
1. Calculate success_rate = 4/5 = 0.8
2. Calculate updated_confidence = (0.90 × 0.5) + (0.8 × 0.5) = 0.85

**Expected Outcome**:
- updated_confidence = 0.85
- Skill remains active (confidence > 0.7)

**Verification**:
- Skill.confidence updated to 0.85
- Skill.frequency = 5
- Skill.last_used = current timestamp

---

### Test 0.4: Phase 2 - Confidence Drops Below Threshold
**Objective**: Verify skill is marked low-confidence if success rate is poor

**Setup**:
- Skill created with initial_confidence = 0.75
- Skill applied 10 times, succeeded 2 times

**Steps**:
1. Calculate success_rate = 2/10 = 0.2
2. Calculate updated_confidence = (0.75 × 0.5) + (0.2 × 0.5) = 0.475

**Expected Outcome**:
- updated_confidence = 0.475
- Skill marked as low-confidence (confidence ≤ 0.7)

**Verification**:
- Skill.confidence updated to 0.475
- Skill.status = "low_confidence"
- Skill is not recommended for new tasks

---

### Test 0.5: Phase 3 - Decay Over Time
**Objective**: Verify confidence decays when skill is not used

**Setup**:
- Skill has confidence = 0.80
- Skill not used for 30 days

**Steps**:
1. Calculate decayed_confidence = 0.80 × 0.95 = 0.76

**Expected Outcome**:
- decayed_confidence = 0.76
- Skill remains active but less preferred

**Verification**:
- Skill.confidence updated to 0.76
- Skill.last_used timestamp is 30+ days old

---

### Test 0.6: Phase 3 - Success Update
**Objective**: Verify confidence increases when skill is used successfully

**Setup**:
- Skill has confidence = 0.80
- Skill used successfully

**Steps**:
1. Calculate updated_confidence = min(1.0, 0.80 + 0.05) = 0.85

**Expected Outcome**:
- updated_confidence = 0.85
- Skill becomes more trusted

**Verification**:
- Skill.confidence updated to 0.85
- Skill.frequency incremented
- Skill.last_used updated

---

### Test 0.7: Phase 3 - Failure Update
**Objective**: Verify confidence decreases when skill fails

**Setup**:
- Skill has confidence = 0.80
- Skill used but failed

**Steps**:
1. Calculate updated_confidence = max(0.0, 0.80 - 0.10) = 0.70

**Expected Outcome**:
- updated_confidence = 0.70
- Skill becomes less trusted

**Verification**:
- Skill.confidence updated to 0.70
- Skill.failure_count incremented
- Skill.last_used updated

---

### Test 0.8: Phase 3 - Confidence Capped at 1.0
**Objective**: Verify confidence cannot exceed 1.0

**Setup**:
- Skill has confidence = 0.98
- Skill used successfully 5 times

**Steps**:
1. Update 1: min(1.0, 0.98 + 0.05) = 1.0
2. Update 2: min(1.0, 1.0 + 0.05) = 1.0
3. Update 3: min(1.0, 1.0 + 0.05) = 1.0

**Expected Outcome**:
- Confidence capped at 1.0
- Does not exceed 1.0

**Verification**:
- Skill.confidence = 1.0
- Skill.confidence never exceeds 1.0

---

### Test 0.9: Phase 3 - Confidence Floored at 0.0
**Objective**: Verify confidence cannot go below 0.0

**Setup**:
- Skill has confidence = 0.05
- Skill fails

**Steps**:
1. Calculate updated_confidence = max(0.0, 0.05 - 0.10) = 0.0

**Expected Outcome**:
- Confidence floored at 0.0
- Does not go negative

**Verification**:
- Skill.confidence = 0.0
- Skill.confidence never goes below 0.0

---

## Test Suite: Pattern Recognition

### Test 14.1: Recognize Repeated Patterns
**Objective**: Verify that repeated patterns are recognized

**Setup**:
- Create items showing repeated pattern
- Pattern repeats 5 times

**Steps**:
1. Create items: A→B→C, A→B→C, A→B→C, A→B→C, A→B→C
2. Call extract_skills(items)
3. Verify pattern is recognized

**Expected Outcome**:
- Pattern A→B→C is recognized

**Verification**:
- Extracted skills include pattern A→B→C

---

### Test 14.2: Ignore Infrequent Patterns
**Objective**: Verify that infrequent patterns are ignored

**Setup**:
- Create items with infrequent pattern
- Pattern repeats 1 time
- Set min_pattern_frequency=3

**Steps**:
1. Create items with pattern repeating 1 time
2. Call extract_skills(items)
3. Verify pattern is not extracted

**Expected Outcome**:
- Pattern is not extracted

**Verification**:
- Extracted skills do not include infrequent pattern

---

### Test 14.3: Recognize Multiple Patterns
**Objective**: Verify that multiple patterns are recognized

**Setup**:
- Create items with multiple patterns
- Pattern 1 repeats 5 times
- Pattern 2 repeats 4 times

**Steps**:
1. Create items with multiple patterns
2. Call extract_skills(items)
3. Verify both patterns are recognized

**Expected Outcome**:
- Both patterns are recognized

**Verification**:
- Extracted skills include both patterns

---

## Test Suite: Generalization

### Test 14.4: Generalize Specific Steps
**Objective**: Verify that specific steps are generalized

**Setup**:
- Create items with specific steps
- Steps vary in details but follow same pattern

**Steps**:
1. Create items: "Retrieve weather data, format response", "Retrieve sports data, format response"
2. Call extract_skills(items)
3. Verify generalization

**Expected Outcome**:
- Generalized skill: "Retrieve data, format response"

**Verification**:
- Extracted skill has generalized steps

---

### Test 14.5: Identify Parameters
**Objective**: Verify that variable parts are identified as parameters

**Setup**:
- Create items with variable parts
- Variable parts: data type, format

**Steps**:
1. Create items with different data types and formats
2. Call extract_skills(items)
3. Verify parameters are identified

**Expected Outcome**:
- Parameters are identified: data_type, format

**Verification**:
- Extracted skill has parameters

---

### Test 14.6: Identify Constants
**Objective**: Verify that constant parts are identified

**Setup**:
- Create items with constant parts
- Constant parts: "Retrieve", "Format", "Provide"

**Steps**:
1. Create items
2. Call extract_skills(items)
3. Verify constants are identified

**Expected Outcome**:
- Constants are identified in skill steps

**Verification**:
- Extracted skill includes constant steps

---

## Test Suite: Skill Creation

### Test 14.7: Create Skill with Name
**Objective**: Verify that skill is created with name

**Setup**:
- Create skill with name="weather_query"

**Steps**:
1. Call create_skill(name="weather_query", ...)
2. Verify skill is created

**Expected Outcome**:
- Skill is created with name

**Verification**:
- `skill.name` == "weather_query"

---

### Test 14.8: Create Skill with Steps
**Objective**: Verify that skill is created with steps

**Setup**:
- Create skill with steps

**Steps**:
1. Call create_skill(..., steps=["Retrieve data", "Format response", "Provide to user"])
2. Verify steps are stored

**Expected Outcome**:
- Skill has steps

**Verification**:
- `skill.steps` == ["Retrieve data", "Format response", "Provide to user"]

---

### Test 14.9: Create Skill with Parameters
**Objective**: Verify that skill is created with parameters

**Setup**:
- Create skill with parameters

**Steps**:
1. Call create_skill(..., parameters=["data_type", "format"])
2. Verify parameters are stored

**Expected Outcome**:
- Skill has parameters

**Verification**:
- `skill.parameters` == ["data_type", "format"]

---

### Test 14.10: Create Skill with Returns
**Objective**: Verify that skill is created with returns

**Setup**:
- Create skill with returns

**Steps**:
1. Call create_skill(..., returns=["formatted_response"])
2. Verify returns are stored

**Expected Outcome**:
- Skill has returns

**Verification**:
- `skill.returns` == ["formatted_response"]

---

## Test Suite: Validation

### Test 14.11: Validate Generalizable Skill
**Objective**: Verify that generalizable skills pass validation

**Setup**:
- Create generalizable skill

**Steps**:
1. Create skill that applies to multiple situations
2. Call validate_skill(skill)
3. Verify validation passes

**Expected Outcome**:
- Validation passes
- Quality score is high

**Verification**:
- `validation.passed` == true
- `validation.quality_score` > 0.7

---

### Test 14.12: Reject Non-Generalizable Skill
**Objective**: Verify that non-generalizable skills fail validation

**Setup**:
- Create non-generalizable skill

**Steps**:
1. Create skill that only applies to one specific situation
2. Call validate_skill(skill)
3. Verify validation fails

**Expected Outcome**:
- Validation fails
- Quality score is low

**Verification**:
- `validation.passed` == false
- `validation.quality_score` < 0.7

---

### Test 14.13: Validate Reusable Skill
**Objective**: Verify that reusable skills pass validation

**Setup**:
- Create reusable skill

**Steps**:
1. Create skill that can be applied to multiple tasks
2. Call validate_skill(skill)
3. Verify validation passes

**Expected Outcome**:
- Validation passes

**Verification**:
- `validation.passed` == true

---

## Test Suite: Skill Application

### Test 14.14: Apply Skill with Parameters
**Objective**: Verify that skill can be applied with parameters

**Setup**:
- Create skill with parameters
- Provide parameter values

**Steps**:
1. Call apply_skill(skill, parameters={"data_type": "weather", "format": "json"})
2. Verify skill is applied

**Expected Outcome**:
- Skill is applied with parameters

**Verification**:
- Execution result is returned

---

### Test 14.15: Apply Skill Returns Expected Output
**Objective**: Verify that skill application returns expected output

**Setup**:
- Create skill
- Apply skill

**Steps**:
1. Call apply_skill(skill, parameters)
2. Verify output

**Expected Outcome**:
- Output matches skill returns

**Verification**:
- Output includes expected return values

---

## Test Suite: Skill Retrieval

### Test 14.16: Get Applicable Skills
**Objective**: Verify that applicable skills are retrieved

**Setup**:
- Create multiple skills
- Provide context

**Steps**:
1. Create skills for different tasks
2. Call get_applicable_skills(context="weather_query")
3. Verify applicable skills are returned

**Expected Outcome**:
- Only applicable skills are returned

**Verification**:
- Returned skills are relevant to context

---

### Test 14.17: Applicable Skills Are Ranked
**Objective**: Verify that applicable skills are ranked by relevance

**Setup**:
- Create multiple applicable skills
- Skills have different relevance

**Steps**:
1. Call get_applicable_skills(context)
2. Verify skills are ranked

**Expected Outcome**:
- Skills are ranked by relevance

**Verification**:
- Most relevant skill is first

---

## Test Suite: Skill Tracking

### Test 14.18: Track Skill Usage
**Objective**: Verify that skill usage is tracked

**Setup**:
- Create skill
- Apply skill multiple times

**Steps**:
1. Apply skill 5 times
2. Check skill usage statistics

**Expected Outcome**:
- Usage count is 5

**Verification**:
- `skill.frequency` == 5

---

### Test 14.19: Track Last Used Time
**Objective**: Verify that last used time is tracked

**Setup**:
- Create skill
- Apply skill

**Steps**:
1. Apply skill
2. Check last_used timestamp

**Expected Outcome**:
- Last used time is updated

**Verification**:
- `skill.last_used` is recent

---

## Test Suite: Edge Cases

### Test 14.20: Empty Items List
**Objective**: Verify behavior with empty items list

**Setup**:
- Create empty items list

**Steps**:
1. Call extract_skills([])
2. Verify behavior

**Expected Outcome**:
- Returns empty skills list or no error

**Verification**:
- Result is empty list or error is handled

---

### Test 14.21: Single Item
**Objective**: Verify behavior with single item

**Setup**:
- Create single item

**Steps**:
1. Call extract_skills([item])
2. Verify behavior

**Expected Outcome**:
- No skills extracted (pattern needs repetition)

**Verification**:
- Result is empty list

---

### Test 14.22: Very Complex Pattern
**Objective**: Verify handling of complex patterns

**Setup**:
- Create complex pattern with many steps

**Steps**:
1. Create pattern with 20+ steps
2. Call extract_skills(items)
3. Verify skill is extracted

**Expected Outcome**:
- Skill is extracted
- Skill is well-defined

**Verification**:
- Extracted skill has all steps
- Skill is valid

---

## Test Suite: Performance

### Test 14.23: Extraction Performance
**Objective**: Verify that extraction is performant

**Setup**:
- Create 1000 items
- Extract skills

**Steps**:
1. Create 1000 items
2. Call extract_skills(items)
3. Measure time

**Expected Outcome**:
- Extraction completes in reasonable time (< 10 seconds)

**Verification**:
- Extraction time is acceptable

