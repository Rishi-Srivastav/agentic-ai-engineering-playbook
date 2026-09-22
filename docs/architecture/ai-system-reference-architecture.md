# AI System Reference Architecture

## Core layers

1. **Experience/API** — authentication, tenant context, request validation.
2. **Orchestration** — workflow or agent loop, state, budgets.
3. **Model gateway** — provider abstraction, model routing, retries, fallbacks.
4. **Knowledge** — ingestion, retrieval, reranking, citations.
5. **Tools** — MCP or internal tool APIs with typed contracts.
6. **Policy** — authorization, safety, data access, approval.
7. **State** — conversation state, durable task state, memory.
8. **Observability** — traces, metrics, logs, audit events.
9. **Evaluation** — offline datasets, online monitoring, regression gates.

## Workflow vs agent

Use a workflow when the sequence is known. Use an agent when the system must select among actions dynamically. A useful architecture often combines both: deterministic outer workflow + bounded agentic step.

## Trust boundaries

Treat these as separate trust boundaries:

- user input → model
- model → tool
- tool → enterprise system
- retrieved document → model
- model output → application
- agent state → next iteration

Every boundary needs validation, authorization, and telemetry.
