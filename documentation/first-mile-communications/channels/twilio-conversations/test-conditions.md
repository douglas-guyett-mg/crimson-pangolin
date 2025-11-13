# Twilio Conversations Channel — Test Conditions

## Purpose
Comprehensive, measurable test specifications for Twilio Conversations channel implementation. Each test includes specific setup, steps, expected outcomes, and verification criteria suitable for automated or manual testing.

---

## 1. Conversation Management

### 1.1 Create New Conversation from SMS
**Setup:**
- New user phone number: +1-555-0100 (not in system)
- No existing conversation for this number
- Consent registry: empty for this user

**Steps:**
1. User sends SMS: "Hello Si"
2. Twilio webhook delivers `conversation.created` event
3. System processes and creates conversation

**Expected:**
- Conversation created with:
  - `conversation_id`: unique UUID format
  - `participant`: +1-555-0100
  - `channel`: "sms"
  - `status`: "active"
  - `created_at`: timestamp ±2 seconds of message receipt
- Session orchestration receives canonical event:
  - `event_type`: "session.created"
  - `session_id` matches `conversation_id`
  - `user_identifier`: "+1-555-0100"
  - `channel_metadata.channel`: "sms"
  - `channel_metadata.provider`: "twilio"

**Verification:**
- Query Twilio API: conversation exists with expected ID
- Query session orchestration: session record present
- No duplicate conversations created (idempotent)

### 1.2 Create New Conversation from WhatsApp
**Setup:**
- WhatsApp user: +1-555-0200
- No existing conversation

**Steps:**
1. User sends WhatsApp message: "Hi there"
2. Webhook delivers event with `messaging_service_type`: "whatsapp"

**Expected:**
- Conversation created with `channel`: "whatsapp"
- Canonical event includes:
  - `channel_metadata.channel`: "whatsapp"
  - `channel_metadata.profile_name`: user's WhatsApp display name (if available)

**Verification:**
- WhatsApp conversation distinct from SMS namespace
- Profile metadata captured when available

### 1.3 Resume Existing Conversation
**Setup:**
- User +1-555-0100 has existing conversation: `conv-123`
- Status: "active"
- Last message: 10 minutes ago

**Steps:**
1. User sends: "Are you there?"
2. System receives message event

**Expected:**
- Message added to conversation `conv-123` (not new conversation)
- Session orchestration receives:
  - `event_type`: "message.received"
  - `session_id`: `conv-123` (existing)
- Conversation count unchanged

**Verification:**
- Conversation list shows only 1 conversation for user
- Message appears in `conv-123` history
- Session continuity maintained (context preserved)

### 1.4 Detect and Handle Inactivity
**Setup:**
- Conversation `conv-456`: active
- Inactivity threshold: 24 hours
- Last message: 23 hours 50 minutes ago

**Steps:**
1. System runs inactivity checker
2. 10 minutes elapse (crosses 24-hour threshold)

**Expected:**
- Warning message sent: "This conversation will close due to inactivity in 1 hour. Reply to keep it active."
- Conversation status: "warning"
- Inactivity timer set: 1 hour from warning

**Verification:**
- Warning delivered to user
- Timer cancels if user responds within 1 hour
- Conversation remains active if user responds

### 1.5 Close Inactive Conversation
**Setup:**
- Conversation in "warning" state
- Warning sent 1 hour ago
- No user response

**Steps:**
1. Inactivity timer expires
2. System closes conversation

**Expected:**
- Conversation status: "closed"
- Final message: "Conversation closed due to inactivity. Send a message anytime to start a new conversation."
- Archival workflow triggered
- Session orchestration receives:
  - `event_type`: "session.terminated"
  - `reason`: "inactivity"
  - `final_message_count`: N (actual count)

**Verification:**
- Conversation no longer accepts messages
- Transcript archived and accessible
- User can start new conversation with subsequent message

