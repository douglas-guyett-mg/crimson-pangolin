# Evaluator Hobgoblin

## Purpose

The Evaluator assesses whether a turn has been completed successfully. It checks if the goal was achieved, identifies any issues, and produces an outcome assessment that informs the Reflector's learning.

## Responsibilities

1. **Define Success**: Determine what success looks like
2. **Assess Outcome**: Check if goal was achieved
3. **Identify Issues**: Find any problems or gaps
4. **Produce Assessment**: Create detailed outcome assessment
5. **Provide to Reflector**: Hand off assessment for learning

## When Evaluator is Activated

Evaluator is activated:
- After turn execution completes
- At key checkpoints during execution
- When errors occur
- When user provides feedback

## What Evaluator Assesses

### Goal Achievement
- Was the primary goal achieved?
- Were secondary goals achieved?
- Were there unintended consequences?

### Quality
- Is the result high quality?
- Does it meet user expectations?
- Are there gaps or limitations?

### Efficiency
- Was the turn executed efficiently?
- Were resources used well?
- Could it have been faster?

### Compliance
- Were constraints respected?
- Were governance rules followed?
- Were values upheld?

### User Satisfaction
- Did the user get what they needed?
- Is the user satisfied?
- Would the user recommend this?

## Decision Inputs

Evaluator makes decisions using:

### Consciousness
- **Success Criteria**: What defines success?
- **Values**: What matters in evaluation?
- **Governance**: What rules apply?

### Working Memory
- **Goal**: What was the original goal?
- **Plan**: What was supposed to happen?
- **Results**: What actually happened?
- **User Feedback**: What did the user say?

### Episodic Memory
- **Similar Situations**: How were similar turns evaluated?
- **Learned Patterns**: What evaluation patterns work?
- **Outcomes**: What made past turns successful?
- **Skills**: What evaluation skills has Si developed?

### Input
- **Turn Results**: What happened?
- **User Feedback**: What did the user say?
- **Context**: What was the situation?

## Evaluation Process

```
Turn execution completes
    ↓
Evaluator receives results
    ↓
Define success criteria:
  - What was the goal?
  - What would success look like?
  - What are the key metrics?
    ↓
Assess goal achievement:
  - Was primary goal achieved?
  - Were secondary goals achieved?
  - Were there unintended consequences?
    ↓
Assess quality:
  - Is result high quality?
  - Does it meet expectations?
  - Are there gaps?
    ↓
Assess efficiency:
  - Was execution efficient?
  - Were resources used well?
  - Could it have been faster?
    ↓
Assess compliance:
  - Were constraints respected?
  - Were rules followed?
  - Were values upheld?
    ↓
Assess user satisfaction:
  - Did user get what they needed?
  - Is user satisfied?
  - What feedback did user provide?
    ↓
Produce outcome assessment:
  - What was the goal?
  - What was expected?
  - What actually happened?
  - Did it match? Why or why not?
  - What worked well?
  - What could be improved?
    ↓
Provide to Reflector
```

## Outcome Assessment

An outcome assessment includes:

### Goal
- What was the original goal?
- What was the user trying to accomplish?

### Expected Outcome
- What was supposed to happen?
- What would success look like?

### Actual Outcome
- What actually happened?
- What were the results?

### Match Assessment
- Did actual match expected?
- If not, why not?
- What was different?

### Quality Assessment
- Was the result high quality?
- Were there gaps or issues?
- What worked well?

### Efficiency Assessment
- Was execution efficient?
- Could it have been faster?
- Were resources used well?

### Compliance Assessment
- Were constraints respected?
- Were rules followed?
- Were values upheld?

### User Satisfaction
- Did user get what they needed?
- Is user satisfied?
- What feedback did user provide?

### Recommendations
- What could be improved?
- What should be remembered?
- What patterns emerged?

## Evaluation Example

**Scenario**: Travel booking turn

**Goal**: Book a flight and hotel for Paris trip

**Expected Outcome**:
- Find 3+ flight options
- Find 3+ hotel options
- Present comparison
- User can make decision

**Actual Outcome**:
- Found 5 flight options
- Found 4 hotel options
- Presented comparison matrix
- User selected flight and hotel

**Match Assessment**: YES ✓
- All expectations met
- User got what they needed

**Quality Assessment**: HIGH
- Options were relevant
- Comparison was clear
- User was satisfied

**Efficiency Assessment**: GOOD
- Execution took 2 minutes
- Could have been faster with more parallelism
- Resources used efficiently

**Compliance Assessment**: COMPLIANT
- No constraints violated
- All rules followed
- Values upheld

**User Satisfaction**: HIGH
- User booked successfully
- User thanked Si
- User indicated satisfaction

**Recommendations**:
- Increase parallelism for faster execution
- This plan structure works well for travel
- User appreciated the comparison format

## Learning Through Evaluator

Evaluator improves through the learning loop:

**Feedback Sources**:
- Did evaluation criteria match actual success?
- Were important factors missed?
- Did evaluation help Reflector learn?
- Did evaluation predictions match outcomes?

**Learning Updates**:
- Episodic Memory: Store successful evaluation patterns
- Skills: Improve evaluation for similar situations
- Consciousness: Refine success criteria if needed

## Key Principle

**Evaluator enables learning.** By assessing outcomes thoroughly, Evaluator provides the information that Reflector needs to learn and improve. Without good evaluation, learning cannot happen.

