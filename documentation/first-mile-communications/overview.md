# Si First-Mile Communications — Overview

## Purpose
Define the functionality of Si’s first-mile communication layer that accepts and returns both text and voice interactions across messaging, browser, and mobile clients. This document focuses on what the layer must accomplish and why, leaving implementation technology choices to downstream engineering.

## Role Within Si
- Provides the outermost interface between humans and Si, translating channel-specific events into canonical session activity.
- Guarantees consistent conversational behavior regardless of entry point while exposing capabilities unique to each channel where necessary (e.g., asynchronous messaging vs. live audio).
- Acts as the gatekeeper for identity, consent, and policy enforcement before requests reach core cognition services.

## Supported Channels
1. **Twilio-hosted messaging** — bidirectional SMS and WhatsApp conversations with optional escalation to voice calls.
2. **Browser experience** — web client offering live streaming for speech and rapid-turn text exchange.
3. **Mobile experience** — React Native client mirroring browser functionality with device-native audio capture and playback.

Each channel must:
- Support text-only, voice-only, and blended experiences without loss of context.
- Preserve session continuity when a user switches modalities mid-conversation.
- Surface channel-specific capabilities through consistent abstractions (e.g., “message event,” “audio frame,” “consent artifact”).

## Core Responsibilities
- **Session Establishment:** Issue or retrieve a canonical session identifier, associate it with user-provided metadata, and initialize state in shared services.
- **Message Normalization:** Convert inbound events (text snippets, audio frames, call events) into a unified envelope carrying timestamps, participant identity, channel metadata, and compliance attributes.
- **Response Distribution:** Deliver Si responses to the originating channel, respecting channel limits (message size, rate caps) and matching modality (send audio to voice endpoints, text to messaging).
- **Consent and Compliance:** Verify opt-in requirements, announce recording when voice is active, and capture audit artifacts.
- **Reliability:** Detect delivery failures, attempt retries where allowed, and escalate to incident workflows when thresholds are exceeded.
- **Observability:** Emit structured events for monitoring, analytics, and traceability across the conversation lifecycle.

## Interaction Lifecycle
1. **Discovery and Opt-In:** The user discovers a channel entry point (phone number, link, app) and consents to messaging or audio capture.
2. **Session Binding:** The first-mile layer binds the user to a session, ensuring the same logical thread persists even if the user moves across channels.
3. **Event Exchange:** Requests flow into canonical conversation events; Si produces responses which are transformed and delivered.
4. **Escalation or Transfer:** Users may opt to move from asynchronous text to real-time voice (or vice versa). First-mile services orchestrate the transition without breaking continuity.
5. **Termination and Archival:** Idle or explicit session closures trigger teardown, persistence of transcripts, and revocation of temporary credentials.

## Non-Functional Goals
- **Consistency:** Comparable response times and behavioral guarantees across channels, with documented deviations.
- **Extensibility:** Ability to add future channels (e.g., email, voice assistants) without rewriting core orchestration.
- **Testability:** Deterministic test hooks for text and voice flows, enabling automated regression suites.
- **Technology-Agnosticism:** Specifications describe required behavior; implementation teams choose protocols and tooling that satisfy requirements.

## Success Metrics
- Sessions can be initiated, sustained, and terminated across all channels without manual intervention.
- Latency budgets are met for conversational responsiveness (text turnaround within human perception norms; audio interactions without noticeable delay).
- Compliance and consent events are recorded for 100% of voice interactions and required jurisdictions.
- Error rates for delivery, transcription, or playback remain within Service Level Objectives agreed with reliability engineering.

## Related Documents
- `documentation/first-mile-communications/test-conditions.md` — exhaustive verification criteria.
- `documentation/first-mile-communications/integration.md` — interfaces with shared Si services and downstream systems.
- Channel-specific documentation in `documentation/first-mile-communications/channels/`.
- Shared service descriptions in `documentation/first-mile-communications/shared-services/`.
