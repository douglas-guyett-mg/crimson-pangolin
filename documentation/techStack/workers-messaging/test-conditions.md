Title: Workers and Messaging — Test Conditions
Status: Draft v1

Idempotency & Ordering
- Duplicate deliveries do not produce duplicate side effects; idempotency verified under retry storms.
- FIFO flows preserve order within a group; reordering detected in tests for Standard queues when acceptable.

Reliability
- Failed messages reach DLQ after max attempts; alarms triggered; replay tool restores successfully.

Performance
- Queues meet throughput targets; P95 processing latency within bounds; visibility timeouts sufficient.

Contracts
- Events validate against versioned schemas; consumers tolerate added fields.

Observability
- Each message/handler execution emits Turn Trace correlation and OTel spans; dashboards display backlog and errors.

