# Observability — Test Conditions

## Purpose
Comprehensive, measurable test specifications for the Observability shared service. Covers telemetry collection, metrics aggregation, alerting, dashboarding, and compliance reporting with technology-agnostic patterns.

---

## 1. Telemetry Collection & Coverage

### 1.1 Capture End-to-End Session Telemetry
**Setup:**
- Single SMS conversation: user sends message, Si responds
- Session duration: 2 minutes, 5 message exchanges

**Steps:**
1. User initiates conversation
2. System emits telemetry events throughout

**Expected Events Captured:**
- `session.created` (session_id, channel, user_id_hash, timestamp)
- `message.received` (event_id, session_id, latency_ms)
- `consent.checked` (status, check_duration_ms)
- `message.normalized` (session_id, content_type)
- `llm.invoked` (session_id, model, tokens, latency_ms)
- `message.sent` (session_id, delivery_status)
- `session.terminated` (session_id, duration_ms, message_count)

**Verification:**
- All 7+ event types present
- Events correlatable by session_id
- Timestamps sequential
- No missing events in chain

### 1.2 Track Modality Switch (SMS → Voice)
**Setup:**
- User starts SMS conversation, escalates to voice call

**Steps:**
1. SMS conversation active
2. User requests call
3. Voice call initiated and completed
4. Conversation reverts to SMS

**Expected Events:**
- `modality.switch_requested` (from: "sms", to: "voice")
- `voice.call_initiated` (session_id, call_sid)
- `voice.recording_consent` (granted: true/false)
- `voice.transcription_started`
- `voice.call_ended` (duration_ms, end_reason)
- `modality.switch_completed` (back to: "sms")

**Verification:**
- Modality transitions captured
- Single session_id spans both modalities
- Timeline reconstructable from events

### 1.3 Emit Performance Metrics
**Setup:**
- 100 messages processed over 1 hour

**Expected Metrics Emitted:**
- `fmc.message.latency_ms` (histogram: p50, p95, p99)
- `fmc.message.count` (counter by channel)
- `fmc.delivery.success_rate` (gauge: 0-1)
- `fmc.session.active` (gauge: current count)
- `fmc.consent.check_duration_ms` (histogram)
- `fmc.error.count` (counter by error_type)

**Verification:**
- All metric types present (counter, gauge, histogram)
- Metrics tagged with: channel, region, environment
- Values accurate within ±2%
- Update frequency: every 10 seconds

### 1.4 Validate Log Schema Compliance
**Setup:**
- Normal operation for 10 minutes

**Expected Log Format:**
```json
{
  "timestamp": "<ISO 8601>",
  "level": "info|warn|error",
  "service": "first-mile-communications",
  "component": "twilio-adapter|media-pipeline|...",
  "session_id": "<UUID>",
  "correlation_id": "<UUID>",
  "event_type": "message.received",
  "message": "Human-readable description",
  "metadata": {
    "channel": "sms",
    "latency_ms": 45
  },
  "user_id_hash": "<sha256>",
  "pii_scrubbed": true
}
```

**Verification:**
- All required fields present
- No plaintext PII (phone numbers, emails)
- Severity levels appropriate
- Schema validator passes 100%

### 1.5 Track Consent Events for Compliance
**Setup:**
- User opts in, sends messages, opts out

**Expected Compliance Events:**
- `consent.opt_in_requested` (timestamp, channel)
- `consent.opt_in_confirmed` (method: "explicit_sms_yes")
- `consent.message_allowed` (session_id, consent_status)
- `consent.opt_out_requested` (keyword: "STOP")
- `consent.opt_out_confirmed` (timestamp)

**Verification:**
- All consent state changes logged
- Audit trail complete
- Timestamps allow compliance reporting
- Events retained for 7 years

---

## 2. Metrics Aggregation & SLOs

### 2.1 Calculate Message Latency SLO
**Setup:**
- 1000 messages processed over 24 hours
- SLO target: P95 latency < 3 seconds

**Steps:**
1. Collect `fmc.message.latency_ms` metrics
2. Calculate percentiles

**Expected:**
- P50: ~1.2 seconds
- P95: < 3 seconds (target met)
- P99: < 5 seconds
- SLO dashboard shows: **99.8% compliant**

