# Web Streaming Channel — Test Conditions

## Functional Verification
- **Text Input & Display:** Validate that text entries, including multiline and formatted content, appear correctly in the transcript with accurate timestamps and participant labels.
- **Audio Capture & Playback:** Record speech segments across supported browsers and operating systems; confirm captured audio reaches Si and responses play back with intelligible quality.
- **Streaming Continuity:** Measure the delay between user input and visible/ audible responses; ensure P95 latency stays within the budgets defined in `service-level-objectives.md` (≤ 800 ms for text updates, ≤ 1500 ms for audio playback start).

## Modality Transitions
- **Voice-to-Text Switch:** Begin a voice conversation and switch to text mid-turn; verify transcript continuity and correct handling of partially streamed audio.
- **Text-to-Voice Switch:** Start with text messages, then enable audio streaming; confirm microphone permissions flow and Si responses adapt without reset.
- **Concurrent Inputs:** Attempt simultaneous text entry and voice streaming; ensure prioritization rules prevent collisions or duplicated content.

## Accessibility & Localization
- **Assistive Technologies:** Test with screen readers, keyboard navigation, and high-contrast modes; confirm all controls and transcripts remain operable.
- **Localization Coverage:** Switch interface language and locale settings; verify prompts, disclaimers, and timestamps reflect the selected locale.
- **Transcription Alternatives:** For users declining audio capture, ensure text-only workflows remain fully functional with identical session capabilities.

## Resilience & Recovery
- **Network Degradation:** Introduce packet loss, high latency, and temporary disconnections; observe buffer behavior, reconnection attempts, and user notifications.
- **Browser Refresh & Resume:** Refresh mid-session; confirm state restoration, transcript persistence, and reconnection to streaming services.
- **Resource Constraints:** Simulate low CPU/memory conditions; ensure application warns users and gracefully degrades without crashing.
- **Availability Tracking:** Run sustained sessions to confirm streaming uptime meets or exceeds the 99.7% monthly target, documenting any partial degradations or planned maintenance exclusions.

## Security & Privacy
- **Consent Enforcement:** Attempt to start audio streaming without acknowledging consent; confirm capture is blocked and user is prompted appropriately.
- **Data Handling:** Inspect client storage; verify sensitive data is limited to policy-compliant caches and cleared on logout or explicit user request.
- **Session Integrity:** Attempt cross-tab or cross-device session hijacking; ensure tokens, CSRF protections, and session timeouts prevent unauthorized use.

## Observability
- **Client Telemetry:** Confirm the web client emits performance and error metrics tagged with session identifiers for diagnostics.
- **Server Instrumentation:** Verify server-side streaming endpoints log connection lifecycle events and surface anomalies to monitoring dashboards.
- **User Feedback Loop:** Ensure user-visible status indicators (connection, recording, mute) accurately reflect backend state and update promptly.
- **Error Budget Monitoring:** Validate dashboards display error budget consumption for the web channel, enabling proactive mitigation before targets are breached.
