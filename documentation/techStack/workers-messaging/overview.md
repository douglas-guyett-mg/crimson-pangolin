Title: Workers and Messaging (SQS/EventBridge) — Overview
Status: Draft v1
Scope: Asynchronous execution backbone for daemon coordination, retries, and background processing.

Queueing & Events
- SQS Standard by default; FIFO for ordered flows (declare explicitly) with content-based dedup.
- DLQs for all queues with alarms; visibility timeouts tuned per task type.
- EventBridge for fan-out domain events (e.g., TurnStarted, TurnCompleted, MemoryWritten).

Idempotency & Ordering
- Idempotency keys stored in DynamoDB; handlers must be idempotent.
- For FIFO queues, message group id set to entity id where strict order is needed.

Retries & Backoff
- Exponential backoff with max attempts; poison messages routed to DLQ; replay tooling with safeguards.

Contracts & Schemas
- Versioned event schemas; backward-compatible evolution; schema registry documented.
- Correlation with Turn Trace ids for end-to-end visibility.

Operations
- Throughput targets per queue; alarms on ApproximateAgeOfOldestMessage and DLQ depth.
- Cost control via batching and concurrency limits; worker autoscaling policies.

When to use Step Functions
- Long-running orchestrations or human-in-the-loop flows; otherwise prefer event-driven SQS workers.

