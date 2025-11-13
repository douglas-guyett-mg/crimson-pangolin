# Skill Extraction

## Overview

Skill Extraction is the system that identifies and extracts reusable skills from memory items and macro-summaries. It recognizes patterns of behavior, generalizes them into reusable procedures, and stores them for future use by the agent.

## Purpose

Skill Extraction serves to:

1. **Identify Patterns**: Recognize repeated sequences of actions
2. **Generalize Procedures**: Extract generalizable procedures from specific instances
3. **Create Reusable Skills**: Package procedures as reusable skills
4. **Enable Skill Reuse**: Make skills available for future tasks
5. **Improve Efficiency**: Reduce need to re-learn similar tasks

## Skill Structure

### Skill Definition
A skill consists of:

- **name**: Human-readable skill name (e.g., "weather_query_handling")
- **description**: What the skill does
- **preconditions**: When the skill applies
- **steps**: Sequence of steps to execute
- **postconditions**: Expected outcome
- **parameters**: Inputs to the skill
- **returns**: Outputs from the skill
- **examples**: Example uses of the skill
- **confidence**: How confident we are in the skill (0-1)
- **frequency**: How often the skill is used
- **last_used**: When the skill was last used

### Skill Types

**Procedural Skills**: Step-by-step procedures
- Example: "Retrieve weather forecast, format response, highlight key days"

**Decision Skills**: Decision-making procedures
- Example: "Determine if user query is about weather, sports, or news"

**Interaction Skills**: How to interact with tools/APIs
- Example: "Call weather API with location, parse response, format for user"

**Communication Skills**: How to communicate with user
- Example: "Provide concise answer, ask clarifying questions if needed"

## Skill Extraction Process

### Stage 1: Pattern Recognition
Identify repeated patterns in memory:
1. Scan macro-summaries for patterns
2. Identify repeated sequences of actions
3. Cluster similar patterns
4. Assess pattern frequency

### Stage 2: Generalization
Generalize patterns into reusable procedures:
1. Extract common steps
2. Identify variable parts (parameters)
3. Identify constant parts (core procedure)
4. Create generalized procedure

### Stage 3: Skill Creation
Create skill from generalized procedure:
1. Name the skill
2. Write description
3. Define preconditions
4. Define steps
5. Define postconditions
6. Define parameters and returns
7. Add examples

### Stage 4: Validation and Confidence Scoring
Validate skill quality and calculate initial confidence:
1. Check if skill is generalizable
2. Check if skill is reusable
3. Check if skill is well-defined
4. Calculate initial confidence score (Phase 1)
5. Decide whether to create skill (must be > 0.7)

## Confidence Scoring Model

Skill confidence is calculated in three phases:

### Phase 1: Extraction (Initial Confidence)

When Skill Extraction Daemon creates a skill, calculate initial confidence:

```
initial_confidence = (pattern_frequency_score × 0.4) +
                     (validation_quality_score × 0.6)

Where:
  pattern_frequency_score = (pattern_count - min_frequency) / (max_frequency - min_frequency)
    - Example: Pattern appeared 15 times out of 100 items
    - min_frequency = 3, max_frequency = 50
    - score = (15 - 3) / (50 - 3) = 12/47 = 0.26

  validation_quality_score = average of validation checks
    - Generalizability check: 0.0-1.0
    - Reusability check: 0.0-1.0
    - Well-defined check: 0.0-1.0
    - Parameter identification: 0.0-1.0
    - Example: (1.0 + 1.0 + 0.8 + 1.0) / 4 = 0.95

Example calculation:
  initial_confidence = (0.26 × 0.4) + (0.95 × 0.6)
                     = 0.104 + 0.57
                     = 0.674 ≈ 0.67

Decision:
  If initial_confidence > 0.7: Create skill
  If initial_confidence ≤ 0.7: Discard skill
```

**Rationale:**
- Pattern frequency validates that this is a real, repeatable behavior
- Validation quality ensures the skill is well-defined and generalizable
- Can be calculated immediately at extraction time (no need to wait for usage)

### Phase 2: Usage (Update with Success Rate)

When skill is applied and completes, update confidence:

```
updated_confidence = (initial_confidence × 0.5) +
                     (success_rate × 0.5)

Where:
  success_rate = successful_uses / total_uses
    - Example: Skill used 5 times, succeeded 4 times
    - success_rate = 4/5 = 0.8

Example calculation:
  updated_confidence = (0.90 × 0.5) + (0.8 × 0.5)
                     = 0.45 + 0.40
                     = 0.85

Decision:
  If updated_confidence > 0.7: Keep skill active
  If updated_confidence ≤ 0.7: Mark as low-confidence (may be deprecated)
```

