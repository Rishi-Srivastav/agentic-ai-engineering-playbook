# Router

A router selects the most appropriate specialist workflow, model, knowledge source or agent based on the incoming request.

The router is an **orchestration decision**, not an authorization mechanism.

## Architecture

~~~mermaid
flowchart TD
    U[Incoming request] --> R[Router]
    R --> C{Classify intent}
    C --> A[Deterministic workflow]
    C --> B[Reservation specialist]
    C --> D[Flight specialist]
    C --> E[Customer specialist]
    C --> F[Unknown / clarification]
    A --> Z[Result]
    B --> Z
    D --> Z
    E --> Z
    F --> Z
~~~

## Good fit

- Different domains have different tools.
- Some requests require deterministic workflows.
- Model cost or latency varies by task.
- Separate teams own separate capabilities.
- Security boundaries differ between domains.

## Routing contract

Keep the output small and typed:

~~~json
{
  "route": "flight_operations",
  "reason_code": "FLIGHT_STATUS",
  "confidence": 0.94
}
~~~

Treat confidence as a **routing signal**, never as proof of authorization.

## Guardrails

1. Keep the routing schema small.
2. Provide an explicit unknown route.
3. Log route decisions and useful classification signals.
4. Validate the selected route against the authenticated user's permissions.
5. Apply tool authorization again inside the selected workflow.
6. Do not allow the model to dynamically invent privileged routes.
7. Monitor false-routing and fallback rates.

## Router vs multi-agent

A router chooses **where the request should go**. A multi-agent system coordinates **multiple specialized agents during execution**. They can be combined: a router can select a supervisor or specialist workflow, which then executes with its own bounded tools.
