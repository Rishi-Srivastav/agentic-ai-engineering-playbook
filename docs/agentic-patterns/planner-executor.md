# Planner–Executor

Planner–executor separates **what should happen** from **how authorized actions are executed**.

This is useful when a request contains multiple dependent steps, but execution still needs deterministic validation and policy controls.

## Architecture

~~~mermaid
sequenceDiagram
    participant U as User
    participant P as Planner
    participant V as Plan Validator
    participant E as Executor
    participant T as Tools
    participant O as Observation

    U->>P: Goal
    P->>P: Produce structured plan
    P->>V: Validate steps, dependencies and permissions
    alt Invalid plan
        V-->>P: Reject / repair
    else Valid plan
        V-->>E: Approved executable plan
        loop Each executable step
            E->>T: Execute authorized step
            T-->>O: Result
            O-->>E: Observation
            E->>V: Check preconditions / policy
            V-->>E: Continue / stop / re-plan
        end
        E-->>U: Final result
    end
~~~

## Structured plan

Use typed plans rather than free-form prose:

~~~json
{
  "steps": [
    {"id":"s1","action":"lookup_customer","depends_on":[]},
    {"id":"s2","action":"create_case","depends_on":["s1"]}
  ]
}
~~~

## Critical security boundary

The executor must **not** gain permissions simply because the planner requested an action.

For every step:

1. Validate the action against an allow-list/tool contract.
2. Re-check authorization using the authenticated identity.
3. Validate arguments and resource ownership.
4. Apply policy and risk controls.
5. Enforce timeout and retry rules.
6. Record the execution result.
7. Stop or re-plan when preconditions fail.

## Common failure modes

- Planner invents unavailable capabilities.
- Plan becomes stale while execution is in progress.
- A failed step causes unsafe downstream execution.
- The planner requests a mutation that the user never authorized.
- Long plans exceed context, latency or cost budgets.

For high-impact mutations, combine Planner–Executor with a human approval gate.