**Rationale:**
- Balances initial confidence with actual performance
- Success rate measures whether skill works in practice
- Gradually shifts from theoretical quality to empirical performance

### Phase 3: Evolution (Decay and Continuous Updates)

Over time, update confidence based on usage and decay:

```
Decay (if skill not used for 30 days):
  confidence = confidence × 0.95  (5% decay per 30-day period)

Success Update (when skill is used successfully):
  confidence = min(1.0, confidence + 0.05)

Failure Update (when skill fails):
  confidence = max(0.0, confidence - 0.10)

Example timeline:
  Day 0: confidence = 0.85, skill used successfully
    → confidence = min(1.0, 0.85 + 0.05) = 0.90

  Day 5: skill used, fails
    → confidence = max(0.0, 0.90 - 0.10) = 0.80

  Day 35: skill not used for 30 days
    → confidence = 0.80 × 0.95 = 0.76

  Day 40: skill used successfully
    → confidence = min(1.0, 0.76 + 0.05) = 0.81
```

**Rationale:**
- Decay handles skills that become obsolete or less relevant
- Success updates reward reliable skills
- Failure updates penalize unreliable skills
- Asymmetric penalties (−10% for failure, +5% for success) are conservative

---

## Operations

### extract_skills(items)
Extracts skills from memory items.

**Parameters**:
- `items`: List of MemoryItem objects or macro-summaries

**Returns**:
- List of extracted skills
- Extraction metadata
- Confidence scores

### create_skill(name, description, steps, parameters, returns)
Creates a new skill.

**Parameters**:
- `name`: Skill name
- `description`: Skill description
- `steps`: List of steps
- `parameters`: List of parameters
- `returns`: List of return values

**Returns**:
- Created skill
- Skill ID

### validate_skill(skill)
Validates a skill.

**Parameters**:
- `skill`: Skill to validate

**Returns**:
- Validation result
- Quality score
- Issues (if any)

### apply_skill(skill, parameters)
Applies a skill with given parameters.

**Parameters**:
- `skill`: Skill to apply
- `parameters`: Parameter values

**Returns**:
- Execution result
- Outcome

### get_applicable_skills(context)
Gets skills applicable to current context.

**Parameters**:
- `context`: Current context

**Returns**:
- List of applicable skills
- Relevance scores

## Design Principles

1. **Generalizable**: Skills are generalizable to new situations
2. **Reusable**: Skills can be applied to multiple tasks
3. **Well-Defined**: Skills have clear steps and parameters
4. **Validated**: Skills are validated before use
5. **Tracked**: Skill usage is tracked for improvement

## Interaction with Other Components

- **Multi-Resolution Summarizer**: Provides macro-summaries for extraction
- **Memory Orchestrator**: Coordinates skill extraction
- **Semantic Memory**: Stores extracted skills
- **Frontal Cortex**: Uses skills for planning
- **Turn Trace**: Logs skill extraction and usage

## Configuration

Skill Extraction can be configured with:

- **min_pattern_frequency**: Minimum frequency to extract skill (default: 3)
- **min_confidence**: Minimum confidence to accept skill (default: 0.7)
- **enable_auto_extraction**: Whether to automatically extract skills (default: true)
- **extraction_interval**: How often to extract skills (default: every 100 items)
- **max_skills**: Maximum number of skills to store (default: 1000)

## Example Scenario

1. Agent has processed 100 items over multiple conversations
2. Multi-Resolution Summarizer generates macro-summaries
3. **Pattern Recognition**:
   - Scan macro-summaries
   - Identify pattern: "User asks question → System retrieves data → System formats response → User thanks"
   - Pattern frequency: 15 times
4. **Generalization**:
   - Extract common steps: "Retrieve data, format response"
   - Identify variable parts: question type, data source
   - Create generalized procedure
5. **Skill Creation**:
   - Name: "query_handling"
   - Description: "Handle user queries by retrieving and formatting data"
   - Steps: ["Identify question type", "Retrieve relevant data", "Format response", "Provide to user"]
   - Parameters: ["question_type", "data_source"]
   - Returns: ["formatted_response"]
6. **Validation**:
   - Check generalizability: Yes, applies to multiple question types
   - Check reusability: Yes, can be applied to new questions
   - Confidence: 0.85
7. **Storage**:
   - Store skill in Semantic Memory
   - Make available for future use
   - Track usage statistics

