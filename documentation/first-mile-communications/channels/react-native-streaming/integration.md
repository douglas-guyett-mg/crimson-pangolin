# React Native Streaming Channel — Integration Guide

## Purpose
Explain how the mobile streaming client coordinates with Si’s backend services and platform features.

## Upstream Dependencies
- **Mobile Authentication Service:** Supplies secure tokens, refresh mechanisms, and policy payloads tailored for mobile usage.
- **Configuration Distribution:** Delivers environment-specific settings, feature flags, and localization bundles at application startup.
- **Consent & Policy Service:** Provides current consent status for audio recording, messaging, and data retention preferences.

## Downstream Interfaces
- **Mobile Streaming Gateway:** Handles bidirectional transfer of text and audio events, providing flow control signals and acknowledgments.
- **Session Orchestration:** Maintains authoritative session state across devices, ensuring mobile interactions stay aligned with other channels.
- **Notification Service:** Dispatches push notifications for new messages, reminders, or session status updates when the app is backgrounded.
- **Telemetry Platform:** Receives anonymized metrics, logs, and crash reports for monitoring and diagnostics.

## Integration Flow
1. **Startup Sequence:** App retrieves configuration, authenticates user or guest, validates consent, and registers device for notifications.
2. **Session Binding:** App requests session linkage or resumption; receives identifiers, policy constraints, and feature flag states.
3. **Streaming Exchange:** Text and audio events flow through the mobile streaming gateway; backpressure signals drive adaptive encoding and buffering choices.
4. **Response Delivery:** Incoming responses update the transcript, trigger optional notifications, and play audio through platform-specific components.
5. **State Persistence:** Key session metadata and transcript summaries store locally (within policy limits) to support offline review.
6. **Shutdown & Cleanup:** On logout or uninstall, app revokes tokens, deregisters notifications, and clears stored data.

## Coordination Requirements
- **API Version Management:** Maintain compatibility between mobile app releases and backend interfaces; publish deprecation timelines and migration notes.
- **Notification Governance:** Align notification payload schemas and rules with customer operations to avoid notification fatigue.
- **Incident Mitigation:** Establish rollback, feature flag, and hotfix procedures to handle mobile-specific regressions without impacting other channels.
