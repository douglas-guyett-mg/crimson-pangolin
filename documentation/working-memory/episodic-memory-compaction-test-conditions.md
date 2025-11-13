# Episodic Memory Compaction — Test Conditions

**Status:** v1.0  
**Date:** 2025-11-05

## Test Category 1: Thread Boundary Detection (Explicit Channels)

### Test 1.1: User Explicitly Selects Thread
**Scenario:** User has multiple threads and explicitly selects one
```
Setup:
  - Thread 1: "Weather in New York"
  - Thread 2: "Flight booking"
  - User clicks Thread 1

Action:
  - Send message: "What about Thursday?"

Expected:
  - Turn added to Thread 1
  - last_activity_at updated
  - No new thread created
  - Thread 1 turn_count incremented
```

### Test 1.2: User Switches Between Threads
**Scenario:** User switches from Thread 1 to Thread 2
```
Setup:
  - Thread 1: Last activity 10:00
  - Thread 2: Last activity 09:00
  - User clicks Thread 2

Action:
  - Send message: "What about return flights?"

Expected:
  - Turn added to Thread 2
  - Thread 2 last_activity_at updated
  - Thread 1 unchanged
  - Both threads remain active
```

---

## Test Category 2: Thread Boundary Detection (Implicit Channels)

### Test 2.1: New Thread on Inactivity Timeout
**Scenario:** User sends message after 30+ minutes of inactivity
```
Setup:
  - Thread 1: Last activity 10:00
  - Current time: 10:45 (45 minutes later)
  - inactivity_timeout_minutes: 30

Action:
  - Send message: "Oh, and what is it on Thursday?"

Expected:
  - New thread created (Thread 2)
  - Thread 2 created_at = 10:45
  - Thread 2 turn_count = 1
  - Thread 1 remains unchanged
  - Thread 1 last_activity_at = 10:00
```

### Test 2.2: Continue Thread Within Timeout
**Scenario:** User sends message within 30 minutes
```
Setup:
  - Thread 1: Last activity 10:00
  - Current time: 10:15 (15 minutes later)
  - inactivity_timeout_minutes: 30

Action:
  - Send message: "What about next week?"

Expected:
  - Turn added to Thread 1
  - Thread 1 last_activity_at = 10:15
  - No new thread created
  - Thread 1 turn_count incremented
```

### Test 2.3: Custom Inactivity Timeout
**Scenario:** Thread has custom timeout (60 minutes)
```
Setup:
  - Thread 1: Last activity 10:00, inactivity_timeout_minutes: 60
  - Current time: 10:45 (45 minutes later)

Action:
  - Send message: "What about next week?"

Expected:
  - Turn added to Thread 1 (45 < 60)
  - No new thread created
  - Thread 1 last_activity_at = 10:45
```

---

## Test Category 3: Semantic Linking and Cross-Thread Episodes

### Test 3.1: Detect Semantic Continuity (Algorithmic)
**Scenario:** New thread is semantically related to previous thread
```
Setup:
  - Thread 1: "What's the weather in New York?"
  - Thread 1: "What about next week?"
  - [30 min inactivity]
  - Thread 2 created: "Oh, and what is it on Thursday?"

Action:
  - Algorithmic scoring runs
  - Semantic linking algorithm calculates score

Expected:
  - Algorithmic score > 0.7 (e.g., 0.88)
  - Keywords match: "weather", "New York", "Thursday"
  - Entities match: Location (New York), Time (Thursday)
  - Algorithmic assessment: LINK
```

### Test 3.1b: Detect Semantic Continuity (LLM)
**Scenario:** LLM assessment confirms algorithmic linking
```
Setup:
  - Thread 1: "What's the weather in New York?"
  - Thread 1: "What about next week?"
  - [30 min inactivity]
  - Thread 2 created: "Oh, and what is it on Thursday?"

Action:
  - LLM assessment runs in parallel
  - LLM receives: previous thread summary + current message
  - LLM assesses: "Are these semantically related?"

Expected:
  - LLM assessment: related = true
  - LLM confidence > 0.7 (e.g., 0.92)
  - LLM reasoning includes: "Both about weather in New York"
  - LLM key_signals: ["weather", "New York", "temporal_continuation"]
```

