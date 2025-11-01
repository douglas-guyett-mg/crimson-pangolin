# Session Orchestration — Test Conditions

## Lifecycle Validation
- **Create & Resume:** Initiate sessions from each channel, verify identifiers are unique yet reusable, and confirm resume requests restore prior state.
- **Suspension & Termination:** Trigger inactivity timeouts and explicit closures; ensure downstream services receive notifications and no further events are accepted.
- **Concurrent Access:** Simulate multiple channels interacting with the same session; confirm conflict resolution policies maintain single source of truth.

## Policy Enforcement
- **Rate Limiting:** Exceed configured event rates; ensure throttling engages and provides actionable feedback to calling adapters.
- **Concurrency Rules:** Attempt simultaneous voice sessions beyond allowed limits; verify requests are denied with compliant error messaging.
- **Consent Updates:** Change consent status mid-session; confirm immediate enforcement and distribution to interested services.

## State Integrity
- **Snapshot Accuracy:** Request state snapshots at various points; validate contents (active modality, participants, last activity) against actual conversation events.
- **Event Ordering:** Inject out-of-order events; ensure orchestration reorders or rejects them according to policy.
- **Audit Completeness:** Review audit logs for sample sessions; confirm lifecycle events include required metadata and trace identifiers.

## Resilience & Recovery
- **Storage Failover:** Simulate backend storage outages; verify failover or graceful degradation without losing committed session data.
- **Restart Scenarios:** Restart orchestration components during active sessions; ensure state restores correctly and channels reconnect successfully.
- **Cross-Region Sync:** Run sessions across multiple regions; confirm replication latency stays within agreed bounds.