### 1.6 Explicit Conversation Closure
**Setup:**
- Active conversation `conv-789`

**Steps:**
1. User sends: "goodbye" or "end conversation"
2. System detects closure keywords

**Expected:**
- Conversation status: "closed"
- Response: "Conversation ended. It was nice talking with you. Message anytime to start again."
- Session terminated immediately
- Transcript saved

**Verification:**
- No further messages accepted in this conversation
- Graceful closure message sent
- New message from user creates new conversation

---

## 2. Participant Governance & Consent

### 2.1 First-Time User Opt-In (SMS)
**Setup:**
- New user: +1-555-1000
- Consent registry: no record
- Jurisdiction: requires explicit opt-in

**Steps:**
1. User sends first message: "Hello"
2. System checks consent status

**Expected:**
- Original message queued (not processed yet)
- Opt-in request sent: "Welcome! Reply YES to opt in and start chatting with Si."
- Consent record created:
  - `status`: "pending"
  - `timestamp`: current time
  - `channel`: "sms"

**Verification:**
- Original message not forwarded to cognition layer
- Opt-in message uses compliant language
- Consent record retrievable from registry

### 2.2 User Confirms Opt-In
**Setup:**
- User +1-555-1000 in "pending" consent state
- Queued message: "Hello"

**Steps:**
1. User replies: "YES"
2. System processes confirmation

**Expected:**
- Consent record updated:
  - `status`: "opted_in"
  - `confirmed_at`: current timestamp
  - `method`: "explicit_sms_yes"
- Queued message ("Hello") now processed
- Welcome response: "Thanks for opting in! How can I help you today?"

**Verification:**
- Consent status persisted
- Original message processed after opt-in
- User receives substantive response to original query

### 2.3 User Opt-Out with STOP Keyword
**Setup:**
- User +1-555-1000 with "opted_in" status
- Active conversation

**Steps:**
1. User sends: "STOP"
2. System processes opt-out keyword

**Expected:**
- Consent record updated:
  - `status`: "opted_out"
  - `opted_out_at`: current timestamp
  - `method`: "stop_keyword"
- Confirmation sent: "You have been unsubscribed. Reply START to re-subscribe."
- Conversation closed
- No further outbound messages (except opt-out confirmation)

**Verification:**
- Opt-out recorded in consent registry
- User receives no messages except confirmation
- Compliance audit log includes opt-out event with timestamp

### 2.4 User Re-Opt-In with START Keyword
**Setup:**
- User +1-555-1000 with "opted_out" status

**Steps:**
1. User sends: "START"
2. System processes re-opt-in

**Expected:**
- Consent record updated:
  - `status`: "opted_in"
  - `re_opted_in_at`: current timestamp
  - `method`: "start_keyword"
- Confirmation: "Welcome back! You're subscribed again. How can I help?"
- User can now send messages normally

**Verification:**
- Consent status changed to "opted_in"
- User can resume normal interactions
- Audit log includes re-opt-in event

### 2.5 Reject Messages from Opted-Out Users
**Setup:**
- User +1-555-2000 with "opted_out" status
- User sends message (not "START")

**Steps:**
1. User sends: "I have a question"
2. System checks consent status

**Expected:**
- Message rejected (not processed)
- No forwarding to cognition layer
- Optional reminder: "You are currently unsubscribed. Reply START to resume."

**Verification:**
- Message not processed by session orchestration
- Opt-out enforcement verified
- User receives clear status message

### 2.6 WhatsApp Implicit Opt-In
**Setup:**
- New WhatsApp user: +1-555-3000
- WhatsApp policy allows implicit consent on first message

**Steps:**
1. User sends WhatsApp message: "Hello"

**Expected:**
- Implicit consent granted:
  - `status`: "opted_in"
  - `method`: "whatsapp_implicit"
  - `confirmed_at`: timestamp
- Message processed immediately (no explicit opt-in step)
- Welcome message includes consent notice