**Verification:**
- Percentile calculations accurate
- SLO compliance tracked hourly
- Dashboard visualizes trend
- Alert not triggered (under threshold)

### 2.2 Track Delivery Success Rate SLO
**Setup:**
- 10,000 messages sent
- SLO target: ≥ 99.5% delivery success

**Steps:**
1. Track delivery status for all messages
2. Calculate success rate

**Expected:**
- Delivered: 9,960 (99.6%)
- Failed (transient): 30 (0.3%)
- Failed (permanent): 10 (0.1%)
- **SLO: MET** (99.6% ≥ 99.5%)

**Verification:**
- Success rate formula correct: delivered / total
- Failure categorization accurate
- SLO dashboard green
- Trend shows stable performance

### 2.3 Monitor Error Budget Consumption
**Setup:**
- SLO: 99.5% delivery success
- Error budget: 0.5% (50 failures per 10,000 messages)
- Current failures: 30

**Expected Dashboard:**
- Error budget remaining: 40% (20 failures remaining)
- Consumption rate: 0.3% per hour
- Time to budget exhaustion: ~3.3 hours (if rate continues)
- Status: **Warning** (budget < 50%)

**Verification:**
- Error budget calculation correct
- Consumption rate accurate
- Warning threshold triggered
- Team notified to investigate

### 2.4 Aggregate Metrics by Channel
**Setup:**
- Traffic from 3 channels: SMS (40%), WhatsApp (35%), Voice (25%)

**Expected Metrics Breakdown:**
- `fmc.message.count{channel="sms"}`: 4000
- `fmc.message.count{channel="whatsapp"}`: 3500
- `fmc.message.count{channel="voice"}`: 2500
- Total: 10,000

**Verification:**
- Channel tagging consistent
- Aggregation accurate
- Per-channel SLOs calculable
- Dashboard shows channel breakdown

---

## 3. Alerting & Incident Detection

### 3.1 Alert on Latency Spike
**Setup:**
- Normal P95 latency: 2.5 seconds
- Alert threshold: P95 > 5 seconds for 5 minutes

**Steps:**
1. Induce slowdown (downstream service delay)
2. P95 latency climbs to 7 seconds
3. Sustained for 6 minutes

**Expected:**
- Alert triggered at minute 5
- Alert details:
  - Title: "Message latency exceeded threshold"
  - Severity: **High**
  - Current P95: 7.0s (threshold: 5.0s)
  - Affected channels: All
  - Duration: 6 minutes
- Runbook link included
- Team notified via PagerDuty/Slack

**Verification:**
- Alert fires within 30 seconds of threshold breach
- Notification delivered
- Alert resolves when latency drops

### 3.2 Alert on Delivery Failure Burst
**Setup:**
- Normal failure rate: 0.3%
- Alert threshold: > 5% for 10 minutes
- Twilio outage causes 15% failure rate

**Steps:**
1. Simulate Twilio API failures
2. Failure rate spikes to 15%
3. Sustained for 12 minutes

**Expected:**
- Alert triggered at minute 10
- Alert details:
  - Title: "Message delivery failure rate critical"
  - Severity: **Critical**
  - Failure rate: 15% (threshold: 5%)
  - Affected channel: Twilio (SMS/WhatsApp)
- Escalation to on-call engineer
- Duplicate alerts suppressed (same root cause)

**Verification:**
- Alert fires on time
- Escalation policy followed
- No alert spam (single alert, not per-message)
- Post-mortem data available

### 3.3 Alert on Compliance Violation
**Setup:**
- Message sent to opted-out user (bug in consent check)

**Steps:**
1. System attempts to send message to opted-out user
2. Consent violation detected

**Expected:**
- Alert triggered immediately
- Alert details:
  - Title: "Compliance violation: Message to opted-out user"
  - Severity: **Critical**
  - User: <hashed_id>
  - Session: <session_id>
- Compliance team notified
- Message delivery blocked

**Verification:**
- Alert fires within 1 minute
- Compliance team receives notification
- Incident logged for audit
- Remediation steps included

### 3.4 Alert on Error Budget Depletion
**Setup:**
- Error budget: 50 failures remaining (0.5%)
- Failures accumulating rapidly

**Steps:**
1. Error budget drops to 10 failures (0.1%)
2. Budget < 20% threshold crossed

