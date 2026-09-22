# Common Production Debugging Scenarios

## 1. Correct document exists, but answer says “I don't know”

Check: ingestion status → document version → chunking → query embedding → metadata filters → top-K → reranker → context assembly.

## 2. Hallucination despite RAG

Check whether the required evidence was retrieved. If yes, inspect context ordering, truncation, prompt instructions, and citation mapping. If no, fix retrieval before changing generation.

## 3. Sudden latency increase

Break latency into: guardrails + retrieval + reranking + model time-to-first-token + tool calls + serialization. Compare p50/p95/p99 and dependency traces.

## 4. Token usage spike

Inspect context growth, conversation history, tool outputs, duplicate retrieval, and recursive agent loops. Add hard budgets.

## 5. MCP tool intermittently fails

Check client/server connection lifecycle, timeout mismatch, dependency health, connection pool exhaustion, rate limits, and cancellation handling.

## 6. Agent loops forever

Look for missing terminal condition, repeated tool arguments, ambiguous tool errors, or a planner that keeps generating equivalent steps. Enforce iteration and wall-clock limits.

## 7. Users see another tenant's data

Treat as a security incident. Verify authorization at retrieval and tool layers; do not rely on prompt instructions. Audit tenant context propagation.

## 8. RAG answers are stale

Check source freshness, sync/index timestamps, version metadata, cache TTLs, and whether old chunks remain indexed.

## 9. Tool executes wrong action

Compare model-selected arguments with validated arguments. Confirm authorization is evaluated after argument validation and before the side effect.

## 10. Production differs from evaluation

Compare model/version, prompts, retrieval index, feature flags, tool schemas, temperature/configuration, context size, and policy versions. Capture an immutable trace for replay.
