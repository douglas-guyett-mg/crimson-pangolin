# Episodic Memory Compaction: Threads, Episodes, and Cross-Thread Linking

**Status:** Specification v1.0  
**Date:** 2025-11-05  
**Technology:** Agnostic

## Overview

Episodic Memory Compaction manages the organization and retrieval of memories across conversation threads and episodes. It handles:
- **Thread boundary detection** (explicit vs. implicit channels)
- **Episode boundary detection** (semantic linking across threads)
- **Cross-thread episode creation** (grouping semantically related threads)
- **Aggressive context retrieval** (pulling context from previous threads)

---

## 1. Conversation Threads

### 1.1 What is a Thread?

A **conversation thread** is a cross-channel semantic grouping of related turns. Threads transcend individual channels, allowing users to continue conversations across SMS, WhatsApp, web app, mobile app, etc.

**Key characteristics:**
- Cross-channel (not tied to specific channel)
- Semantic grouping (related by topic/context)
- User-centric (one user, multiple threads)
- Temporal boundaries (explicit or implicit)

### 1.2 Thread Boundary Detection

Thread boundaries are detected differently depending on channel type:

#### 1.2.1 Explicit Channels (Mobile Chat App, Web App)

User explicitly selects which thread to continue:

```
User interface shows:
  ├─ Thread 1: "Weather in New York"
  ├─ Thread 2: "Flight booking"
  └─ Thread 3: "Restaurant recommendations"

User clicks "Weather in New York" → Turn added to Thread 1
```

**Boundary detection:**
- User's explicit selection
- No timeout needed
- Clear thread continuity

#### 1.2.2 Implicit Channels (SMS, WhatsApp, Email)

Thread boundary determined by inactivity timeout:

```
Thread 1: "Weather in New York"
  ├─ Turn 1 (10:00): "What's the weather?"
  ├─ Turn 2 (10:05): "What about next week?"
  └─ Last activity: 10:05

[Inactivity period: 10:05 - 10:45 = 40 minutes]

Turn 3 (10:45): "Oh, and what is it on Thursday?"
  → New thread created (>30 min inactivity)
  → Thread 2: "Weather follow-up"
```

**Boundary detection algorithm:**
```
When new message arrives:
  1. Get last_activity_at from current thread
  2. Calculate inactivity_duration = now - last_activity_at
  3. If inactivity_duration > inactivity_timeout (default: 30 min):
       Create new thread
       Set thread_id = new_id
       Set created_at = now
  4. Else:
       Continue current thread
       Update last_activity_at = now
```

**Configuration:**
- `inactivity_timeout_minutes`: Default 30 minutes
- Can be customized per channel or user
- Configurable per thread

### 1.3 Thread Metadata

Each thread stores:
```
{
  thread_id: "thread_abc123",
  user_id: "user_xyz",
  created_at: "2025-11-05T10:00:00Z",
  last_activity_at: "2025-11-05T10:45:00Z",
  channels: ["sms", "whatsapp"],  // Channels used in this thread
  inactivity_timeout_minutes: 30,
  explicit_channel: false,  // true if user explicitly selected
  primary_episode_id: "episode_1",
  cross_thread_episodes: ["episode_multi_1"],  // Linked episodes
  turn_count: 3,
  summary: "User asking about weather in New York"
}
```

---

## 2. Episodes and Cross-Thread Linking

### 2.1 Primary Episodes (Per-Thread)

Each thread has a primary episode:

```
Thread 1: "Weather in New York"
  ├─ Turn 1: "What's the weather?"
  ├─ Turn 2: "What about next week?"
  └─ Episode 1 (primary)
```

**Primary episode contains:**
- All turns in the thread
- Micro/meso/macro summaries
- Extracted skills and patterns
- Metadata (created_at, turn_count, etc.)

### 2.2 Cross-Thread Episodes (Semantic Linking)

When a new thread is semantically related to previous threads, create a **cross-thread episode**:

```
Thread 1: "Weather in New York" (10:00-10:15)
  └─ Episode 1 (primary)

[30 minutes inactivity]

Thread 2: "Weather follow-up" (10:45)
  ├─ Turn 3: "Oh, and what is it on Thursday?"
  ├─ Episode 2 (primary)
  └─ Episode Multi-1 (cross-thread, links Thread 1 + Thread 2)
```

**Cross-thread episode contains:**
- Turns from multiple threads
- Linked primary episodes
- Combined summaries
- Shared context

### 2.3 Semantic Linking Algorithm (Hybrid Approach)

When a new turn arrives, detect if it's semantically related to previous threads using a **hybrid approach** that runs both algorithmic and LLM-based assessment in parallel:

```
New turn arrives: "Oh, and what is it on Thursday?"

1. Check thread boundary
   → New thread created (>30 min inactivity)

2. Detect semantic continuity (PARALLEL EXECUTION)

   PATH A: Algorithmic Scoring
   ├─ Query episodic memory: "What previous threads are related?"
   ├─ Search by:
   │  - Keywords: "weather", "Thursday", "New York"
   │  - Entities: Location (New York), Time (Thursday)
   │  - Topics: Weather-related queries
   │  - Recency: Recent threads (last 24 hours)
   ├─ Score matches by relevance
   └─ Result: algorithmic_score = 0.88

   PATH B: LLM Assessment (PARALLEL)
   ├─ Pass to LLM:
   │  - Previous thread summary: "User asked about weather in New York"
   │  - Previous turns: ["What's the weather in New York?", "What about next week?"]
   │  - Current message: "Oh, and what is it on Thursday?"
   ├─ LLM prompt: "Are these semantically related? Explain your reasoning."
   └─ Result: llm_assessment = {related: true, confidence: 0.92, reasoning: "..."}

3. Compare Results
   ├─ algorithmic_score = 0.88 (> 0.7 threshold) → LINK
   ├─ llm_assessment.related = true → LINK
   ├─ Agreement: YES
   └─ Confidence: HIGH (both agree)

4. Create cross-thread episode
   a. Link Thread 1 + Thread 2
   b. Set episode_type = "cross_thread"
   c. Store linked_threads = [thread_1_id, thread_2_id]
   d. Generate combined summary
   e. Record both scores for tuning: {algorithmic: 0.88, llm: 0.92}

5. Update thread metadata
   a. Thread 2: cross_thread_episodes = [episode_multi_1]
   b. Thread 1: cross_thread_episodes = [episode_multi_1]
```

**Algorithmic Continuity Scoring:**
```
score = (keyword_match * 0.3) +
        (entity_match * 0.3) +
        (topic_match * 0.2) +
        (recency_score * 0.2)

If score > 0.7: Algorithmic assessment = LINK
```

**LLM Assessment:**
```
Input to LLM:
  - Previous thread summary
  - Previous turns (last N turns)
  - Current message
  - Instruction: "Assess if these are semantically related"

Output from LLM:
  {
    related: boolean,
    confidence: 0.0-1.0,
    reasoning: string,
    key_signals: [string]  // What made you decide?
  }

If related = true: LLM assessment = LINK
```

**Decision Logic:**
```
If algorithmic_score > 0.7 AND llm_assessment.related = true:
  → LINK (high confidence)

If algorithmic_score > 0.7 XOR llm_assessment.related = true:
  → UNCERTAIN (record for tuning)
  → Use higher confidence signal

If algorithmic_score ≤ 0.7 AND llm_assessment.related = false:
  → DON'T LINK (high confidence)
```

### 2.4 Active Tuning of Semantic Linking

The system actively tracks and tunes semantic linking performance:

**Tracking Metrics:**
```
For each linking decision, record:
  - algorithmic_score: 0.0-1.0
  - llm_confidence: 0.0-1.0
  - decision: LINK or DON'T_LINK
  - agreement: YES or NO (did both approaches agree?)
  - outcome: CORRECT or INCORRECT (did user confirm/deny?)
  - timestamp: when decision was made
```

**Outcome Validation:**
```
After linking decision, monitor for user feedback:

If threads were linked:
  - User continues conversation naturally → CORRECT
  - User says "that's not what I meant" → INCORRECT
  - User switches to different topic → INCORRECT

If threads were NOT linked:
  - User asks "didn't we talk about this?" → INCORRECT
  - User starts fresh conversation → CORRECT
  - User continues with new topic → CORRECT
```

**Tuning Process:**
```
Periodically (e.g., every 100 linking decisions):

1. Analyze disagreements
   - When algorithmic and LLM disagree, which is more often correct?
   - Adjust weights or thresholds accordingly

2. Analyze threshold performance
   - Current 0.7 threshold: what's the false positive/negative rate?
   - Should threshold be 0.65? 0.75? 0.8?
   - Adjust based on user feedback patterns

3. Improve LLM assessment
   - What signals does LLM miss that algorithm catches?
   - What signals does algorithm miss that LLM catches?
   - Update LLM prompt to include missing signals

4. Update configuration
   - New threshold value
   - New LLM prompt
   - New weighting for algorithmic scoring
```

**Tuning Metadata:**
```
Store tuning history:
  {
    tuning_date: "2025-11-07",
    previous_threshold: 0.7,
    new_threshold: 0.72,
    reason: "False positive rate was 8%, reduced to 5%",
    metrics: {
      accuracy_before: 0.92,
      accuracy_after: 0.94,
      false_positive_rate_before: 0.08,
      false_positive_rate_after: 0.05,
      false_negative_rate_before: 0.02,
      false_negative_rate_after: 0.03
    }
  }
```

