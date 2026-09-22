# Amazon Bedrock Knowledge Bases Reference

Amazon Bedrock Knowledge Bases supports direct retrieval as well as retrieve-and-generate workflows. Direct `Retrieve` is valuable when an application needs to inspect or transform retrieved results before generation. AWS also documents reranking and RAG evaluation capabilities.

Production debugging should distinguish:

- source ingestion/sync failure
- parsing/chunking failure
- embedding/indexing failure
- retrieval/filter failure
- reranking failure
- generation failure
- citation/provenance failure

Useful official references:

- Retrieval: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-retrieval.html
- How KBs work: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html
- RAG evaluation: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-create.html
- Agentic retrieval: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-test-agentic-retrieve.html
