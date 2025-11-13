Title: Streaming Transport — Test Conditions
Status: Draft v1

Latency & Smoothness
- Web P95 time-to-first-token ≤ 800ms; mobile ≤ 2s under nominal conditions.
- No visible stalls > 1s during token emission on baseline connections.

Correctness
- Message sequence follows token → ... → done with occasional heartbeats; errors surface with reason codes.

Resilience
- Client reconnection restores stream on transient network failures without server-side leaks.
- Long streams complete within Lambda/transport limits or are handed off when needed.

Compatibility
- Verified across Chrome, Safari, Firefox; RN on iOS and Android with background/foreground transitions.

