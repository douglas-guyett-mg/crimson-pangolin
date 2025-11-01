# Documentation Plan: Si First-Mile Communications

**Author:** Codex Agent  
**Date:** 2025-10-30  
**Status:** In Progress (v0.2)  
**Source Request:** 2025-10-30 — “Spec out the first mile on how anyone communicates with Si”

## Purpose
Establish a comprehensive documentation blueprint for Si’s inbound and outbound communication layer (“first mile”), covering Twilio Conversations (SMS and WhatsApp), a browser-based streaming web app, and a React Native streaming app. The plan ensures alignment with Si’s core engineering tenants: documentation-as-code, tests-first, modularity, and technology-agnosticism.

## Scope
- **In Scope:** Channel-specific entry points (text + voice), real-time streaming requirements, message/voice normalization, authentication and enrollment flows, resiliency and observability expectations, cross-channel orchestration, testing and verification criteria.
- **Out of Scope:** Downstream cognition/orchestration internals, specific UI designs, vendor-specific deployment scripts, billing/compliance policy deep-dives (unless they impact first-mile behavior).

## Target Audience
- Systems and platform architects defining communication boundaries.
- Backend engineers integrating channel adapters.
- QA and reliability engineers validating multimodal communication paths.
- Product/ops stakeholders verifying coverage across supported channels.

## Existing Materials & Alignment
- Reviewed `agent-plans/si_core_engineering_tennants.md` for guiding principles; no conflicting direction.
- No existing first-mile communication specs detected in `agent-plans/` or `documentation-plans/`. This plan will become the primary reference.
- Coordinate with future documentation under `working_memory_system.md` to ensure consistent terminology around “sessions” and “turns.”

## Reference Research (authoritative sources to cite during documentation)
- **Twilio Conversations API Reference** — REST resources for Conversations, Participants, Messages, and Events.
- **Twilio Conversations Webhooks & Callbacks** — inbound/outbound message lifecycle, delivery receipts, error events.
- **Twilio Media Streams & Voice Webhooks** — PSTN/SIP voice capture, bidirectional streaming (for voice transcripts + audio handoff).
- **W3C WebRTC & MediaStream Recording APIs** — browser-based low-latency audio streaming and fallback capture.
- **Web Transport (HTTP/3) & WebSocket RFCs** — multiplexed, reliable data channels for text streaming.
- **React Native Audio / WebRTC libraries (e.g., react-native-webrtc, Expo AV)** — mobile capture/playback patterns.
- **Accessibility & localization guidelines (WCAG 2.2, i18n best practices)** — ensure channel parity and Inclusivity.

## Documentation Deliverables
Create a modular documentation tree under `documentation/first-mile-communications/`:

```
documentation/
  first-mile-communications/
    overview.md
    test-conditions.md
    integration.md
    channels/
      twilio-conversations/
        overview.md
        test-conditions.md
        integration.md
      web-streaming/
        overview.md
        test-conditions.md
        integration.md
      react-native-streaming/
        overview.md
        test-conditions.md
        integration.md
    shared-services/
      session-orchestration/
        overview.md
        test-conditions.md
      media-pipeline/
        overview.md
        test-conditions.md
      observability/
        overview.md
        test-conditions.md
```

- [x] `documentation/first-mile-communications/overview.md`
- [x] `documentation/first-mile-communications/test-conditions.md`
- [x] `documentation/first-mile-communications/integration.md`
- [x] Channel documentation for Twilio Conversations (overview, test conditions, integration)
- [x] Channel documentation for web streaming (overview, test conditions, integration)
- [x] Channel documentation for React Native streaming (overview, test conditions, integration)
- [x] Shared services documentation (session orchestration, media pipeline, observability — overview and test conditions)
- [x] Service-level objectives specification capturing latency budgets, availability targets, and ownership
- Each `overview.md` captures “what/why,” key boundaries, and user journeys.
- Each `test-conditions.md` enumerates deterministic and edge-case verification criteria (latency ceilings, failover, geo scenarios, accessibility).
- `integration.md` files document cross-channel choreography, normalization rules, rollout sequencing, and dependencies on other Si systems.

## Planned Documentation Outline
1. **Executive Overview:** Context, goals, supported channels, success metrics, relationship to Si orchestrator.
2. **Channel Capability Profiles:** Detailed requirements for Twilio Conversations, web streaming, and React Native streaming (text + voice), including state diagrams and lifecycle tables.
3. **Shared Services:** Session management, identity binding, media format normalization, transcript routing, consent & opt-in management.
4. **Reliability & Observability:** Metrics, alerting thresholds, replay/diagnostic tooling.
5. **Security & Compliance Considerations:** Data residency, message retention, privacy impact, rate limiting.
6. **Testing Strategy:** Unit, simulation, load, chaos, accessibility, and disaster recovery requirements mapped per component.
7. **Integration Pathways:** How first-mile components feed Frontal Cortex / Working Memory and vice versa; rollout plan across environments.

## Dependencies & Cross-Doc Links
- **Frontal Cortex / Turn Trace Specifications:** Needed for consistent session identifiers and event logging.
- **Authentication & Identity Docs (future)** — placeholder dependency for user account binding.
- **Working Memory Specifications:** For transcript handoff formats and retention policies.
- **Tooling/Observability Plans:** For metrics ingestion and alerting standards.

## Research Questions to Resolve During Documentation
1. Document functional streaming requirements without committing to specific transport implementations; ensure technology selections are captured in a separate architecture artifact if needed.
2. Validate Twilio account capabilities: Conversations + Media Streams concurrency limits, regional considerations.
3. Clarify authentication & consent flow for WhatsApp opt-in and voice call recording.
4. Determine SLA targets (latency, availability) per channel in coordination with SRE owners.

## Test Coverage Focus Areas
- Channel setup/teardown (happy path + failure injection).
- Text/voice echo tests across all endpoints (including Twilio sandbox numbers).
- Media quality benchmarks (packet loss, jitter, codec negotiation).
- Multi-channel session continuity (user switches from SMS to voice mid-session).
- Data integrity and compliance checks (PII redaction, retention windows).
- Observability smoke tests (alerts for dropped connections, webhook failures).

## Execution Plan
1. Inventory integration stakeholders (Twilio platform team, web app, mobile team, SRE, compliance) and capture baseline requirements.
2. Draft `overview.md` and shared services specs first to anchor language and definitions.
3. Iterate channel-specific folders, ensuring parity between Twilio, web, and mobile entries. **(Completed)**
4. Validate test conditions with QA/SRE; refine based on measurable thresholds.
5. Finalize integration narratives and schedule critical design review before moving to implementation docs. **(Completed for first-mile scope; review scheduling pending stakeholder confirmation)**

## Success Criteria
- Documentation enables an independent team to implement all three channels without additional clarifications.
- Test conditions cover ≥95% of identified user journeys and failure modes.
- Terminology, identifiers, and data contracts align with existing Si specs.
- Plan accepted by stakeholders (communication leads, architecture, QA) prior to implementation phase.

## Open Issues & Follow-Up
- Await confirmation on telemetry stack (e.g., OpenTelemetry vs. vendor tooling) for observability sections.
- Need decision on unified vs. channel-specific authentication broker.
- Pending discussion on minimum device/browser support matrices for streaming experiences.
