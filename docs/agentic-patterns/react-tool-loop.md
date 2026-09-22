# ReAct / Tool Loop

## Flow

```text
Input → reason/select action → validate → tool → observe → decide next step → ... → final
```

## Production controls

- Maximum iterations: hard cap.
- Maximum tool calls per request.
- Per-tool timeout.
- Global wall-clock deadline.
- Token and spend budget.
- Duplicate-call detection.
- Tool result size limits.
- Authorization on every tool invocation.
- Explicit terminal conditions.

## Common failure

A model repeatedly calls the same failing tool. Detect repeated `(tool, normalized_arguments)` tuples and stop or switch to a fallback path.
