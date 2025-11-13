# Web Streaming Channel — Test Conditions

## Functional Verification
- **Text Input & Display:** Validate that text entries, including multiline and formatted content, appear correctly in the transcript with accurate timestamps and participant labels.
- **Audio Capture & Playback:** Record speech segments across supported browsers and operating systems; confirm captured audio reaches Si and responses play back with audio quality score >= 3.5 MOS (Mean Opinion Score) or THD (Total Harmonic Distortion) < 1%, sample rate 16kHz minimum, bit depth 16-bit minimum.
- **Streaming Continuity:** Measure the delay between user input and visible/ audible responses; ensure P95 latency stays within the budgets defined in `service-level-objectives.md` (≤ 800 ms for text updates, ≤ 1500 ms for audio playback start).

## Modality Transitions
- **Voice-to-Text Switch:** Begin a voice conversation and switch to text mid-turn; verify transcript continuity and correct handling of partially streamed audio.
- **Text-to-Voice Switch:** Start with text messages, then enable audio streaming; confirm microphone permissions flow and Si responses adapt without reset.
- **Concurrent Inputs:** Attempt simultaneous text entry and voice streaming; ensure prioritization rules prevent collisions or duplicated content.

## Accessibility & Localization
- **Assistive Technologies:** Test with screen readers, keyboard navigation, and high-contrast modes; confirm 100% of controls accessible via keyboard-only navigation (Tab, Enter, Space, Arrow keys), all interactive elements have ARIA labels, screen reader announces all state changes within 500ms, WCAG 2.1 Level AA compliance verified.
- **Localization Coverage:** Switch interface language and locale settings; verify prompts, disclaimers, and timestamps reflect the selected locale.
- **Transcription Alternatives:** For users declining audio capture, ensure text-only workflows remain fully functional with identical session capabilities.

## Resilience & Recovery
- **Network Degradation:** Introduce packet loss (10%), high latency (500ms+), and temporary disconnections (5-30 seconds); verify system reduces video bitrate to 480p, buffers 5 seconds of audio, displays "Poor Connection" indicator within 2 seconds, maintains 95% message delivery, auto-reconnects within 10 seconds when connection restored.
- **Browser Refresh & Resume:** Refresh mid-session; confirm state restoration, transcript persistence, and reconnection to streaming services.
- **Resource Constraints:** Simulate low CPU (throttled to 25%) and low memory (< 100MB available) conditions; verify application displays warning banner within 2 seconds when CPU usage > 80% for 10 consecutive seconds, message reads "Performance may be affected. Close other applications.", disables video preview if memory < 50MB, maintains audio streaming, does not crash or freeze for 5 minutes.
- **Availability Tracking:** Run sustained sessions to confirm streaming uptime meets or exceeds the 99.7% monthly target, documenting any partial degradations or planned maintenance exclusions.

## Security & Privacy
- **Consent Enforcement:** Attempt to start audio streaming without acknowledging consent; confirm capture is blocked, microphone access denied at browser API level, user sees modal dialog with text "Microphone access required. Please grant permission in browser settings.", dialog includes "Grant Permission" and "Use Text Only" buttons.
- **Data Handling:** Inspect client storage; verify sensitive data is limited to policy-compliant caches and cleared on logout or explicit user request.
- **Session Integrity:** Attempt cross-tab or cross-device session hijacking; ensure tokens, CSRF protections, and session timeouts prevent unauthorized use.

## Observability
- **Client Telemetry:** Confirm the web client emits performance and error metrics tagged with session identifiers for diagnostics.
- **Server Instrumentation:** Verify server-side streaming endpoints log connection lifecycle events and surface anomalies to monitoring dashboards.
- **User Feedback Loop:** Ensure user-visible status indicators (connection, recording, mute) match backend state 100% when queried via API, indicators update within 200ms of backend state change, indicators use standard colors (green=connected/active, red=disconnected/error, yellow=degraded), include text labels for accessibility.
- **Error Budget Monitoring:** Validate dashboards display error budget consumption for the web channel, enabling proactive mitigation before targets are breached.