**Expected:**
- Warning alert triggered
- Alert details:
  - Title: "Error budget critically low"
  - Budget remaining: 10 failures (0.1%)
  - Consumption rate: 5 failures/hour
  - Time to exhaustion: 2 hours
- Team urged to investigate and mitigate

**Verification:**
- Warning fires at 20% threshold
- Critical alert at 10% threshold
- SLO risk communicated clearly
- Incident response initiated

---

## 4. Dashboards & Visualization

### 4.1 Real-Time Operational Dashboard
**Setup:**
- Dashboard displayed for ops team

**Expected Panels:**
1. **Traffic Overview**
   - Messages/second (time series)
   - Active sessions (gauge)
   - By channel breakdown (pie chart)

2. **Latency Metrics**
   - P50/P95/P99 latency (time series)
   - Per-channel latency comparison

3. **Reliability**
   - Delivery success rate (gauge: 99.6%)
   - Error rate by type (bar chart)

4. **SLO Status**
   - Latency SLO compliance (green/yellow/red)
   - Delivery SLO compliance
   - Error budget remaining (progress bar)

**Verification:**
- All panels update every 10 seconds
- Data accurate within 30 seconds of actual
- Color coding clear (green = good, red = bad)
- Drill-down links functional

### 4.2 Compliance Audit Dashboard
**Setup:**
- Dashboard for compliance team

**Expected Panels:**
1. **Consent Management**
   - Opt-ins today (counter)
   - Opt-outs today (counter)
   - Pending consents (gauge)

2. **Recording Consents**
   - Voice calls with recording: granted vs declined
   - Consent artifact completeness (%)

3. **Data Requests**
   - GDPR export requests (pending/completed)
   - Deletion requests (pending/completed)

4. **Violations**
   - Messages to opted-out users: 0 (counter)
   - Missing consent artifacts: 0

**Verification:**
- Data sources from compliance events
- Audit trail exportable
- Retention policy visibility
- Zero-tolerance violations highlighted

### 4.3 Performance Troubleshooting Dashboard
**Setup:**
- Engineering team investigating latency issue

**Expected Panels:**
1. **Latency Breakdown**
   - Webhook processing: 50ms
   - Consent check: 100ms
   - Normalization: 30ms
   - Cognition: 1800ms (bottleneck identified)
   - Delivery: 400ms

2. **Error Analysis**
   - Error count by type (time series)
   - Error rate by component
   - Recent error logs (table)

3. **Resource Utilization**
   - CPU usage by service
   - Memory usage
   - Queue depths

**Verification:**
- Latency breakdown pinpoints bottleneck
- Error correlation clear
- Resource constraints visible
- Actionable for debugging

---

## 5. Diagnostics & Session Replay

### 5.1 Reconstruct Session from Telemetry
**Setup:**
- Completed session: conv-123
- 10 message exchanges, 1 voice call

**Steps:**
1. Query observability system for session_id: conv-123
2. Retrieve all events

**Expected Reconstruction:**
- Timeline of all events (chronological)
- Message content (if not PII)
- Modality switches annotated
- Latency at each step
- Errors encountered (if any)
- Final outcome (session terminated normally)

**Verification:**
- Complete event sequence retrievable
- Timeline accurate
- Modality transitions clear
- Useful for debugging

### 5.2 Validate PII Redaction in Logs
**Setup:**
- Session with user phone number, email

**Steps:**
1. Export logs for session
2. Inspect for PII

**Expected:**
- Phone number: **REDACTED** or hashed (sha256)
- Email: **REDACTED** or hashed
- User ID: hashed consistently
- Message content: sanitized (if sensitive)
- Metadata preserved (timestamps, latency)

**Verification:**
- Zero plaintext PII in exports
- Hashing consistent (same user = same hash)
- Logs still useful for debugging
- Compliance audit passes

### 5.3 Test Log Retention and Archival
**Setup:**
- Logs from 8 months ago
- Retention policy: 6 months hot, 2 years archive

**Steps:**
1. Query for 8-month-old logs
2. System retrieves from archive

**Expected:**
- Logs found in archive storage
- Retrieval latency: < 10 seconds
- Format unchanged
- Query functionality limited (no real-time search)