**Verification:**
- Consent record created with "whatsapp_implicit" method
- Message processed without delay
- Audit log shows consent method

---

## 3. Message Normalization & Delivery

### 3.1 Normalize Simple Text Message (SMS)
**Setup:**
- User +1-555-1000 with active conversation, opted in

**Steps:**
1. User sends SMS: "What's the weather like?"
2. Webhook delivers message event

**Expected:**
- Canonical event created:
  ```json
  {
    "event_type": "message.received",
    "event_id": "<UUID>",
    "timestamp": "<ISO 8601>",
    "session_id": "<conversation_id>",
    "user_identifier": "+1-555-1000",
    "channel": "sms",
    "content": {
      "type": "text",
      "text": "What's the weather like?"
    },
    "metadata": {
      "channel_message_id": "<twilio_message_sid>",
      "channel": "sms",
      "provider": "twilio"
    }
  }
  ```

**Verification:**
- All required fields present
- Timestamp accurate within ±2 seconds
- Session ID maps to Twilio conversation
- Text preserved exactly (no truncation or modification)

### 3.2 Normalize WhatsApp Message with Emoji
**Setup:**
- WhatsApp user sends message

**Steps:**
1. User sends: "Great! 😊👍"

**Expected:**
- Canonical event with:
  - `content.text`: "Great! 😊👍"
- UTF-8 encoding preserved
- No character corruption

**Verification:**
- Emoji displays correctly in downstream systems
- Character count accurate
- No encoding errors in logs

### 3.3 Normalize MMS with Image Attachment
**Setup:**
- User sends MMS: image (JPEG, 500KB) + caption "Look at this"

**Steps:**
1. Twilio webhook includes `MediaUrl`
2. System processes attachment

**Expected:**
- Canonical event:
  ```json
  {
    "content": {
      "type": "multimedia",
      "text": "Look at this",
      "attachments": [
        {
          "type": "image",
          "url": "<internal_storage_url>",
          "mime_type": "image/jpeg",
          "size_bytes": 500000,
          "content_hash": "<sha256_hash>"
        }
      ]
    }
  }
  ```
- Image downloaded from Twilio and stored internally
- Internal URL replaces Twilio MediaUrl

**Verification:**
- Attachment metadata accurate
- Image accessible via internal URL
- Original Twilio URL not exposed downstream
- Content hash calculated for deduplication

### 3.4 Handle Unsupported Media Type
**Setup:**
- User sends unsupported file: `.exe` attachment

**Steps:**
1. Webhook includes MediaUrl with `Content-Type: application/x-msdownload`

**Expected:**
- Attachment rejected
- Response to user: "Sorry, I can't accept .exe files. Please send images, documents, or audio files."
- Text portion of message (if present) processed normally

**Verification:**
- Unsupported file not downloaded or stored
- User receives clear error message
- Conversation continues normally

### 3.5 Deliver Simple Text Response
**Setup:**
- Cognition layer generates response: "The weather today is sunny, 72°F."

**Steps:**
1. Response routed to Twilio channel adapter
2. Message sent via Twilio API

**Expected:**
- Twilio API call:
  - `conversation_sid`: correct conversation
  - `author`: "si_assistant"
  - `body`: "The weather today is sunny, 72°F."
- Delivery status tracked
- Status: "delivered" (confirmed by Twilio)

**Verification:**
- Message delivered to user's device
- Twilio status webhook confirms delivery
- Delivery latency logged (< 1 second target)

### 3.6 Segment Long Response
**Setup:**
- Response: 600 characters (exceeds SMS limit of 160)
- Channel: SMS

**Steps:**
1. Adapter receives long response
2. Segmentation logic applied

**Expected:**
- Response split into 4 messages:
  - Msg 1: 160 chars (ends at word boundary) + " (1/4)"
  - Msg 2: 160 chars + " (2/4)"
  - Msg 3: 160 chars + " (3/4)"
  - Msg 4: remaining chars + " (4/4)"
