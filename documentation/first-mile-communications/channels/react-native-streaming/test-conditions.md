# React Native Streaming Channel — Test Conditions

## Functional Coverage
- **Authentication Flows:** Validate sign-in, token refresh, and guest access on both iOS and Android; ensure tokens expire and renew according to policy.
- **Text Messaging:** Test rapid exchanges, long-form messages, and attachments; confirm transcript synchronization across devices and offline persistence rules.
- **Audio Streaming:** Measure capture and playback performance across device models; ensure audio segments submit in order, responses remain intelligible, and P95 latency stays within the budgets recorded in `service-level-objectives.md` (≤ 1 second for text rendering, ≤ 2 seconds for audio playback onset).

## Modality & State Transitions
- **Foreground ↔ Background:** Move the app between foreground, background, and terminated states during active conversations; confirm notifications, reconnection, and transcript continuity.
- **Voice ↔ Text Switching:** Switch modalities while driving audio streaming; verify queue handling and transcript accuracy.
- **Device Handover:** Start a session on one device, continue on another using the same account, and validate session continuity and state synchronization.

## Network Resilience
- **Variable Connectivity:** Simulate cellular handoffs, low bandwidth, and intermittent outages; confirm adaptive quality adjustments, buffering, and fallback messaging.
- **Offline Mode:** Force offline status, queue outbound messages, then restore connectivity; ensure synchronization, ordering, and duplicate prevention.
- **Error Feedback:** Trigger server timeouts or authorization failures; confirm user-facing messages guide remediation steps and logs capture context.

## Platform Compliance
- **Permissions Handling:** Deny microphone or notification permissions; verify graceful degradation and clear guidance for enabling required capabilities.
- **Data Storage Policies:** Inspect local storage; ensure sensitive data is encrypted (where platform allows) and cleared on logout or by user request.
- **Battery & Performance:** Run extended sessions; monitor resource consumption and confirm no abnormal battery drain or performance degradation.
- **Availability Check:** Conduct long-running sessions to verify mobile availability meets the 99.3% monthly target and that push notifications achieve the 99% delivery objective.

## Accessibility & Localization
- **Assistive Technology:** Evaluate VoiceOver (iOS) and TalkBack (Android) interactions; confirm all controls and transcripts are discoverable and actionable.
- **Dynamic Type & Layout:** Adjust font sizes and accessibility settings; ensure layouts adapt without clipping or overlap.
- **Localization:** Test supported languages, including right-to-left scripts; verify interface elements, prompts, and timestamps adapt appropriately.

## Observability
- **Client Logging:** Confirm the app captures diagnostic logs with session identifiers and uploads them according to policy.
- **Metrics Emission:** Ensure latency, error, and usage metrics feed the observability platform for real-time monitoring.
- **Crash Reporting:** Trigger controlled crashes; verify reports include sufficient context for triage without leaking sensitive user content.
- **Error Budget Visibility:** Confirm telemetry dashboards expose error budget consumption for the mobile channel, enabling early detection of sustained degradations.
