# RAG Engineering

RAG quality is a pipeline problem, not only a prompt problem.

```text
Sources → ingestion → parsing → chunking → metadata → embeddings/index
                                           |
Query → normalization → retrieval → filters → reranking → context assembly → LLM → citations
```

Tune each stage independently.