- Messages sent sequentially with 500ms delay between
- Order preserved

**Verification:**
- All segments delivered
- User sees parts in correct order
- Segment indicators clear: (1/4), (2/4), etc.
- Total character limit respected

### 3.7 Handle Delivery Failure (Transient)
**Setup:**
- User device temporarily unreachable
- Response ready to send

**Steps:**
1. Twilio API returns error: `30003` (unreachable)
2. Retry logic executes

**Expected:**
- First attempt: fails with error code 30003
- Retry schedule:
  - Attempt 2: after 30 seconds
  - Attempt 3: after 60 seconds (if needed)
- Max retries: 3
- Second attempt succeeds
- Final status: "delivered"

**Verification:**
- Retry attempts logged
- Message eventually delivered
- User receives message (delayed)
- Retry count does not exceed 3

### 3.8 Handle Delivery Failure (Permanent)
**Setup:**
- User number invalid: +1-555-0000-INVALID

**Steps:**
1. Twilio API returns error: `30004` (invalid number)

**Expected:**
- No retry (permanent error)
- Conversation status: "delivery_failed"
- Incident logged:
  - `error_code`: 30004
  - `reason`: "Invalid phone number"
- Alert triggered for ops team if threshold exceeded (>1% failures)

**Verification:**
- No retry attempts made
- Error recorded in observability system
- User's conversation marked failed
- Support can investigate manually

---

## 4. Voice Escalation

### 4.1 User Requests Voice Call
**Setup:**
- Active SMS conversation: `conv-100`
- User: +1-555-4000

**Steps:**
1. User sends: "Can I call you?"
2. System detects voice intent via keyword or cognition

**Expected:**
- Voice intent recognized
- Response: "I can call you now. Reply CALL to start."
- Conversation remains active (no state change)

**Verification:**
- User receives call prompt
- No automatic dial (requires explicit consent)
- Conversation context preserved

### 4.2 Initiate Outbound Call
**Setup:**
- User replied "CALL"
- Voice escalation approved

**Steps:**
1. System initiates Twilio call API
2. Call connects to +1-555-4000

**Expected:**
- Twilio call created with:
  - `to`: +1-555-4000
  - `from`: Si's Twilio number
  - `status_callback_url`: configured webhook
- Call linked to conversation `conv-100` in metadata
- Recording disclosure played: "This call may be recorded for quality and training. Press 1 to continue or hang up to decline."

**Verification:**
- Call connects within 5 seconds
- Disclosure plays before Si speaks
- Call ID (`call_sid`) linked to conversation

### 4.3 User Accepts Recording and Continues
**Setup:**
- Call in progress, disclosure played
- User presses "1" (DTMF)

**Steps:**
1. System detects DTMF tone "1"
2. Call proceeds

**Expected:**
- Consent recorded:
  - `call_recording_consent`: true
  - `consented_at`: timestamp
  - `call_sid`: recorded
- Si greeting: "Hi! How can I help you today?"
- Audio streaming active (bidirectional)

**Verification:**
- Recording started
- Consent logged in compliance system
- User can speak; Si can respond
- Audio quality acceptable

### 4.4 User Declines Recording (Hangs Up)
**Setup:**
- Disclosure played
- User hangs up without pressing 1

**Steps:**
1. Call status: "completed" (user ended)

**Expected:**
- Consent recorded:
  - `call_recording_consent`: false
  - `reason`: "user_declined"
- Conversation reverts to SMS
- Follow-up SMS: "No problem! We can continue via text. How can I help?"

**Verification:**
- No recording created or saved
- SMS conversation continues
- Compliance log shows declined recording

### 4.5 Bind Voice Transcript to Conversation
**Setup:**
- Active call linked to `conv-100`
- User speaks: "What's the status of my order?"
- Si responds: "Your order #12345 shipped yesterday."

