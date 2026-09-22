# MCP Observability

Every request should have a correlation ID and record:

- client/server identity
- tool name
- sanitized arguments hash
- authorization decision
- start/end timestamps
- latency
- dependency calls
- result size
- error class
- retry count

Metrics:

- tool success rate
- p50/p95/p99 latency
- timeout rate
- authorization denials
- validation failures
- payload size
- dependency error rate

Never log raw secrets or sensitive tool arguments by default.