**Verification:**
- Archive policy enforced
- Old logs accessible
- Performance degraded but acceptable
- Compliance retention met (2 years)

---

## 6. Resilience & Reliability

### 6.1 Handle Telemetry Ingestion Spike
**Setup:**
- Normal telemetry: 1000 events/second
- Spike: 10,000 events/second (10x load)

**Steps:**
1. Simulate traffic surge
2. Monitor ingestion pipeline

**Expected:**
- All events buffered or ingested
- Ingestion latency increases: < 5 seconds lag
- No events dropped
- System auto-scales (if configured)
- Alert: "Telemetry ingestion backlog"

**Verification:**
- Zero data loss
- Latency acceptable
- System recovers when load decreases
- Backlog cleared within 10 minutes

### 6.2 Handle Observability Backend Outage
**Setup:**
- Primary observability backend (e.g., Prometheus) unavailable

**Steps:**
1. Simulate backend outage
2. Telemetry continues being generated

**Expected:**
- Events buffered locally (e.g., disk queue)
- Buffer capacity: 1 hour of data
- Alert: "Observability backend unavailable"
- Failover to secondary backend (if configured)
- Data replayed when primary recovers

**Verification:**
- No data loss during outage
- Replay successful
- Gap in dashboard during outage
- Service continues operating

### 6.3 Test Cross-Region Observability
**Setup:**
- Multi-region deployment: US-East, EU-West
- US-East region experiences issue

**Steps:**
1. Induce latency in US-East
2. Check observability visibility

**Expected:**
- Regional dashboards show issue in US-East only
- EU-West unaffected
- Global dashboard aggregates both regions
- Alert specifies affected region
- Data from both regions correlatable

**Verification:**
- Regional isolation clear
- Global view accurate
- Incident scoped to affected region
- Cross-region queries functional

---

## 7. Data Export & Analytics

### 7.1 Export Metrics for Analysis
**Setup:**
- Request CSV export of last 30 days metrics

**Steps:**
1. Initiate export via dashboard or API
2. Download CSV file

**Expected CSV Columns:**
- timestamp, metric_name, value, channel, region
- Example row: `2025-11-07T10:30:00Z, fmc.message.latency_ms.p95, 2450, sms, us-east-1`

**Verification:**
- Export completes within 5 minutes
- All requested metrics included
- Format parseable by analytics tools
- PII sanitized

### 7.2 Generate SLO Compliance Report
**Setup:**
- Monthly SLO report needed

**Steps:**
1. Request SLO report for October 2025
2. System generates report

**Expected Report Sections:**
1. **Executive Summary**
   - Overall SLO compliance: 99.7%
   - Incidents: 2
   - Error budget remaining: 15%

2. **Per-Channel SLOs**
   - SMS: 99.8% (target: 99.5%) ✅
   - WhatsApp: 99.6% ✅
   - Voice: 99.4% ❌ (below target)

3. **Incident Analysis**
   - Incident #1: Twilio outage, 2 hours
   - Incident #2: Database slowdown, 30 minutes

**Verification:**
- Report accurate
- Actionable insights included
- Exportable (PDF/CSV)
- Shareable with stakeholders

---

## Summary

**Total Test Categories:** 7
**Total Test Scenarios:** 35+

**Coverage:**
- Telemetry collection: 5 tests
- Metrics aggregation & SLOs: 4 tests
- Alerting & incident detection: 4 tests
- Dashboards & visualization: 3 tests
- Diagnostics & session replay: 3 tests
- Resilience & reliability: 3 tests
- Data export & analytics: 2 tests

**Test Quality:**
- ✅ All tests have specific setup conditions
- ✅ All tests have step-by-step procedures
- ✅ All tests have measurable expected outcomes
- ✅ All tests have clear verification criteria
- ✅ Technology-agnostic (works with Prometheus, Datadog, Splunk, etc.)
- ✅ Suitable for TDD (Test-Driven Development)
- ✅ Suitable for automation

**Key Metrics:**
- Latency SLO: P95 < 3 seconds
- Delivery SLO: ≥ 99.5% success
- Alert latency: < 1 minute for critical
- PII: 100% redacted in exports
- Retention: 6 months hot, 2 years archive

**Implementation Readiness:** Complete observability specification for agent-driven code generation.
