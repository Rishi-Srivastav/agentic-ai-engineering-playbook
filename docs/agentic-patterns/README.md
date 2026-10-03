# Agentic Patterns

This catalogue explains the most reusable production patterns for agentic systems. Each pattern answers four questions: **when to use it, how control flows, where it fails, and what production guardrails are required**.

> **Principal-engineering rule:** start with the simplest deterministic architecture that satisfies the business requirement. Introduce an autonomous agent loop only where dynamic reasoning, tool selection, or adaptation creates measurable value.

## Pattern landscape

| Pattern | Best fit | Core control problem |
|---|---|---|
| ReAct / tool loop | Dynamic tool selection and iterative problem solving | Prevent runaway or unsafe loops |
| Planner–executor | Multi-step tasks with dependencies | Prevent plan drift and unauthorized execution |
| Router | Different domains, models, or workflows | Prevent incorrect routing |
| Reflection | Improve a generated result against explicit criteria | Control extra latency/cost |
| Evaluator–optimizer | Quality-sensitive generation | Make evaluation reliable and reproducible |
| Human-in-the-loop | High-impact or irreversible actions | Make approval explicit and auditable |
| Multi-agent | Strong domain specialization or isolation | Control coordination complexity |
| Workflow + agent | Enterprise processes with bounded autonomy | Keep deterministic controls around the agent |

## How to choose

~~~mermaid
flowchart TD
    A[Business request] --> B{Is the path deterministic?}
    B -->|Yes| C[Use workflow / service orchestration]
    B -->|No| D{Does the agent need tools repeatedly?}
    D -->|Yes| E[ReAct / Tool Loop]
    D -->|No| F{Are there multiple specialists?}
    F -->|Yes| G[Router or Multi-Agent]
    F -->|No| H{Does the task require multiple dependent steps?}
    H -->|Yes| I[Planner–Executor]
    H -->|No| J{Is output quality improved by critique?}
    J -->|Yes| K[Reflection / Evaluator–Optimizer]
    J -->|No| L[Simple single-agent interaction]
    E --> M{High-impact side effect?}
    I --> M
    G --> M
    L --> M
    M -->|Yes| N[Human approval / policy gate]
    M -->|No| O[Bounded autonomous execution]
~~~

## Cross-cutting production controls

Regardless of the pattern, define:

- **Identity and authorization** at the application/tool boundary.
- **Tool contracts** with typed inputs and outputs.
- **Timeouts, retries and circuit breakers** for external dependencies.
- **Iteration and cost budgets** for autonomous loops.
- **Idempotency** for mutations.
- **Audit events** for consequential actions.
- **PII and secret minimization** in prompts, tool outputs and logs.
- **Evaluation gates** before model/prompt/tool changes reach production.
- **Observability** for latency, tool calls, failures, token usage and business outcomes.
- **Explicit termination conditions** rather than assuming the model will stop safely.

## Mental model

Think of an agentic architecture as:

~~~mermaid
flowchart LR
    U[User / API Client] --> A[Application Boundary]
    A --> AG[Agent / Reasoning]
    AG --> P[Policy + Authorization]
    P --> T[Tools / APIs]
    T --> D[Enterprise Systems]
    D --> T
    T --> AG
    AG --> V[Validation / Evaluation]
    V --> A
    A --> U
~~~

The model provides **reasoning**, but the surrounding application provides **authority, constraints, state, reliability and auditability**.
