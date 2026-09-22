# Agentic AI Engineering Playbook

Production-oriented patterns and implementation guidance for **agentic AI, RAG, MCP servers, guardrails, evaluation, and debugging**.

> Goal: make AI systems understandable, testable, observable, secure, and operable—not just impressive in a demo.

## What this repo covers

| Area | Topics |
|---|---|
| Agentic patterns | Tool calling, ReAct, planner/executor, router, reflection, evaluator-optimizer, human-in-the-loop, multi-agent, workflow vs agent |
| RAG | Ingestion, chunking, metadata, embeddings, hybrid retrieval, reranking, query transformation, citations, evaluation, production tuning |
| MCP | Server design, tools/resources/prompts, schemas, security, authorization, lifecycle, errors, observability, testing |
| Guardrails | Input/output validation, prompt injection, PII, authorization, tool permissions, budgets, rate limits, human approval |
| Production debugging | Retrieval failures, hallucinations, latency, token spikes, tool failures, MCP connectivity, stale indexes, model errors |
| Evaluation | Golden sets, retrieval metrics, answer quality, agent trajectories, regression tests, cost/latency SLOs |

## Recommended learning path

1. `docs/architecture/ai-system-reference-architecture.md`
2. `docs/agentic-patterns/` — choose the smallest pattern that solves the problem
3. `docs/rag/` — build grounding before adding agent autonomy
4. `docs/mcp/` — expose narrow, typed, least-privilege capabilities
5. `docs/guardrails/` — secure every trust boundary
6. `docs/evaluation/` — measure before optimizing
7. `docs/production-debugging/` — debug from traces, not guesses

## Design principles

- **Deterministic first:** use normal code/workflows when the path is known; use agents when dynamic decisions are valuable.
- **Least privilege:** an LLM should receive only the tools, data, and permissions required for the task.
- **Typed boundaries:** validate tool inputs and outputs like public APIs.
- **Ground before generate:** retrieve authoritative context and preserve provenance.
- **Fail closed:** authorization, destructive actions, and policy checks should default to deny.
- **Bound autonomy:** cap iterations, tool calls, tokens, wall-clock time, and spend.
- **Observe every step:** capture request IDs, model calls, retrieval results, tool calls, latency, tokens, and policy decisions.
- **Evaluate continuously:** maintain a golden dataset and regression suite for retrieval, generation, and agent trajectories.

## Reference architecture

```text
User/API
   |
   v
API Gateway / Auth
   |
   v
Agent Orchestrator -----> Policy / Guardrail Engine
   |                              |
   +----> LLM --------------------+
   |       |                       |
   |       +----> RAG ------------+----> Vector / Search Store
   |       |
   |       +----> MCP Client -----> MCP Servers -----> Enterprise APIs
   |
   +----> Memory / State
   |
   v
Observability / Tracing / Evaluation / Audit
```

## AWS / Bedrock notes

The RAG guidance is compatible with Amazon Bedrock Knowledge Bases. Bedrock exposes both direct retrieval and retrieve-and-generate flows; direct retrieval is useful when you need to inspect, rerank, filter, or evaluate retrieved chunks before generation. See the AWS documentation linked in `docs/rag/bedrock-reference.md`.

## Repository structure

```text
docs/
  architecture/
  agentic-patterns/
  rag/
  mcp/
  guardrails/
  evaluation/
  production-debugging/
examples/
  agent/
  rag/
  mcp/
checklists/
templates/
```

## Production readiness checklist

See `checklists/production-readiness.md`. The short version:

- Authentication + authorization
- Prompt injection defenses
- Tool permission boundaries
- Input/output schemas
- PII/secrets handling
- Timeouts/retries/circuit breakers
- Token/cost budgets
- Retrieval quality and freshness
- Citation/provenance
- Traceability and audit logs
- Golden-set regression tests
- Rollback and kill switch
- Human approval for high-impact actions

## References

- Model Context Protocol: https://modelcontextprotocol.io/
- MCP specification: https://modelcontextprotocol.io/specification/
- Amazon Bedrock Knowledge Bases: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html
- AWS Bedrock retrieval: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-retrieval.html
- AWS RAG evaluation: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-create.html
- OWASP GenAI Security: https://genai.owasp.org/

## License

MIT — see `LICENSE`.
