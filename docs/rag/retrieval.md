# Retrieval

## Baseline

1. Embed the query.
2. Retrieve top-K candidates.
3. Apply metadata/security filters.
4. Rerank if needed.
5. Select a context budget.
6. Generate with source attribution.

## Hybrid retrieval

Combine semantic retrieval with lexical retrieval when exact identifiers, codes, names, or error messages matter.

## Query transformation

For difficult questions, consider decomposition, query expansion, or hypothetical-document techniques—but measure whether the extra model call improves retrieval enough to justify its cost and latency.

## Retrieval debugging

Log candidate IDs, scores, rank, filters, source version, and reranker score. A final answer without retrieval telemetry is difficult to diagnose.