### Test 3.1c: Parallel Execution and Agreement
**Scenario:** Both algorithmic and LLM assessments run in parallel and agree
```
Setup:
  - Thread 1: "What's the weather in New York?"
  - Thread 1: "What about next week?"
  - [30 min inactivity]
  - Thread 2 created: "Oh, and what is it on Thursday?"

Action:
  - Both algorithmic and LLM assessments run in parallel
  - Results are compared

Expected:
  - Algorithmic score: 0.88 (> 0.7) → LINK
  - LLM assessment: related = true → LINK
  - Agreement: YES
  - Confidence: HIGH
  - Cross-thread episode created
  - Both scores recorded: {algorithmic: 0.88, llm: 0.92}
```

### Test 3.2: No Semantic Continuity (Algorithmic)
**Scenario:** New thread is unrelated to previous thread
```
Setup:
  - Thread 1: "What's the weather in New York?"
  - [30 min inactivity]
  - Thread 2 created: "Book me a flight to Tokyo"

Action:
  - Algorithmic scoring runs

Expected:
  - Algorithmic score < 0.7 (e.g., 0.35)
  - No keyword/entity/topic overlap
  - Algorithmic assessment: DON'T_LINK
```

### Test 3.2b: No Semantic Continuity (LLM)
**Scenario:** LLM assessment confirms algorithmic non-linking
```
Setup:
  - Thread 1: "What's the weather in New York?"
  - [30 min inactivity]
  - Thread 2 created: "Book me a flight to Tokyo"

Action:
  - LLM assessment runs in parallel

Expected:
  - LLM assessment: related = false
  - LLM confidence > 0.7 (e.g., 0.88)
  - LLM reasoning: "Different topics - weather vs. flight booking"
```

### Test 3.2c: Parallel Execution and Agreement (No Link)
**Scenario:** Both assessments agree NOT to link
```
Setup:
  - Thread 1: "What's the weather in New York?"
  - [30 min inactivity]
  - Thread 2 created: "Book me a flight to Tokyo"

Action:
  - Both assessments run in parallel

Expected:
  - Algorithmic score: 0.35 (< 0.7) → DON'T_LINK
  - LLM assessment: related = false → DON'T_LINK
  - Agreement: YES
  - Confidence: HIGH
  - No cross-thread episode created
  - Both scores recorded: {algorithmic: 0.35, llm: 0.12}
```

### Test 3.2d: Disagreement - Algorithmic Says Link, LLM Says No
**Scenario:** Assessments disagree on linking decision
```
Setup:
  - Thread 1: "What's the weather in New York?"
  - Thread 2: "What about the weather in Boston?"

Action:
  - Algorithmic score: 0.72 (> 0.7) → LINK
  - LLM assessment: related = false (confidence 0.65) → DON'T_LINK
  - Disagreement detected

Expected:
  - Agreement: NO
  - Confidence: UNCERTAIN
  - Decision: Use higher confidence signal
    - Algorithmic confidence: 0.72
    - LLM confidence: 0.65
    - Use algorithmic (higher confidence) → LINK
  - Record disagreement for tuning: {algorithmic: 0.72, llm: 0.35, disagreement: true}
```

### Test 3.2e: Disagreement - Algorithmic Says No, LLM Says Link
**Scenario:** LLM detects continuity that algorithm misses
```
Setup:
  - Thread 1: "I'm planning a trip to Paris"
  - Thread 2: "What's the best time to visit?"

Action:
  - Algorithmic score: 0.68 (< 0.7) → DON'T_LINK
  - LLM assessment: related = true (confidence 0.85) → LINK
  - Disagreement detected

Expected:
  - Agreement: NO
  - Confidence: UNCERTAIN
  - Decision: Use higher confidence signal
    - Algorithmic confidence: 0.68
    - LLM confidence: 0.85
    - Use LLM (higher confidence) → LINK
  - Record disagreement for tuning: {algorithmic: 0.68, llm: 0.85, disagreement: true}
```

