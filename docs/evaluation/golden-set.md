# Golden Set Template

Each test case should contain:

```json
{
  "id": "rag-001",
  "input": "...",
  "expected_sources": ["doc-12"],
  "expected_behavior": "answer_with_citation",
  "must_not": ["invent_policy"],
  "risk": "medium"
}
```

Include positive, negative, adversarial, stale-data, missing-data, authorization, and tool-failure cases.
