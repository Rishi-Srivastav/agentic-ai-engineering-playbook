# MCP Lifecycle and Errors

Implement explicit startup, capability negotiation, request correlation, cancellation/timeouts, and graceful shutdown according to the MCP version you support.

## Error taxonomy

```text
INVALID_ARGUMENT
UNAUTHORIZED
FORBIDDEN
NOT_FOUND
CONFLICT
RATE_LIMITED
DEPENDENCY_TIMEOUT
DEPENDENCY_UNAVAILABLE
INTERNAL
```

Do not leak stack traces, credentials, SQL, or internal topology into model-visible errors. Return a concise safe message and log the detailed diagnostic server-side.
