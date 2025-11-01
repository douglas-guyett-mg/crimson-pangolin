# Si First-Mile Communications — Integration Guide

## Purpose
Describe how the first-mile communication layer interacts with other Si systems, highlighting required contracts, sequencing, and shared responsibilities.

## Core Interfaces
- **Identity & Access Services:** Consume user and organization records to validate enrollment, permissions, and consent. Provide hooks for verifying multifactor signals when policy requires elevated assurance.
- **Session Orchestration:** Coordinate with shared services to register sessions, persist context keys, and track modality state. Must support resume tokens allowing users to rejoin ongoing conversations.
- **Cognition & Reasoning Layer:** Deliver normalized requests to Si’s reasoning gateway and receive synthesized responses. Maintain back-pressure signals so front-end channels can throttle user-facing delivery when necessary.
- **Working Memory & Transcript Storage:** Write canonical transcripts (text + audio references) and metadata for retrieval, analytics, and future context injection.
- **Observability Platform:** Publish structured telemetry (events, metrics, traces) following Si’s instrumentation schema to enable cross-system diagnostics.

## Message Flow Expectations
1. **Inbound Event Reception:** Channel-specific adapters capture raw events and convert them into canonical conversation envelopes.
2. **Policy Enforcement:** Envelopes pass through consent, rate limiting, and security filters before reaching cognition services.
3. **Response Preparation:** Responses from cognition are enriched with channel directives (formatting, segmentation, audio markers) while preserving core message content.
4. **Delivery & Confirmation:** Outbound delivery occurs through channel adapters, which emit delivery confirmations or failure notices back into the shared telemetry stream.
5. **Archival & Analytics:** Completed exchanges are stored in working memory and analytics pipelines to support future context, reporting, and compliance audits.

## Dependency Contracts
- **Schema Evolution:** Any change to canonical envelope schemas or session metadata requires coordination with downstream consumers to maintain compatibility.
- **Latency Budgets:** Shared understanding of latency targets per stage (adapter processing, policy checks, cognition round trip) to meet end-to-end conversational expectations.
- **Error Handling:** Standardized error codes and remediation instructions must be propagated so that user-facing clients can present actionable feedback.
- **Feature Flags:** First-mile deployments respect Si’s feature flag framework, enabling progressive rollout by channel, region, or user cohort.

## Environment Alignment
- **Development & Staging:** Provide sandbox credentials or simulated channel endpoints to validate functionality without impacting production traffic.
- **Production:** Employ hardened configurations, compliance logging, and cross-region redundancy. Ensure observability integrations point to production-grade telemetry backends.
- **Disaster Recovery:** Maintain documented reconstitution steps for channel credentials, routing tables, and session state in the event of regional outages.

## Collaboration Touchpoints
- **Platform Engineering:** Coordinates on shared service availability, scaling, and infrastructure readiness.
- **Security & Compliance:** Reviews consent workflows, data handling, and audit trails before major releases.
- **Customer Operations:** Informed of channel changes, onboarding processes, and escalation workflows to support end users.
- **Quality Engineering:** Co-owns automated test suites and manual validation plans outlined in `test-conditions.md`.
