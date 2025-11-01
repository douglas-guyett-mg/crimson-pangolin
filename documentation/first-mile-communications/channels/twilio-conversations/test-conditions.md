# Twilio Conversations Channel — Test Conditions

## Messaging Flow Validation
- **Conversation Creation:** Initiate messages from new SMS and WhatsApp numbers; confirm conversations are created, participants registered, and canonical session identifiers assigned.
- **Message Round Trip:** Send text messages of varying lengths and character sets; verify Si responses match expected content, respect channel length limits, and include receipt confirmations.
- **Latency Budget Check:** Measure end-to-end response latency at P95; confirm it remains at or below the five-second budget defined in `service-level-objectives.md`.
- **Media Attachment Handling:** Transmit supported media types; ensure references are captured, stored, and acknowledged even when Si responds with text-only messages.

## Compliance & Policy Tests
- **Opt-In/Opt-Out Enforcement:** Test keyword-based enrollment and cessation flows; validate that opt-out blocks further messaging until explicit opt-in is recorded.
- **Consent Logging:** For regions requiring consent statements, confirm that required messages are sent, acknowledged, and logged before subsequent conversation.
- **Retention Controls:** Execute data removal requests; verify conversation transcripts and associated metadata follow mandated retention or deletion timelines.

## Voice Escalation Scenarios
- **Call Initiation from Text:** Request a voice session from an active conversation; ensure call state references the originating conversation and transcripts capture both modalities.
- **Fallback Handling:** Decline voice consent and confirm system offers text alternatives without degrading session continuity.
- **Call Drop Recovery:** Simulate mid-call disconnections; verify that reconnection attempts preserve context and update transcripts appropriately.

## Reliability & Error Handling
- **Rate Limit Behavior:** Exceed channel throughput limits; confirm messages queue, throttle, or notify users according to policy while preserving order.
- **Delivery Failure Alerts:** Force delivery errors (invalid numbers, unreachable recipients); ensure alerts trigger and retries or escalation protocols execute as defined.
- **Availability Monitoring:** Run sustained messaging tests and voice escalations; ensure observed uptime meets or exceeds the 99.5% monthly target, accounting for documented third-party outages.
- **Webhook Integrity:** Tamper with inbound callbacks; validate authentication, signature verification, and rejection of malformed events.

## Observability
- **Conversation Telemetry:** Check that every message and call event emits structured logs with conversation IDs, participant identifiers, and timing metrics.
- **Analytics Alignment:** Confirm analytics pipeline receives accurate counts for engagement, opt-outs, and escalation usage.
- **Audit Trail:** Generate compliance reports; verify they contain complete records of consent, conversation history, and voice disclosures for selected sessions.
- **Error Budget Tracking:** Validate telemetry captures error budget consumption for this channel and feeds reliability dashboards with up-to-date figures.