---

## 3. Aggressive Context Retrieval

### 3.1 Context Assembly for Implicit Channels

When assembling working memory for a turn in an implicit channel, aggressively pull context from previous threads:

```
Turn 3 (implicit channel): "Oh, and what is it on Thursday?"

Context assembly:
  1. Get current thread context (Thread 2)
  2. Check for cross-thread episodes
     → Found: Episode Multi-1 (links Thread 1)
  3. Pull context from Thread 1
     - Previous question: "What's the weather in New York?"
     - Previous answer: "Sunny, high 72°F"
     - Location: New York
  4. Inject into working memory:
     "You were asking about weather in New York. 
      You asked about next week. Now asking about Thursday."
  5. Assemble full context with Thread 1 + Thread 2
```

### 3.2 Context Retrieval Strategy

**For explicit channels:**
- User selected thread explicitly
- Pull context from that thread only
- Minimal cross-thread pulling

**For implicit channels:**
- Aggressive cross-thread pulling
- Pull from recent related threads
- Include summaries from previous threads
- Highlight continuity: "You were asking about..."

### 3.3 Context Injection

When injecting context into prompts:

```
System prompt includes:
  "You were discussing weather in New York.
   Previous questions:
   - What's the weather tomorrow? (Sunny, high 72°F)
   - What about next week? (Mixed, 65-75°F)
   
   Current question: What is it on Thursday?
   (Continuing from previous thread after 30 min break)"
```

---

## 4. Episode Compaction

### 4.1 Compaction Triggers

Episodes are compacted when:
- Thread ends (new thread created)
- Episode reaches size threshold (e.g., 50 turns)
- Time threshold reached (e.g., 24 hours)
- Manual trigger (user closes thread)

### 4.2 Compaction Process

```
1. Collect all turns in episode
2. Generate multi-resolution summaries
   - Micro: 1-3 sentences
   - Meso: Paragraph-length
   - Macro: Distilled patterns/skills
3. Extract skills and patterns
4. Create episode summary
5. Archive raw turns (keep for audit)
6. Store compacted episode
```

### 4.3 Retrieval After Compaction

After compaction, retrieve via:
- Episode summary (for context assembly)
- Extracted skills (for skill reuse)
- Linked episodes (for cross-thread context)
- Raw turns (for detailed analysis)

---

## 5. Integration with Memory Systems

### 5.1 Episodic Memory Store

Stores:
- Threads (metadata, turn references)
- Episodes (primary and cross-thread)
- Compacted summaries
- Extracted skills

### 5.2 Working Memory Assembly

When assembling working memory:
```
1. Get current thread
2. Check for cross-thread episodes
3. Pull context from linked threads
4. Assemble full context
5. Inject into prompt
```

### 5.3 Skill Extraction

Skills extracted from:
- Primary episodes (per-thread patterns)
- Cross-thread episodes (multi-thread patterns)
- Compacted summaries (distilled patterns)

---

## 6. Test Conditions

### 6.1 Thread Boundary Detection Tests
- ✅ Explicit channel: User selection creates thread boundary
- ✅ Implicit channel: 30-min inactivity creates thread boundary
- ✅ Implicit channel: <30-min activity continues thread
- ✅ Thread metadata stored correctly

### 6.2 Semantic Linking Tests
- ✅ Related turns detected and linked
- ✅ Cross-thread episode created
- ✅ Semantic continuity score calculated
- ✅ Threshold-based linking works

### 6.3 Context Retrieval Tests
- ✅ Explicit channel: Context from current thread only
- ✅ Implicit channel: Aggressive cross-thread pulling
- ✅ Context injected into prompts correctly
- ✅ Continuity highlighted in context

### 6.4 Compaction Tests
- ✅ Episodes compacted on trigger
- ✅ Summaries generated correctly
- ✅ Skills extracted from episodes
- ✅ Retrieval works after compaction

### 6.5 Integration Tests
- ✅ Thread creation and linking
- ✅ Episode creation and compaction
- ✅ Context assembly with cross-thread pulling
- ✅ Skill extraction across threads

---

## 7. Relationship to Other Systems

- **Conversation History Plugin**: Stores utterances with turn tracking
- **Multi-Resolution Summarizer**: Generates summaries for episodes
- **Skill Extraction**: Extracts patterns from episodes
- **Working Memory Orchestrator**: Assembles context with cross-thread pulling
- **Turn Trace**: Logs thread and episode operations

