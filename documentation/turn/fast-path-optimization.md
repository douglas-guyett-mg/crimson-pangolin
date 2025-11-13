# Fast Path Optimization

## Overview

The turn architecture is designed as modular, separate daemons for clarity and maintainability. However, for speed optimization, these daemons can be combined into a single LLM call in a "fast path" when the technology stack is chosen.

## Design Philosophy

**Modularity First, Optimization Second**

- **Design Phase**: Daemons are documented as separate components with clear responsibilities
- **Implementation Phase**: Technology choices determine whether to use modular or optimized approach
- **Optimization Phase**: Fast path combines daemons into single LLM call for speed

This approach ensures:
- Clear specification of what each daemon does
- Flexibility in implementation strategy
- Ability to optimize without losing clarity

## Fast Path Concept

### Modular Approach (Current Design)

```
Router → Clarifier → Executor → Evaluator → Reflector → Responder

Each daemon is a separate decision point
Each daemon makes its own decision
Multiple LLM calls may be needed
```

### Fast Path Approach (Optimization)

```
Single LLM Call:
  Input: User message + consciousness + working memory + episodic memory
  Output: All daemon decisions in one call
    - Urgency assessment (Router)
    - Clarification needed? (Clarifier)
    - Relevant context (Context Assembler)
    - Constraint violations? (Constraint Checker)
    - Execution plan (Plan Generator)
    - Tool calls (Executor)
    - Response (Responder)
```

## When Fast Path Makes Sense

### Advantages
- **Speed**: Single LLM call is faster than multiple calls
- **Latency**: Reduced round-trip time
- **Cost**: Fewer API calls
- **Simplicity**: Single decision point

### Disadvantages
- **Complexity**: Single prompt must handle all decisions
- **Flexibility**: Harder to customize individual daemons
- **Debugging**: Harder to understand which daemon made which decision
- **Modularity**: Loses separation of concerns

## When Modular Approach Makes Sense

### Advantages
- **Clarity**: Each daemon has clear responsibility
- **Flexibility**: Can customize individual daemons
- **Debugging**: Easy to understand which daemon made which decision
- **Modularity**: Separation of concerns
- **Testability**: Each daemon can be tested independently

### Disadvantages
- **Speed**: Multiple LLM calls are slower
- **Latency**: Multiple round-trips
- **Cost**: More API calls
- **Complexity**: More coordination needed

## Hybrid Approach

A hybrid approach could combine both:

```
Fast Path for simple queries:
  Single LLM call → Response

Modular Path for complex queries:
  Router → (Clarifier | Context Assembler | Plan Generator) → Executor → Evaluator → Reflector

Decision: Router decides which path to use based on query complexity
```

## Implementation Considerations

### Prompt Engineering
If using fast path, the prompt must:
- Clearly specify all daemon responsibilities
- Provide examples of each type of decision
- Include consciousness, working memory, episodic memory
- Request structured output (JSON, YAML, etc.)

### Output Format
Fast path output should include:
- Urgency level
- Clarification needed (yes/no + questions)
- Relevant context
- Constraint violations (if any)
- Execution plan (if needed)
- Tool calls (if needed)
- Response to user

### Fallback Strategy
If fast path fails or produces unexpected output:
- Fall back to modular approach
- Use individual daemons for decision-making
- Provide more detailed guidance

## Technology Stack Implications

### LLM Choice
- **Fast LLMs** (e.g., Claude Haiku): Fast path may be necessary for speed
- **Slow LLMs** (e.g., Claude Opus): Modular approach may be acceptable

### Infrastructure
- **Low latency infrastructure**: Modular approach is feasible
- **High latency infrastructure**: Fast path is necessary

### Cost Constraints
- **Low cost**: Fast path reduces API calls
- **High budget**: Modular approach is acceptable

### Accuracy Requirements
- **High accuracy**: Modular approach allows specialized prompts
- **Good enough accuracy**: Fast path is acceptable

## Recommendation

**Start with modular design, optimize with fast path if needed.**

1. **Phase 1**: Implement modular architecture as documented
2. **Phase 2**: Measure performance and identify bottlenecks
3. **Phase 3**: If speed is insufficient, implement fast path
4. **Phase 4**: Monitor fast path accuracy and adjust as needed

This approach ensures:
- Clear specification from the start
- Flexibility to optimize later
- Ability to measure impact of optimization
- Fallback to modular approach if needed

## Key Principle

**The architecture is independent of implementation.** Whether implemented as modular daemons or a single fast path LLM call, the turn architecture remains the same. The daemons are a specification of what needs to happen, not necessarily how it's implemented.

