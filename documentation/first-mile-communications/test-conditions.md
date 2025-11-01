# Si First-Mile Communications — Test Conditions

## Purpose
Enumerate measurable verification criteria required to prove the first-mile communication layer meets Si’s quality, compliance, and reliability expectations.

## General Readiness Tests
- **Session Lifecycle Validation:** Simulate session creation, reuse, and teardown across all channels; confirm consistent session identifiers and archival state.
- **Canonical Event Integrity:** Inject text, audio, and control events; verify normalization produces expected fields (timestamp precision, participant metadata, consent markers) without loss or duplication.
- **Response Delivery:** For each modality, confirm outbound messages arrive in order, within configured latency budgets, and include correlation keys for traceability.
- **SLO Compliance:** During test cycles, measure latency percentiles and availability windows; compare results against the targets recorded in `service-level-objectives.md` and document any variances.

## Channel Parity Tests
- **Text Parity:** Send identical prompts via Twilio messaging, browser client, and mobile client; validate that Si’s responses are semantically equivalent and context is preserved.
- **Voice Parity:** Conduct live audio sessions through browser and mobile clients; confirm transcripts feed Si consistently and voice responses meet clarity thresholds defined by product.
- **Modality Switching:** Mid-session, switch from text to voice (and voice to text) on each channel; confirm no context loss, session identifier continuity, and graceful handling of queued responses.

## Compliance & Consent Tests
- **Opt-In Enforcement:** Attempt interactions without required opt-in or consent; ensure system halts processing, informs the user of missing prerequisites, and records denial events.
- **Recording Disclosure:** For voice sessions, confirm required notifications are played or displayed before capturing audio; verify acknowledgments are logged.
- **Data Retention Controls:** Trigger retention policies (e.g., data deletion requests, jurisdiction-based retention windows) and confirm transcripts and audio artifacts follow directives.

## Reliability & Resilience Tests
- **Retry & Fallback Behavior:** Force delivery failures (network drops, temporary channel outages); verify retry strategy, fallback messaging, or graceful degradation occurs within configured limits.
- **Throughput & Load:** Run sustained high-volume traffic for each channel, measuring latency, error rates, and resource utilization; confirm adherence to Service Level Objectives.
- **Chaos Scenarios:** Randomly terminate streaming sessions, revoke credentials, or introduce malformed events; ensure system detects anomalies, isolates impact, and maintains audit trail.

## Observability & Diagnostics Tests
- **Tracing Completeness:** Verify that every inbound and outbound event emits structured telemetry with session identifiers, channel metadata, and timing markers.
- **Alerting Accuracy:** Simulate threshold breaches (latency, error rate, consent failure); confirm alerts trigger once, provide actionable context, and resolve when conditions normalize.
- **SLO Dashboards:** Verify observability dashboards surface current latency and availability performance versus targets, including error budget burn-down indicators.
- **Replay Capability:** Select a historical session; reconstruct the entire conversation, including modality switches, using stored artifacts without referencing live systems.

## Accessibility & Inclusivity Tests
- **Localization:** Exercise supported languages and locale formats in text and audio; confirm prompts, disclaimers, and responses respect locale settings.
- **Alternate Input Modes:** Validate assistive technology compatibility (screen readers for web, voice-over features for mobile) and ensure no critical functionality depends on a single mode (e.g., onscreen buttons only).
- **Low-Bandwidth Operation:** Inject degraded network conditions; confirm fallbacks that maintain core communication (text fallback for audio, buffered playback) while capturing telemetry for quality reporting.

## Cross-System Integration Tests
- **Downstream Handoff:** Confirm normalized events are consumed correctly by cognition services, working memory, and analytic pipelines; upstream responses must translate back into channel-friendly formats.
- **Incident Response Workflow:** Trigger conditions that require manual intervention (e.g., repeated delivery failures); ensure escalation to operations channels occurs with necessary details.
- **Version Compatibility:** Validate backward compatibility during staged rollouts by running mixed-version clients and ensuring orchestration handles variations without user-visible issues.