### Test 3.3: Active Tuning - Outcome Validation
**Scenario:** System tracks outcomes and validates linking decisions
```
Setup:
  - 100 linking decisions made over time
  - Each decision recorded: {algorithmic_score, llm_confidence, decision, agreement}

Action:
  - Monitor user behavior after each linking decision
  - Record outcomes: CORRECT or INCORRECT

Expected:
  - For LINK decisions:
    - User continues conversation naturally → CORRECT
    - User says "that's not what I meant" → INCORRECT
  - For DON'T_LINK decisions:
    - User asks "didn't we talk about this?" → INCORRECT
    - User starts fresh conversation → CORRECT
  - Outcomes recorded in Turn Trace
```

### Test 3.4: Active Tuning - Threshold Adjustment
**Scenario:** System analyzes performance and adjusts threshold
```
Setup:
  - 100 linking decisions with outcomes
  - Current threshold: 0.7
  - Metrics:
    - Accuracy: 92%
    - False positive rate: 8%
    - False negative rate: 2%

Action:
  - Tuning process runs
  - Analyzes: "Should threshold be higher to reduce false positives?"
  - Calculates: "If threshold = 0.72, false positive rate drops to 5%"

Expected:
  - New threshold: 0.72
  - Tuning metadata recorded:
    {
      tuning_date: "2025-11-07",
      previous_threshold: 0.7,
      new_threshold: 0.72,
      reason: "Reduce false positive rate",
      metrics: {
        accuracy_before: 0.92,
        accuracy_after: 0.94,
        false_positive_rate_before: 0.08,
        false_positive_rate_after: 0.05
      }
    }
```

### Test 3.5: Active Tuning - LLM Prompt Improvement
**Scenario:** System improves LLM assessment based on disagreements
```
Setup:
  - 100 linking decisions
  - Disagreements analyzed:
    - Algorithm: 0.72 (LINK), LLM: 0.35 (NO_LINK) → Outcome: CORRECT (should link)
    - Algorithm: 0.68 (NO_LINK), LLM: 0.85 (LINK) → Outcome: CORRECT (should link)
  - Pattern: LLM misses temporal continuity signals

Action:
  - Tuning process identifies: "LLM prompt should emphasize temporal signals"
  - Updates LLM prompt to include: "Look for temporal continuity (e.g., 'Oh, and...', 'What about...')"

Expected:
  - New LLM prompt includes temporal signal guidance
  - Next 100 decisions show improved LLM accuracy
  - Fewer disagreements with algorithm
```

### Test 3.6: Multiple Cross-Thread Episodes
**Scenario:** Thread linked to multiple previous threads
```
Setup:
  - Thread 1: "Weather in New York"
  - Thread 2: "Weather in Boston"
  - [30 min inactivity]
  - Thread 3: "What about weather in both cities?"

Action:
  - Semantic linking algorithm runs

Expected:
  - Thread 3 linked to both Thread 1 and Thread 2
  - Cross-thread episode created with all three threads
  - linked_threads = [thread_1_id, thread_2_id, thread_3_id]
  - All threads updated with cross_thread_episodes reference
```

---

## Test Category 4: Context Retrieval

### Test 4.1: Explicit Channel Context Retrieval
**Scenario:** User on explicit channel (web app)
```
Setup:
  - Thread 1: "Weather in New York"
  - User explicitly selected Thread 1
  - New turn: "What about Thursday?"

Action:
  - Assemble working memory

Expected:
  - Context from Thread 1 only
  - No cross-thread pulling
  - Prompt includes: "You were asking about weather in New York"
  - Previous answers included
```

### Test 4.2: Implicit Channel Aggressive Pulling
**Scenario:** User on implicit channel (SMS)
```
Setup:
  - Thread 1: "Weather in New York" (10:00-10:15)
  - Thread 2: "Weather follow-up" (10:45)
  - Cross-thread episode links Thread 1 + Thread 2
  - New turn in Thread 2: "What about Thursday?"

Action:
  - Assemble working memory

Expected:
  - Context from Thread 2 (current)
  - Context from Thread 1 (previous, via cross-thread episode)
  - Prompt includes: "You were asking about weather in New York"
  - Continuity highlighted: "Continuing from previous thread"
  - Previous answers included
```

### Test 4.3: Context Injection Format
**Scenario:** Context injected into prompt
```
Setup:
  - Thread 1: Previous weather questions
  - Thread 2: Current follow-up
  - Cross-thread episode exists

Action:
  - Generate system prompt

Expected:
  - Prompt structure:
    "You were discussing weather in New York.
     Previous questions:
     - What's the weather tomorrow? (Sunny, 72°F)
     - What about next week? (Mixed, 65-75°F)
     
     Current question: What is it on Thursday?
     (Continuing from previous thread after 30 min break)"
```

