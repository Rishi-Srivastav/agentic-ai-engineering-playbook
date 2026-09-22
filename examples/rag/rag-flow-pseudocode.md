# RAG Flow — Pseudocode

```text
query = normalize(user_query)
query = optional_query_transform(query)

candidates = hybrid_or_vector_search(query, top_k=50)
candidates = enforce_acl(candidates, user_context)
candidates = rerank(candidates)
context = fit_to_context_budget(candidates)

answer = model.generate(
    question=user_query,
    context=context,
    instruction="Answer only from supported evidence; cite sources."
)

return validate_citations(answer, context)
```
