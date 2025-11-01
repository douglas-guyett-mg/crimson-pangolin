# Twilio Conversations Channel — Overview

## Purpose
Describe the functional expectations for the Twilio-hosted messaging and voice entry point that enables users to interact with Si through SMS, WhatsApp, and optional call escalation.

## Channel Scope
- **Messaging:** Bidirectional text exchange via SMS and WhatsApp using the Conversations model (participants, conversations, messages, events).
- **Voice Escalation:** Ability to initiate or receive calls tied to an existing conversation, enabling real-time audio alongside text history.
- **Notification Handling:** Delivery receipts, read indicators, and error callbacks inform reliability monitoring and user-facing status updates.

## Key Responsibilities
- **Conversation Management:** Create or join conversations based on user identifiers, ensuring that conversation state remains synchronized with Si sessions.
- **Participant Governance:** Enforce enrollment, consent, and opt-in requirements for each participant before allowing message flow.
- **Message Normalization:** Convert Twilio conversation messages (text, media, metadata) into canonical Si events, capturing timestamps, participant identity, channel tags, and compliance attributes.
- **Voice Session Binding:** When a call is initiated, bind call state to the existing conversation and maintain transcript continuity between text and voice exchanges.
- **Lifecycle Management:** Detect conversation inactivity, issue warnings before closure, and archive transcripts for compliance and analytics.

## User Journeys
1. **SMS/WhatsApp Onboarding:** User sends an introductory message; system validates opt-in, creates conversation if needed, and responds with a welcome message plus consent confirmation.
2. **Ongoing Chat:** User and Si exchange messages; rich media (images, documents) are stored with references while the textual narrative remains accessible.
3. **Voice Escalation:** User requests a call; Si initiates or schedules voice interaction, linking audio stream and transcript to the conversation history.
4. **Session Closure:** After inactivity or explicit user command, conversation closes with final summary and instructions for re-entry.

## Constraints & Considerations
- Must respect channel-specific throughput limits and message length restrictions, splitting or queuing responses when necessary.
- Handle opt-out keywords immediately and record cessation of communication as required by regulations.
- Ensure localized messaging for international participants, including fallback content when language resources are unavailable.
- Voice escalation must comply with recording disclosure rules, offering users the ability to continue via text if they decline.

## Success Indicators
- Conversations align 1:1 with Si session identifiers and remain consistent across text and voice modalities.
- Delivery success rates meet or exceed targets, with automatic remediation for transient errors.
- Compliance events (opt-ins, opt-outs, disclosures) are traceable in logs and available for audits within mandated retention windows.