---

## Test Category 5: Episode Compaction

### Test 5.1: Compaction on Thread End
**Scenario:** Thread ends (new thread created)
```
Setup:
  - Thread 1: 5 turns about weather
  - [30 min inactivity]
  - Thread 2 created

Action:
  - Compaction triggered for Thread 1

Expected:
  - Episode 1 compacted
  - Summaries generated (micro/meso/macro)
  - Skills extracted
  - Raw turns archived
  - Episode 1 marked as compacted
```

### Test 5.2: Compaction on Size Threshold
**Scenario:** Episode reaches 50 turns
```
Setup:
  - Thread 1: 50 turns
  - Turn 51 arrives

Action:
  - Compaction triggered

Expected:
  - Episode 1 compacted
  - Summaries generated
  - Skills extracted
  - Turn 51 starts new episode
```

### Test 5.3: Retrieval After Compaction
**Scenario:** Retrieve context from compacted episode
```
Setup:
  - Episode 1 compacted
  - New turn in Thread 2 (linked via cross-thread episode)

Action:
  - Assemble working memory

Expected:
  - Episode 1 summary retrieved
  - Skills from Episode 1 available
  - Context injected correctly
  - Raw turns accessible if needed
```

---

## Test Category 6: Integration Tests

### Test 6.1: Full Workflow (Implicit Channel)
**Scenario:** Complete workflow on SMS
```
Setup:
  - User sends: "What's the weather in New York?" (10:00)
  - Si responds
  - User sends: "What about next week?" (10:05)
  - Si responds
  - [30 min inactivity]
  - User sends: "Oh, and what is it on Thursday?" (10:45)

Expected:
  - Thread 1 created (turns 1-2)
  - Thread 2 created (turn 3)
  - Cross-thread episode created
  - Context from Thread 1 pulled into Thread 2
  - Si understands continuity
  - Response includes: "You were asking about weather in New York"
```

### Test 6.2: Full Workflow (Explicit Channel)
**Scenario:** Complete workflow on web app
```
Setup:
  - User creates Thread 1: "Weather in New York"
  - User creates Thread 2: "Flight booking"
  - User clicks Thread 1
  - User sends: "What about Thursday?"

Expected:
  - Turn added to Thread 1
  - Context from Thread 1 only
  - No cross-thread pulling
  - Si responds about weather
```

### Test 6.3: Cross-Channel Continuity
**Scenario:** User switches channels mid-thread
```
Setup:
  - User on SMS: "What's the weather in New York?" (10:00)
  - Si responds
  - User switches to WhatsApp: "What about next week?" (10:05)

Expected:
  - Same thread continues (cross-channel)
  - Thread metadata includes both channels
  - Context maintained across channels
  - Si understands continuity
```

---

## Test Category 7: Edge Cases

### Test 7.1: Rapid Thread Switching
**Scenario:** User rapidly switches between threads
```
Setup:
  - Thread 1, Thread 2, Thread 3 exist
  - User sends to Thread 1, then Thread 2, then Thread 3 rapidly

Expected:
  - All turns added to correct threads
  - No cross-thread episodes created (different topics)
  - Each thread maintains correct state
```

### Test 7.2: Ambiguous Semantic Continuity
**Scenario:** New thread could be related to multiple previous threads
```
Setup:
  - Thread 1: "Weather in New York"
  - Thread 2: "Weather in Boston"
  - Thread 3: "What about weather?"

Expected:
  - Semantic linking algorithm scores all matches
  - Links to highest-scoring threads
  - Creates cross-thread episode with all related threads
  - Handles ambiguity gracefully
```

### Test 7.3: Very Old Thread Reactivation
**Scenario:** User references thread from 1 week ago
```
Setup:
  - Thread 1: "Weather in New York" (1 week ago)
  - [7 days inactivity]
  - New message: "Remember when we talked about New York weather?"

Expected:
  - New thread created (>30 min inactivity)
  - Semantic linking detects reference to Thread 1
  - Cross-thread episode created
  - Context from Thread 1 retrieved
  - Si understands reference to old thread
```

