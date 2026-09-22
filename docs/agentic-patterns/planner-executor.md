# Planner–Executor

Separate planning from execution when tasks contain multiple dependent steps.

## Pattern

```text
Goal → Planner → structured plan → Executor → observation → next step → final
```

Use a typed plan such as:

```json
{"steps":[{"id":"s1","action":"lookup_customer","depends_on":[]},{"id":"s2","action":"create_case","depends_on":["s1"]}]}
```

Validate the plan before execution. The executor must not automatically gain permissions simply because the planner requested an action.
