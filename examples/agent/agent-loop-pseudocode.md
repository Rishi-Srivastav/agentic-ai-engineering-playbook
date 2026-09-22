# Bounded Agent Loop — Pseudocode

```text
state = initialize(request)
deadline = now + MAX_WALL_TIME

for step in 1..MAX_STEPS:
    if now >= deadline: return bounded_timeout()

    decision = model.decide(state)
    validate(decision)

    if decision.type == FINAL:
        return output_policy(decision.answer)

    authorize(decision.tool, user_context)
    enforce_budget(decision)

    result = tool.invoke(decision.tool, decision.arguments)
    state = append_observation(state, sanitize(result))

return bounded_failure()
```