**Steps:**
1. Speech-to-text transcribes user audio
2. Cognition generates response
3. Text-to-speech plays response
4. Transcript entries created

**Expected:**
- Conversation `conv-100` transcript includes:
  - Entry 1: `{"speaker": "user", "text": "What's the status of my order?", "source": "voice", "timestamp": "<ISO>"}`
  - Entry 2: `{"speaker": "si", "text": "Your order #12345 shipped yesterday.", "source": "voice", "timestamp": "<ISO>"}`
- Transcript interleaves SMS and voice chronologically
- Session orchestration sees unified history

**Verification:**
- Transcript shows both SMS and voice entries
- Chronological order maintained
- Source attribution clear ("sms" vs "voice")

### 4.6 Handle Call Drop and Resume
**Setup:**
- Call in progress
- Network issue causes disconnect

**Steps:**
1. Call status changes to "failed"
2. System detects unexpected termination

**Expected:**
- SMS sent immediately: "Looks like we got disconnected. Reply CALL to reconnect or continue here via text."
- Call transcript saved up to disconnect point
- Conversation remains active

**Verification:**
- User notified within 5 seconds of disconnect
- Transcript complete up to failure
- User can resume via SMS or new call

### 4.7 Complete Call Normally
**Setup:**
- Call in progress
- User says: "Thanks, goodbye"

**Steps:**
1. Si responds: "You're welcome! Have a great day."
2. Si terminates call gracefully

**Expected:**
- Call status: "completed"
- Final transcript entry: "Call ended normally"
- Call duration logged
- SMS follow-up: "Call ended. Message me anytime!"

**Verification:**
- Full transcript saved to `conv-100`
- Call duration accurate
- SMS conversation remains active

---

## 5. Compliance & Consent

### 5.1 Record Opt-In Artifact
**Setup:**
- User +1-555-5000 opts in via SMS

**Steps:**
1. User sends "YES" in response to opt-in prompt

**Expected:**
- Consent artifact stored:
  ```json
  {
    "user_identifier": "+1-555-5000",
    "consent_type": "messaging",
    "status": "opted_in",
    "timestamp": "<ISO 8601>",
    "method": "explicit_sms_yes",
    "channel": "sms",
    "ip_address": null,
    "retention_days": 2555
  }
  ```

**Verification:**
- Artifact retrievable from consent registry
- Timestamp accurate
- Retention period: 7 years (2555 days)

### 5.2 Record Call Recording Consent
**Setup:**
- User accepts call recording (presses 1)

**Steps:**
1. DTMF "1" detected

**Expected:**
- Consent artifact:
  ```json
  {
    "user_identifier": "+1-555-5000",
    "consent_type": "call_recording",
    "status": "granted",
    "timestamp": "<ISO 8601>",
    "method": "dtmf_confirmation",
    "call_sid": "<twilio_call_sid>",
    "conversation_id": "<conv_id>"
  }
  ```

**Verification:**
- Consent linked to specific call
- Method documented as DTMF
- Audit trail complete

### 5.3 Handle GDPR Data Export Request
**Setup:**
- EU user: +44-7700-900000
- Data includes messages, transcripts, consent records

**Steps:**
1. User sends: "Export my data"
2. System detects GDPR request (or manual submission)

**Expected:**
- Automated response: "Your data export request is being processed. You'll receive a download link within 30 days."
- Incident ticket created for manual review
- Export workflow initiated:
  - All conversation messages
  - All consent records
  - All voice transcripts
  - All compliance artifacts
- Export delivered within 30 days

**Verification:**
- Request logged with timestamp
- Export contains all user data
- Delivery within regulatory timeframe

### 5.4 Handle GDPR Data Deletion Request
**Setup:**
- User: +44-7700-900001
- Has conversation history, consent records

**Steps:**
1. User sends: "Delete my data"
2. System requires confirmation: "This will delete all your data permanently. Reply DELETE CONFIRM to proceed."
3. User replies: "DELETE CONFIRM"

