# Service Level Objectives — First-Mile Communications

## Purpose
Document the service-level expectations that guide design, implementation, and validation of Si’s first-mile communication layer. This specification defines latency budgets, availability targets, ownership, and measurement practices for every supported channel.

## Scope
- Applies to Twilio-hosted messaging (SMS and WhatsApp), browser streaming, and React Native streaming experiences.
- Covers end-to-end interactions from the moment a user initiates contact until the response is delivered back through the originating channel.
- Excludes downstream cognition latency targets; those are defined in the reasoning system documentation.

## Terminology
- **Latency Budget:** Maximum acceptable elapsed time (usually at a P95 or P99 percentile) between a user action and Si’s observable response in the same modality.
- **Availability Target:** Percentage of time a channel is fully functional during the agreed evaluation window (typically monthly or quarterly).
- **Error Budget:** Complement of the availability target, representing the allowable downtime or severe degradation within the same window.
- **Measurement Window:** Period over which metrics are calculated (e.g., rolling 30 days, calendar quarter).
- **SLO Owner:** Role accountable for maintaining the target and coordinating incident response when breaches occur.

## Ownership & Governance
- **Primary Owners:** Reliability Engineering lead for first-mile services, partnered with Channel Product owners for Twilio, Web, and Mobile.
- **Review Cadence:** Quarterly SLO review with Reliability Engineering, Platform Engineering, and Product stakeholders; ad-hoc updates triggered by material changes in architecture or user volume.
- **Change Control:** Any adjustment to targets or measurement methodology requires documentation updates plus alignment with QA test suites and alerting rules.

## Measurement Principles
- Capture latency from user entry point to confirmed delivery (including adapter processing, policy checks, and downstream cognition round trip).
- Use percentile-based measurements (P50, P95, P99) to reflect real user experience; track averages only as supporting context.
- Distinguish transient failures (auto-recoverable) from hard outages when calculating availability.
- Maintain channel-specific dashboards with shared schema to enable cross-channel comparisons.
- Reconcile observability data with customer-reported incidents to ensure measurement accuracy.

## Channel SLOs

### Twilio Messaging (SMS & WhatsApp)
- **Latency Budget (initial target):** P95 ≤ 5 seconds from user message receipt at first-mile boundary to Si response delivered back to Twilio.
- **Availability Target:** ≥ 99.5% monthly (error budget ≤ 3.65 hours per month) excluding carrier-wide outages declared by Twilio.
- **Key Supporting Metrics:** Delivery success rate ≥ 99%, opt-in compliance events recorded for 100% of conversations, median retry recovery < 15 seconds.
- **Measurement Notes:** Instrument both inbound webhook processing and outbound delivery confirmations; flag carrier delays separately to protect error budget.

### Browser Streaming
- **Latency Budget (initial target):** P95 ≤ 800 ms for incremental text token display; P95 ≤ 1500 ms for audio playback start after user speech pause.
- **Availability Target:** ≥ 99.7% monthly (error budget ≤ 2.16 hours per month) for core streaming capability; excludes planned maintenance windows < 30 minutes announced 24 hours ahead.
- **Key Supporting Metrics:** Stream drop rate < 1% per session, reconnect success rate ≥ 98%, accessibility feature availability 100%.
- **Measurement Notes:** Combine client telemetry (edge latency, dropped frames) with server-side traces; treat degraded fallback to text-only as partial availability and record in incident summaries.

### React Native Streaming
- **Latency Budget (initial target):** P95 ≤ 1 second for text response rendering; P95 ≤ 2 seconds for audio playback start after user speech pause, accounting for mobile network variability.
- **Availability Target:** ≥ 99.3% monthly (error budget ≤ 5.04 hours per month) for baseline functionality (text plus audio streaming); push notification delivery availability ≥ 99%.
- **Key Supporting Metrics:** Session resume success rate ≥ 97%, offline message sync success ≥ 99%, app crash-free sessions ≥ 99.8%.
- **Measurement Notes:** Segment metrics by platform (iOS, Android) and network type (Wi-Fi, cellular). Treat store-enforced downtime (e.g., forced updates) as availability impact unless explicitly exempted in product policy.

## Alert Thresholds & Incident Response
- Trigger **Warning Alerts** when 25% of the monthly error budget is consumed or when latency exceeds targets for two continuous measurement intervals.
- Trigger **Critical Alerts** when 50% of the error budget is consumed or a single outage exceeds 30 minutes.
- Incident response led by SLO owner; follow Si incident management protocol, including post-incident review and updates to mitigation plans.

## Verification
- QA and SRE teams map automated and manual tests to each latency and availability target; see `documentation/first-mile-communications/test-conditions.md` for coverage.
- Observability dashboards must include up-to-date SLO tracking widgets reviewed in weekly reliability sync.
- Post-release validation checks confirm SLO compliance for new features or configuration changes.

## Open Follow-Ups
- Validate initial targets with Reliability Engineering and update with production baselines after first month of unified telemetry.
- Define explicit exemption policy for third-party outages (carrier disruptions, app store downtime).
- Publish customer-facing SLA commitments once internal SLOs are ratified and stability is proven.
