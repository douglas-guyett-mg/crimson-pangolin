# Si First-Mile Communications — Canonical Event Schema

## Purpose
Define the data structure and contract for canonical session activity produced by the first-mile communications layer. This schema represents the normalized form of all inbound events (text, audio, control signals) from any channel (messaging, browser, mobile) before they are passed to downstream cognition services.

## Role Within Si
The canonical event schema is the boundary contract between:
- **Upstream**: Channel-specific implementations (Twilio, browser, mobile) that produce raw events
- **Downstream**: Core cognition services (working memory, episodic memory, reasoning engines) that consume normalized events

This schema ensures that downstream systems can process events consistently regardless of origin channel, while allowing channel-specific metadata to be preserved for context and compliance.

## Core Principles
- **Channel-Agnostic**: The schema describes logical structure, not channel-specific implementation details
- **Immutable Contract**: Once defined, the schema is the authoritative specification for event structure
- **Extensible Metadata**: Channel-specific and compliance data is preserved in structured metadata sections
- **Traceable**: Every event carries identifiers enabling end-to-end tracing through the system

## Canonical Event Structure

### Required Core Fields

**session_id** (string, UUID format)
- Unique identifier for the conversation session
- Persists across channel switches and modality changes
- Used for session continuity and context retrieval

**event_id** (string, UUID format)
- Unique identifier for this specific event
- Enables deduplication and tracing
- Immutable once created

**timestamp** (ISO 8601 datetime with timezone)
- When the event was created at the source
- Precision: milliseconds or better
- Used for ordering and latency calculations

**event_type** (string, enumerated)
- Type of event: "message", "audio_frame", "control", "compliance"
- Determines which optional fields are populated
- Immutable

**participant_id** (string)
- Identifier for the user or system entity originating the event
- May be anonymized or pseudonymous based on compliance level
- Used for session binding and audit trails

**channel** (string, enumerated)
- Origin channel: "messaging", "browser", "mobile"
- Determines channel-specific capabilities and constraints
- Used for response routing and capability negotiation

### Content Fields (varies by event_type)

**text_content** (string, optional)
- For "message" and "control" events
- The actual text message or command
- May be empty for audio-only events

**audio_metadata** (object, optional)
- For "audio_frame" events
- Fields: format (e.g., "opus", "pcm"), sample_rate, duration_ms, encoding
- Describes the audio payload without including raw audio data

**modality** (string, enumerated)
- "text", "audio", or "blended"
- Indicates which modalities are active in this event
- Used for response format selection

### Metadata Section

**metadata** (object)
- Preserves channel-specific and compliance information
- Structured as key-value pairs
- Does not affect core event processing

**metadata.channel_metadata** (object)
- Channel-specific fields (e.g., Twilio message SID, browser session token)
- Opaque to downstream systems but preserved for audit and debugging

**metadata.compliance** (object)
- Compliance-related fields: consent_level, recording_disclosed, retention_policy
- Used by downstream systems for compliance enforcement
- Immutable once set

**metadata.source_metadata** (object)
- Information about event origin: user_agent, ip_address_hash, device_type
- Used for analytics and security monitoring
- May be redacted based on privacy policies

### Correlation Fields

**correlation_id** (string, optional)
- Links related events (e.g., request-response pairs)
- Enables tracing across system boundaries
- Inherited from parent events when applicable

**parent_event_id** (string, optional)
- For events that are responses or continuations
- Links back to the originating request
- Enables conversation reconstruction

## Example Canonical Events

### Text Message Event
```json
{
  "session_id": "sess_abc123def456",
  "event_id": "evt_xyz789",
  "timestamp": "2025-10-30T14:23:45.123Z",
  "event_type": "message",
  "participant_id": "user_001",
  "channel": "messaging",
  "text_content": "What's the weather in New York?",
  "modality": "text",
  "metadata": {
    "channel_metadata": {
      "twilio_message_sid": "SM1234567890abcdef"
    },
    "compliance": {
      "consent_level": "full",
      "recording_disclosed": false,
      "retention_policy": "standard"
    },
    "source_metadata": {
      "user_agent": "Twilio SMS",
      "device_type": "phone"
    }
  }
}
```

### Audio Frame Event
```json
{
  "session_id": "sess_abc123def456",
  "event_id": "evt_audio_001",
  "timestamp": "2025-10-30T14:24:12.456Z",
  "event_type": "audio_frame",
  "participant_id": "user_001",
  "channel": "browser",
  "audio_metadata": {
    "format": "opus",
    "sample_rate": 48000,
    "duration_ms": 100,
    "encoding": "base64"
  },
  "modality": "audio",
  "correlation_id": "corr_stream_001",
  "metadata": {
    "channel_metadata": {
      "browser_session_token": "tok_browser_xyz"
    },
    "compliance": {
      "consent_level": "full",
      "recording_disclosed": true,
      "retention_policy": "standard"
    },
    "source_metadata": {
      "user_agent": "Mozilla/5.0",
      "device_type": "desktop"
    }
  }
}
```

### Control Event
```json
{
  "session_id": "sess_abc123def456",
  "event_id": "evt_control_001",
  "timestamp": "2025-10-30T14:25:00.000Z",
  "event_type": "control",
  "participant_id": "user_001",
  "channel": "mobile",
  "text_content": "switch_modality:audio",
  "modality": "text",
  "metadata": {
    "channel_metadata": {
      "mobile_session_id": "mob_sess_123"
    },
    "compliance": {
      "consent_level": "full",
      "recording_disclosed": false,
      "retention_policy": "standard"
    }
  }
}
```

## Constraints and Validation

- **session_id** and **event_id** must be valid UUIDs
- **timestamp** must be in ISO 8601 format with timezone
- **event_type** must be one of: "message", "audio_frame", "control", "compliance"
- **channel** must be one of: "messaging", "browser", "mobile"
- **modality** must be one of: "text", "audio", "blended"
- **text_content** must be non-empty for "message" and "control" events
- **audio_metadata** must be present for "audio_frame" events
- All required fields must be present; missing fields cause validation failure

## Extensibility

New fields may be added to the **metadata** section without breaking downstream systems. However:
- Core fields (session_id, event_id, timestamp, etc.) are immutable
- New event_types require coordination with downstream systems
- Channel-specific metadata should be namespaced under **metadata.channel_metadata**

