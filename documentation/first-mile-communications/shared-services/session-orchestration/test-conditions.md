# Session Orchestration — Test Conditions

## Lifecycle Validation
- **Create & Resume:** Initiate sessions from each channel, verify identifiers are unique yet reusable, and confirm resume requests restore prior state.
- **Suspension & Termination:** Trigger inactivity timeouts and explicit closures; ensure downstream services receive notifications and no further events are accepted.
- **Concurrent Access:** Simulate multiple channels interacting with the same session; confirm last-write-wins with vector clocks for conflict resolution, conflicts resolved within 100ms, audit log records all conflict resolutions with timestamps and winning channel ID, single source of truth maintained (verified by checksum comparison across all replicas).

## Policy Enforcement
- **Rate Limiting:** Exceed configured event rates (send 150 events/min when limit is 100/min); ensure throttling engages and returns HTTP 429 with JSON response: {error: "Rate limit exceeded", retry_after: <seconds>, current_rate: <events/min>, limit: <events/min>}, subsequent requests blocked for calculated retry_after duration.
- **Concurrency Rules:** Attempt simultaneous voice sessions beyond allowed limits; verify requests are denied with compliant error messaging.
- **Consent Updates:** Change consent status mid-session (withdraw consent for audio recording); confirm changes propagated to all services within 500ms, new audio events rejected within 100ms with error "Consent withdrawn", existing audio streams terminated immediately, audit log records consent change with timestamp and initiating party.

## State Integrity
- **Snapshot Accuracy:** Request state snapshots at various points; validate contents (active modality, participants, last activity) against actual conversation events.
- **Event Ordering:** Inject out-of-order events (events with timestamp delta > 5 seconds from expected sequence); ensure orchestration rejects events beyond tolerance with error code ORD_001 "Event timestamp out of sequence", events within 5-second tolerance window reordered using Lamport timestamps, final event stream maintains causal ordering, audit log records all reordering operations.
- **Audit Completeness:** Review audit logs for sample sessions; confirm lifecycle events include required metadata and trace identifiers.

## Resilience & Recovery
- **Storage Failover:** Simulate backend storage outages; verify failover or graceful degradation without losing committed session data.
- **Restart Scenarios:** Restart orchestration components during active sessions; ensure state restores correctly and channels reconnect successfully.
- **Cross-Region Sync:** Run sessions across multiple regions (US-East, EU-West, APAC); confirm P99 cross-region replication latency < 2 seconds, P50 < 500ms, measured at 1-minute intervals, eventual consistency achieved within 5 seconds, conflict resolution uses region-aware vector clocks.
