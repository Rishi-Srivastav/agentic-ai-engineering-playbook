# Agentic Patterns

The pattern catalogue below is intentionally implementation-oriented.

| Pattern | Primary use | Main risk |
|---|---|---|
| ReAct / tool loop | Dynamic tool selection | runaway loops |
| Planner-executor | Multi-step tasks | plan drift |
| Router | Route requests to specialists | misclassification |
| Reflection | Improve a draft/result | extra cost/latency |
| Evaluator-optimizer | Generate → score → improve | evaluator bias |
| Human-in-the-loop | High-impact decisions | approval bottleneck |
| Multi-agent | Specialized collaboration | coordination complexity |
| Workflow + agent | Controlled autonomy | boundary design |

Rule of thumb: do not introduce an autonomous loop until a deterministic workflow is insufficient.
