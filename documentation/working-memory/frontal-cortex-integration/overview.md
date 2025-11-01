# Frontal Cortex Integration

## Overview

Frontal Cortex Integration defines how the Working Memory system interfaces with the Frontal Cortex (planning and decision-making system). It specifies how memory is assembled for planning, how plans are executed, how outcomes are recorded, and how feedback loops improve future planning.

## Purpose

Frontal Cortex Integration serves to:

1. **Provide Context for Planning**: Assemble relevant memory for planning tasks
2. **Support Plan Execution**: Track plan execution and tool invocations
3. **Record Outcomes**: Store execution results and outcomes
4. **Enable Feedback**: Provide feedback to improve future planning
5. **Coordinate Systems**: Ensure memory and planning systems work together

## Integration Points

### Context Assembly for Planning
When Frontal Cortex needs to plan:
1. Frontal Cortex calls `assemble_context(prompt, budget, constraints)`
2. Memory Orchestrator calls remember() on all memory types with the prompt
3. Memory Orchestrator applies budget constraints
4. Memory Orchestrator returns assembled context
5. Frontal Cortex uses context for planning

**Context Types**:
- **Goal Context**: Current goals and objectives
- **State Context**: Current state of the world
- **History Context**: Relevant past experiences
- **Skill Context**: Available skills and procedures
- **Constraint Context**: Current constraints and limitations

### Plan Execution Tracking
When Frontal Cortex executes a plan:
1. Frontal Cortex calls `track_tool_invocation(tool, parameters, result)`
2. Memory Orchestrator records the invocation
3. Memory Orchestrator updates working memory
4. Memory Orchestrator tracks budget consumption
5. Frontal Cortex continues execution

**Tracked Information**:
- Tool name and parameters
- Execution time
- Result and outcome
- Errors or exceptions
- Budget consumed

### Outcome Recording
When plan execution completes:
1. Frontal Cortex calls `commit(outcome, success, feedback)`
2. Memory Orchestrator writes outcome to memory
3. Memory Orchestrator updates episodic memory
4. Memory Orchestrator extracts lessons learned
5. Memory Orchestrator updates skills if applicable

**Outcome Information**:
- Success or failure
- Actual outcome vs. expected
- Lessons learned
- Feedback for improvement
- Skill updates

### Feedback Loop
Continuous improvement through feedback:
1. Frontal Cortex provides feedback on plan quality
2. Memory Orchestrator records feedback
3. Memory Orchestrator updates skill confidence
4. Memory Orchestrator adjusts future planning
5. System improves over time

## Operations

### assemble_context(prompt, budget, constraints)
Assembles context for planning based on a prompt.

**Parameters**:
- `prompt`: Planning task description/prompt
- `budget`: Token budget for context
- `constraints`: Additional constraints

**Returns**:
- Assembled context
- Context metadata
- Budget status

### track_tool_invocation(tool, parameters, result)
Tracks tool invocation during plan execution.

**Parameters**:
- `tool`: Tool name
- `parameters`: Tool parameters
- `result`: Tool result

**Returns**:
- Invocation record
- Budget update

### commit(outcome, success, feedback)
Commits plan outcome to memory.

**Parameters**:
- `outcome`: Plan outcome
- `success`: Whether plan succeeded
- `feedback`: Feedback for improvement

**Returns**:
- Commit confirmation
- Updated memory state

### get_applicable_skills(context)
Gets skills applicable to current context.

**Parameters**:
- `context`: Current context

**Returns**:
- List of applicable skills
- Relevance scores

## Design Principles

1. **Tight Integration**: Memory and planning are tightly integrated
2. **Bidirectional**: Information flows both ways
3. **Feedback-Driven**: System improves through feedback
4. **Efficient**: Context assembly is efficient
5. **Auditable**: All interactions are logged

## Interaction with Other Components

- **Memory Orchestrator**: Central coordinator
- **Frontal Cortex**: Planning and decision-making system
- **Skill Extraction**: Extracts skills from outcomes
- **Turn Trace**: Logs all interactions
- **Budget Manager**: Tracks budget consumption

## Configuration

Frontal Cortex Integration can be configured with:

- **context_token_limit**: Maximum tokens for context (default: 2000)
- **enable_skill_extraction**: Whether to extract skills from outcomes (default: true)
- **enable_feedback_loop**: Whether to use feedback for improvement (default: true)
- **feedback_weight**: How much to weight feedback (default: 0.5)

## Example Scenario

1. User asks: "What's the weather in Paris?"
2. **Context Assembly**:
   - Frontal Cortex calls `assemble_context(prompt="answer_weather_query", budget=2000, constraints={})`
   - Memory Orchestrator calls remember("answer_weather_query") on all memory types:
     - Working Memory returns recent weather queries
     - Semantic Memory returns available weather tools
     - Episodic Memory returns relevant past weather queries
     - Conversation History returns recent weather discussions
   - Returns combined context with weather query skill
3. **Plan Execution**:
   - Frontal Cortex creates plan: "Use weather skill to get forecast"
   - Frontal Cortex calls `track_tool_invocation(tool="weather_api", parameters={location: "Paris"}, result={forecast: "sunny"})`
   - Memory Orchestrator memorizes invocation
4. **Outcome Recording**:
   - Frontal Cortex calls `commit(outcome="Provided weather forecast", success=true, feedback="Query was answered correctly")`
   - Memory Orchestrator memorizes outcome to episodic memory
   - Memory Orchestrator updates skill confidence
5. **Feedback Loop**:
   - System records successful outcome
   - Skill confidence increases
   - Future weather queries will use this skill more confidently

