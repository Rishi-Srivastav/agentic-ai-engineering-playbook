# Ingestion and Chunking

## Ingestion checklist

- Preserve source identifiers and versions.
- Normalize encodings and boilerplate.
- Extract headings, tables, lists, and page numbers where useful.
- Attach access-control metadata.
- Make ingestion idempotent.
- Track source hash/version and ingestion timestamp.

## Chunking

Start with semantic boundaries rather than a fixed token count. Preserve enough context for a chunk to stand alone, but avoid huge chunks that dilute retrieval precision.

Store metadata such as:

```json
{"document_id":"policy-123","version":"7","section":"refunds","tenant":"acme","acl":["support"],"updated_at":"2026-09-01"}
```

Never rely on the model to enforce ACL metadata. Apply access filtering before context reaches the model.
