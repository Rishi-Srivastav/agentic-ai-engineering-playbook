# Router

A router selects a specialist workflow, model, or knowledge source.

## Good fit

- Different domains have different tools.
- Some requests require deterministic workflows.
- Model cost/latency varies by task.

## Guardrails

- Keep the routing schema small.
- Provide an explicit `unknown` route.
- Log route decisions and confidence signals.
- Do not use confidence as authorization.