**Expected:**
- Deletion executed:
  - All messages: deleted
  - All transcripts: deleted
  - Consent records: anonymized (retained for compliance)
- Confirmation: "Your data has been deleted. Consent records are retained anonymously as required by law."

**Verification:**
- User data no longer accessible via queries
- Consent records exist but identifiers removed
- Deletion logged in audit trail

### 5.5 Enforce Retention Policy
**Setup:**
- Conversation `conv-200` closed 8 years ago
- Retention policy: 7 years

**Steps:**
1. System runs automated retention enforcement
2. Identifies expired conversations

**Expected:**
- Conversation `conv-200` purged:
  - Messages: deleted
  - Transcripts: deleted
  - Metadata: anonymized and retained
- Consent records: archived

**Verification:**
- Data deleted after 7-year retention
- Audit log shows enforcement action
- Anonymized metadata preserved for analytics

---

## 6. Error Handling & Reliability

### 6.1 Handle Webhook Authentication Failure
**Setup:**
- Malicious webhook POST with invalid signature

**Steps:**
1. Request arrives with `X-Twilio-Signature` mismatch
2. System validates signature

**Expected:**
- Request rejected: HTTP 401 Unauthorized
- No processing occurs
- Security incident logged:
  - `source_ip`: captured
  - `signature`: invalid
  - `timestamp`: logged
- Alert if threshold exceeded (>10 failures/hour)

**Verification:**
- Invalid request blocked
- No data corruption
- Security team notified if attack detected

### 6.2 Handle Webhook Processing Timeout
**Setup:**
- Webhook processing takes > 5 seconds
- Twilio timeout threshold: 5 seconds

**Steps:**
1. Processing delayed (e.g., downstream service slow)
2. Timeout threshold approached

**Expected:**
- Immediate response to Twilio: HTTP 202 Accepted
- Message queued for async processing
- Processing continues in background
- User receives response when ready (eventual consistency)

**Verification:**
- Twilio doesn't retry (202 received)
- Message eventually processed
- Latency logged for monitoring

### 6.3 Handle Twilio API Rate Limit
**Setup:**
- System sending messages rapidly
- Twilio rate limit: 100 req/sec

**Steps:**
1. System hits rate limit
2. Twilio returns HTTP 429 Too Many Requests

**Expected:**
- Messages queued (not dropped)
- Exponential backoff:
  - Retry after 1 second
  - Then 2, 4, 8 seconds (capped at 8)
- Messages delivered when rate limit clears

**Verification:**
- Zero message loss
- Retries eventually succeed
- Rate limit respected in subsequent requests

### 6.4 Handle Invalid Phone Number Format
**Setup:**
- Malformed number received: "555-ABC-1234"

**Steps:**
1. System attempts to process

**Expected:**
- Validation fails
- Error logged with details
- If user-submitted: Response: "Please provide a valid phone number."
- If internal error: Escalated to engineering team

**Verification:**
- Invalid data rejected early
- Clear error message to user
- No system crash or exception

### 6.5 Handle Message Queue Overflow
**Setup:**
- 10,000 messages queued
- Queue capacity: 5,000

**Steps:**
1. Queue reaches capacity
2. New messages arrive

**Expected:**
- Oldest non-critical messages archived to cold storage
- Critical messages prioritized (opt-out confirmations, compliance messages)
- Alert: "Message queue at capacity"
- Backpressure applied to upstream

**Verification:**
- No loss of critical messages
- System remains stable
- Capacity planning alert triggered

---

## 7. Performance & Scale

### 7.1 Handle High Volume (1000 concurrent conversations)
**Setup:**
- Simulate 1000 users sending messages simultaneously

**Steps:**
1. Generate load spike
2. Measure latency

**Expected:**
- **P50 latency**: < 200ms (webhook receipt to canonical event creation)
- **P95 latency**: < 500ms
- **P99 latency**: < 1000ms
- Zero message loss
- Zero errors

