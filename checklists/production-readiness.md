# Production Readiness Checklist

## Architecture
- [ ] Workflow vs agent decision documented
- [ ] Trust boundaries documented
- [ ] Failure modes identified

## Security
- [ ] Authentication
- [ ] Per-tool authorization
- [ ] Tenant isolation
- [ ] Prompt-injection tests
- [ ] PII/secret controls
- [ ] Output validation

## Reliability
- [ ] Timeouts
- [ ] Retries with backoff where safe
- [ ] Circuit breakers
- [ ] Idempotency
- [ ] Kill switch
- [ ] Bounded agent loops

## RAG
- [ ] Source versioning
- [ ] Metadata/ACL filtering
- [ ] Retrieval evaluation
- [ ] Citation mapping
- [ ] Freshness monitoring

## Observability
- [ ] Correlation IDs
- [ ] Distributed traces
- [ ] Token/cost metrics
- [ ] Tool-call audit
- [ ] Policy decisions logged safely

## Evaluation
- [ ] Golden set
- [ ] Regression suite
- [ ] Adversarial cases
- [ ] Online quality metrics
- [ ] Rollback criteria
