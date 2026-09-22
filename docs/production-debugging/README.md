# Production Debugging

Debug AI systems by tracing the pipeline in order.

```text
Request
  ↓
Policy
  ↓
Retrieval
  ↓
Prompt/context
  ↓
Model
  ↓
Tool calls
  ↓
Output policy
  ↓
Response
```

Never jump directly from “bad answer” to “change the prompt.”
