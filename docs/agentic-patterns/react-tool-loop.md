# ReAct / Tool Loop

The ReAct-style pattern lets an agent iteratively decide which tool to call based on observations from previous tool calls.

It is powerful for open-ended tasks, but it is also the pattern most likely to create runaway execution if boundaries are weak.

## Execution flow

~~~mermaid
flowchart TD
    I[User request] --> R[Reason / decide next action]
    R --> G{Need a tool?}
    G -->|No| F[Generate final response]
    G -->|Yes| V[Validate tool + arguments + authorization]
    V --> T[Execute tool]
    T --> O[Observe structured result]
    O --> C{Termination condition met?}
    C -->|No| B{Budget remaining?}
    B -->|Yes| R
    B -->|No| X[Stop safely / fallback]
    C -->|Yes| F
~~~

## Production controls

- Hard maximum iterations.
- Maximum tool calls per request.
- Per-tool timeout.
- Global wall-clock deadline.
- Token and spend budget.
- Duplicate-call detection.
- Tool result size limits.
- Authorization on every tool invocation.
- Explicit terminal conditions.
- Circuit breakers for unhealthy dependencies.

## Duplicate-call detection

Normalize the tool name and arguments and track repeated calls. A model repeatedly issuing the same failing operation should not consume the entire request budget.

## Mutation safety

For writes, add this control chain:

~~~text
Agent → policy → authorization → idempotency check → tool → audit
~~~

Never rely on the model to decide whether a mutation is safe to retry.

## Failure example

If get_flight_status fails with a transient 503, a bounded retry may be reasonable. If change_seat times out after the server may have committed the operation, blindly retrying can create duplicate or conflicting effects. Reconcile state first using an idempotency key or authoritative read.
