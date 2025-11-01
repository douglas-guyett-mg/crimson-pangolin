# Responder Hobgoblin

## Purpose

The Responder generates responses to send to the user. It transforms Si's internal work (plans, results, decisions) into natural language that the user can understand and act on.

## Responsibilities

1. **Receive Input**: Accept information to communicate
2. **Determine Response Type**: Decide what kind of response is needed
3. **Generate Response**: Create natural language response
4. **Optimize for User**: Tailor response to user's needs
5. **Send Response**: Deliver response to user

## When Responder is Activated

Responder is activated:
- When immediate response is needed (high urgency)
- When clarification is needed (Clarifier)
- When results are ready (Executor)
- When turn is complete (Evaluator)
- When error occurs (Error Handler)

## Response Types

### Immediate Response (High Urgency)
**When**: Urgent situation requires immediate guidance
**Purpose**: Prevent user from taking harmful action
**Example**: "Stop, don't do anything. Let me contact your bank to verify."

### Clarification Request
**When**: Input is ambiguous
**Purpose**: Gather missing information
**Example**: "Which bug are you referring to? Can you describe the symptoms?"

### Partial Response (Medium Urgency)
**When**: Initial analysis complete, deeper work continues
**Purpose**: Provide early feedback while work continues
**Example**: "I found several flights. Let me check hotels while you review these options."

### Complete Response (Low Urgency)
**When**: All work complete
**Purpose**: Provide comprehensive answer
**Example**: "Here are the best flight and hotel options with comparison and recommendations."

### Follow-up Response
**When**: Background work completes after initial response
**Purpose**: Provide additional information
**Example**: "I confirmed with your bank - that email is fraudulent. Ignore it."

### Error Response
**When**: Error occurs
**Purpose**: Inform user and suggest alternatives
**Example**: "The flight search failed. Let me try an alternative search."

## Decision Inputs

Responder makes decisions using:

### Consciousness
- **Communication Style**: How should Si communicate?
- **Values**: What matters in communication?
- **Governance**: What constraints apply?

### Working Memory
- **User Profile**: Who is the user?
- **Conversation History**: What's been discussed?
- **Turn Context**: What's the situation?
- **Results**: What information is available?

### Episodic Memory
- **Similar Responses**: What responses worked before?
- **Learned Patterns**: What communication patterns are effective?
- **User Preferences**: What does this user prefer?
- **Outcomes**: What responses succeeded?

### Input
- **Information to Communicate**: What needs to be said?
- **Response Type**: What kind of response?
- **Urgency**: How urgent is this?

## Response Generation Process

```
Responder receives information to communicate
    ↓
Determine response type:
  - Immediate response?
  - Clarification request?
  - Partial response?
  - Complete response?
  - Follow-up response?
  - Error response?
    ↓
Gather context:
  - Who is the user?
  - What's their background?
  - What have they already been told?
  - What do they need to know?
    ↓
Generate response:
  - What's the main message?
  - What supporting details?
  - What tone is appropriate?
  - What format is best?
    ↓
Optimize for user:
  - Is it clear?
  - Is it concise?
  - Is it actionable?
  - Is it appropriate for user?
    ↓
Send response
```

## Response Quality Criteria

### Clarity
- Is the response clear and understandable?
- Would the user understand what to do?
- Are there ambiguities?

### Conciseness
- Is the response appropriately brief?
- Is unnecessary information removed?
- Is important information included?

### Actionability
- Can the user act on this response?
- Are next steps clear?
- Is guidance provided?

### Appropriateness
- Is the tone appropriate?
- Is the complexity appropriate?
- Is the format appropriate?

### Completeness
- Is all necessary information included?
- Are there gaps?
- Are there unanswered questions?

## Response Example

**Scenario**: Travel booking turn

**Information to Communicate**:
- 5 flight options found
- 4 hotel options found
- Comparison matrix created
- User can now decide

**Response Type**: Complete response (low urgency)

**Generated Response**:
```
I found great options for your Paris trip! Here's what I found:

FLIGHTS (March 15-22):
1. Air France - $450 (direct, 7h)
2. United - $380 (1 stop, 9h)
3. Delta - $420 (direct, 7h)

HOTELS (near airport):
1. Hilton - $120/night (4-star)
2. Marriott - $100/night (3-star)
3. Budget Inn - $60/night (2-star)

RECOMMENDATION:
For best value, I'd suggest United flight ($380) + Marriott hotel ($100/night) = $1,280 total

Ready to book? Just let me know which options you prefer!
```

**Quality Assessment**:
- Clarity: HIGH (options are clear)
- Conciseness: GOOD (includes essentials)
- Actionability: HIGH (user knows what to do)
- Appropriateness: GOOD (tone is helpful)
- Completeness: HIGH (all info included)

## Learning Through Responder

Responder improves through the learning loop:

**Feedback Sources**:
- Did the user understand the response?
- Did the user act on the response?
- Was the response helpful?
- Did the user ask follow-up questions?
- Did the user express satisfaction?

**Learning Updates**:
- Episodic Memory: Store successful response patterns
- Skills: Improve response generation for similar situations
- Consciousness: Refine communication style if needed

**Example**:
- Turn 1: Responder generates travel booking response
- User understands and books successfully
- Evaluator: Response was effective → GOOD
- Reflector: Learn "This response format works well for travel"

## Key Principle

**Responder is the voice of Si.** It transforms internal work into user-facing communication. Responder's quality directly impacts user satisfaction and understanding.

