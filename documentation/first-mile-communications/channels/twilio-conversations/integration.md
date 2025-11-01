# Twilio Conversations Channel — Integration Guide

## Purpose
Outline the interfaces and coordination required between the Twilio channel adapter and other Si systems.

## Upstream Dependencies
- **Channel Credentials Management:** Secure storage and rotation of messaging and voice credentials; integration with secrets management platform.
- **Identity Service:** Mapping of phone numbers and WhatsApp identifiers to Si user accounts, including support for unregistered visitors.
- **Consent Registry:** Central repository ensuring opt-in status is synchronized before allowing message flow.

## Downstream Consumers
- **Session Orchestration:** Receives normalized conversation events, maintains session truth, and exposes state for other channels.
- **Cognition Layer:** Processes normalized requests and returns responses with guidance on segmentation to respect messaging limits.
- **Transcript Storage & Analytics:** Stores messages, voice references, and compliance events for retrieval and reporting.

## Integration Flow
1. **Inbound Webhook Processing:** Twilio posts a conversation event; adapter authenticates, transforms, and enriches it with session metadata.
2. **Policy Enforcement:** Consent, rate limits, and security policies run before forwarding to cognition services.
3. **Response Delivery:** After response generation, adapter segments messages, queues them for delivery, and records status updates supplied by Twilio.
4. **Voice Session Linkage:** When voice escalations occur, adapter associates call events with the existing conversation, ensuring downstream systems receive unified transcripts.
5. **Termination & Archival:** Upon conversation close, adapter triggers archival workflows and releases channel-specific resources.

## Coordination Requirements
- **Schema Stability:** Maintain change logs for payload structures shared with downstream teams; provide sandboxes for validation before production deployment.
- **Monitoring Alignment:** Share channel-specific metrics (delivery rate, escalation frequency, opt-out trends) with observability teams to calibrate alerts.
- **Incident Response:** Define communication playbooks for carrier outages, webhook failures, or credential compromises, including fallback channels for urgent outreach.