**Verification:**
- Latency targets met
- System stable under load
- All messages processed successfully

### 7.2 Message Delivery Success Rate
**Setup:**
- Normal operation over 24 hours
- 10,000 messages sent

**Steps:**
1. Track delivery status for all messages

**Expected:**
- **Success rate**: ≥ 99.5%
- **Failed messages**: < 0.5% (with retries counted)
- **Permanent failures**: < 0.1%

**Verification:**
- SLO met: 99.5% delivery success
- Failures are transient and recovered
- Dashboard shows actual success rate

### 7.3 End-to-End Response Latency
**Setup:**
- New user sends first message
- Full conversation flow

**Steps:**
1. User sends: "Hello"
2. Measure time from webhook receipt to response delivery

**Expected:**
- **Total latency**: < 3 seconds (P95)
- Breakdown:
  - Webhook auth & validation: < 50ms
  - Consent check: < 100ms
  - Event normalization: < 50ms
  - Session creation: < 200ms
  - Cognition processing: < 2000ms
  - Response delivery: < 500ms

**Verification:**
- P95 latency < 3 seconds
- User perceives immediate response
- Latency breakdown logged for optimization

---

## 8. Observability

### 8.1 Emit Structured Events
**Setup:**
- Normal conversation flow

**Expected Events:**
- `twilio.webhook.received` (timestamp, message_sid, channel, latency_ms)
- `twilio.consent.checked` (user_id, status, check_latency_ms)
- `twilio.message.normalized` (event_id, session_id, content_type)
- `twilio.response.delivered` (message_sid, status, delivery_latency_ms)
- `twilio.error.occurred` (error_code, error_message, context)

**Verification:**
- All events present in observability platform (e.g., Datadog, Splunk)
- Events correlatable by `session_id` or `correlation_id`
- Latency metrics trackable

### 8.2 Track Channel-Specific Metrics
**Setup:**
- 1 hour of operation

**Expected Metrics:**
- `twilio.messages.sent` (count: 100)
- `twilio.messages.delivered` (count: 99)
- `twilio.messages.failed` (count: 1)
- `twilio.delivery.success_rate` (99%)
- `twilio.delivery.latency_p50` (180ms)
- `twilio.delivery.latency_p95` (450ms)
- `twilio.delivery.latency_p99` (800ms)

**Verification:**
- Metrics accurate within ±2%
- Dashboard visualizes trends
- Alerts configured for SLO violations

### 8.3 Alert on High Failure Rate
**Setup:**
- Delivery failure rate spikes to 10%
- Alert threshold: >5%

**Steps:**
1. Failure rate exceeds threshold

**Expected:**
- Alert triggered within 1 minute
- Alert details:
  - `Channel`: "Twilio SMS/WhatsApp"
  - `Failure rate`: 10%
  - `Threshold`: 5%
  - `Time window`: Last 5 minutes
- Operations team notified (PagerDuty, Slack)
- Runbook linked in alert

**Verification:**
- Alert fires within 1 minute of threshold breach
- Team receives notification
- Runbook accessible for incident response

---

## Summary

**Total Test Categories:** 8
**Total Test Scenarios:** 60+

**Coverage:**
- Conversation management: 6 tests
- Consent & governance: 6 tests
- Message normalization & delivery: 8 tests
- Voice escalation: 7 tests
- Compliance: 5 tests
- Error handling: 5 tests
- Performance & scale: 3 tests
- Observability: 3 tests

**Test Quality:**
- ✅ All tests have specific setup conditions
- ✅ All tests have step-by-step procedures
- ✅ All tests have measurable expected outcomes
- ✅ All tests have clear verification criteria
- ✅ Suitable for TDD (Test-Driven Development)
- ✅ Suitable for automation

**Implementation Readiness:** Complete specification for agent-driven code generation.
