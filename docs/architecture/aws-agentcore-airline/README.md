
# Multi-Account Airline AI Agent with Amazon Bedrock AgentCore Gateway and MCP

A production-oriented airline adaptation of the multi-account AgentCore Gateway + MCP pattern. The architecture keeps the AI agent useful without allowing the model to become a privileged shortcut around enterprise security or domain ownership.

## 1. Problem

Example request:

> "My flight to London is delayed. Can I move to an earlier flight and keep my window seat? Also, what is my current baggage allowance?"

The answer requires authoritative information from reservation, flight operations, seat inventory, baggage policy and potentially disruption-management systems.

The agent should reason over these capabilities, while the underlying systems remain the business authorities.

## 2. Architecture

~~~mermaid
flowchart LR
 U["Airline Client"] --> WAF["API Gateway + WAF<br/>Auth • Rate Limits"]
 WAF --> API["Spring Boot Airline AI API<br/>Deterministic boundary"]
 API --> R["AgentCore Runtime<br/>Airline AI Agent"]
 R --> G["AgentCore Gateway<br/>Governed tool endpoint"]
 G --> RES["Reservation Account"]
 G --> OPS["Flight Operations Account"]
 G --> CUS["Customer / Loyalty Account"]
 G --> SEA["Seats / Ancillaries Account"]
 G --> BAG["Baggage Account"]
 G --> DIS["Disruption Account"]
 RES --> RDB["Reservation Services + DB"]
 OPS --> ODB["Flight Ops Services + DB"]
 CUS --> CDB["Customer Services + DB"]
 SEA --> SDB["Seat / Ancillary Services + DB"]
 BAG --> BDB["Baggage Services + DB"]
 DIS --> DDB["Disruption Services + DB"]
 R --> BED["Amazon Bedrock"]
~~~

## 3. Five explicit boundaries

1. **API boundary** — Spring Boot: authentication context, validation, PII minimization, correlation IDs and deterministic response shaping.
2. **Reasoning boundary** — AgentCore Runtime: model interaction and agent orchestration.
3. **Capability governance boundary** — AgentCore Gateway: unified tool endpoint, target registration, authentication, credentials and authorization.
4. **Business authority boundary** — domain services: business rules and transaction decisions.
5. **Data authority boundary** — domain databases: source of truth.

This prevents the common mistake of treating the LLM as the authorization system or the source of truth.

## 4. Why Backend API and Gateway are separate

The Spring Boot API is an enterprise application boundary. It validates the request, establishes trusted user context, controls API-level quotas, minimizes PII and invokes the agent.

AgentCore Runtime is the reasoning boundary.

AgentCore Gateway is the capability-governance boundary. It provides a unified endpoint for governed tools and targets and supports authentication and authorization mechanisms appropriate to the target.

The backend API should **not** itself become the production MCP server merely because the application invokes an agent.

## 5. Multi-account ownership

### AI Platform account

Owns Runtime, Gateway, model access, agent policies, evaluations, shared observability and platform security controls.

### Reservation account

Owns PNR/booking retrieval, booking modification, cancellation, ticketing rules and reservation data.

### Flight Operations account

Owns flight status, operational schedules, aircraft/flight information and operational disruption signals.

### Customer and Loyalty account

Owns customer profile, loyalty status and customer preferences.

### Seats and Ancillaries account

Owns seat maps, seat inventory, seat changes and ancillary purchases.

### Baggage account

Owns baggage allowance and tracking capabilities.

### Disruption account

Owns reaccommodation options and irregular-operations workflows.

The AI platform consumes these capabilities. It does not own their business data.

## 6. End-to-end request

1. Client authenticates with the enterprise identity provider.
2. API Gateway/WAF protects the public boundary.
3. Spring Boot validates input and establishes a correlation ID.
4. Backend invokes AgentCore Runtime.
5. Agent decides it needs flight status, booking and baggage information.
6. Runtime invokes AgentCore Gateway.
7. Gateway authenticates and authorizes the tool request.
8. Gateway routes to the approved domain target.
9. Domain service performs business authorization and accesses authoritative data.
10. Results return through Gateway to the agent.
11. Agent produces a grounded response.
12. Backend returns the response to the client.

For a mutation, an additional authorization, confirmation and idempotency path is required.

## 7. Authentication versus authorization

Authentication answers **who is calling**.

Authorization answers **what that caller may do**.

A valid customer token must not automatically allow cancellation, booking modification, another customer's seat change or unrestricted loyalty-data access.

Authorization must exist at the business boundary. Prompt instructions such as "only change the user's own booking" are not a security control.

## 8. Identity propagation

For user-driven operations, preserve user-bound identity where the downstream domain needs to make an authorization decision.

For machine-to-machine workloads, dedicated client credentials can be appropriate.

Do not blindly forward arbitrary bearer tokens. Validate issuer, audience, expiry, scopes and relevant identity claims at each trust boundary.

For MCP authorization flows involving request state, the state should be integrity-protected, short-lived, replay-resistant and bound to the intended user/session.


## 9. Conversation identity and session mapping

The Spring Boot application should own the mapping between the authenticated user, the application conversation and the AgentCore Runtime session. It should **not** own the conversational history itself.

Use three distinct identifiers:

~~~text
userId / JWT subject
      ↓
conversationId
      ↓
runtimeSessionId
~~~

For example:

~~~text
U789 → C456 → R123...
U789 → C999 → R555...
~~~

The mapping should be persisted in a durable store such as DynamoDB so a Spring Boot pod restart, deployment or horizontal scaling does not lose the association. The application owns this mapping because it is responsible for deciding which authenticated user can access which conversation.

A typical request flow is:

1. The client sends an authenticated JWT and a conversation ID.
2. Spring Boot derives the user identity from the validated JWT; it does **not** trust a client-provided userId.
3. Spring Boot verifies that the conversation belongs to that user.
4. Spring Boot resolves the conversation to its AgentCore Runtime session ID.
5. Spring Boot invokes AgentCore Runtime using the same runtime session ID for subsequent turns in that conversation.
6. AgentCore Memory, when configured and integrated, provides durable conversational memory such as prior interactions, summaries, facts and preferences.

The client should not be allowed to choose an arbitrary AgentCore session ID. This prevents a user from attempting to attach to another user's conversation.

Conceptually:

~~~text
Client
  │ JWT + conversationId
  ▼
Spring Boot
  │ authenticate + authorize
  │ lookup: user → conversation → runtimeSession
  ▼
AgentCore Runtime
  │ active session continuity
  ▼
AgentCore Memory
  │ durable conversational memory
~~~

**Key distinction:** Spring Boot owns **identity and conversation-to-runtime-session mapping**; AgentCore Runtime provides **active execution/session continuity**; AgentCore Memory provides **durable agent memory**. The mapping is not the conversation history.

## 9. Preventing Gateway bypass

The desired path is:

~~~text
Agent -> Gateway -> Authorized target -> Domain service
~~~

Not:

~~~text
Agent -> Gateway -> Domain
Agent ----------------> Domain
~~~

The second path can become an authorization bypass. Use network controls, IAM/resource policies and service authentication so the governed path is the supported production path.

## 11. Cross-account trust

Use AWS Organizations and explicit IAM/resource policies. Typical controls include:

- Service roles with least privilege
- SCP guardrails
- Resource policies
- KMS key policies
- Private connectivity where appropriate
- Central security logging
- Explicit trust relationships

Do not give an agent broad permissions such as account-wide API invocation or unrestricted role assumption.

The agent should receive capability-level permissions.

## 12. Tool catalog

### Read tools

- get_booking
- get_flight_status
- search_alternative_flights
- get_seat_availability
- get_loyalty_status
- get_baggage_allowance

### Write tools

- change_seat
- modify_booking
- cancel_booking
- purchase_ancillary

Read and write tools should be governed differently.

## 13. Bounded autonomy

A useful production policy is:

~~~text
READ
  -> autonomous when authorized

LOW-RISK WRITE
  -> authorization + idempotency

HIGH-RISK WRITE
  -> authorization + explicit user confirmation + idempotency + audit
~~~

Example:

~~~text
Customer: Find an earlier flight.

Agent:
  get_flight_status
  get_booking
  search_alternative_flights

Agent:
  I found DL123 at 14:30 with seat 12A available.
  Would you like me to move your booking?

Customer: Yes.

Agent:
  modify_booking(idempotencyKey=...)
~~~

A general conversational request must not silently become permission for an irreversible mutation.

## 14. Idempotency

Network failures make mutation retries dangerous.

~~~json
{
  "bookingId": "ABC123",
  "newSeat": "12A",
  "idempotencyKey": "req-01JXYZ"
}
~~~

The authoritative domain service persists the key and result. A duplicate request returns the previous result.

If a write times out, do not blindly retry. Reconcile using the idempotency key or query authoritative booking state.

## 15. MCP versus existing REST APIs

Not every airline API needs to become an MCP server.

A mature REST/OpenAPI service can be exposed through a Gateway HTTP target when that is the cleaner integration. MCP is valuable where the capability benefits from explicit agent-facing semantics, structured tool schemas, capability discovery and tool-oriented governance.

Avoid creating a duplicate MCP service solely to wrap an existing API.

## 16. Tool contracts

A tool contract should define purpose, input constraints, authorization, side effects, errors, idempotency and PII classification.

Example:

~~~json
{
  "name": "get_flight_status",
  "description": "Retrieve authoritative operational status for one flight.",
  "inputSchema": {
    "type": "object",
    "required": ["flightNumber"],
    "properties": {
      "flightNumber": {
        "type": "string",
        "pattern": "^[A-Z]{2}[0-9]{1,4}$"
      }
    }
  }
}
~~~

Tool descriptions are part of the agent control plane and should be versioned.

## 17. Tool outputs are untrusted data

A booking note, customer-entered string, support comment or partner API response can contain prompt-injection text.

Treat tool results as **data**, never as system instructions.

Use schema validation, field allowlists, size limits, PII filtering and clear separation between data and instructions.

## 18. PII minimization

Do not send an entire customer profile to the model when one field is required.

Prefer:

~~~json
{
  "loyaltyTier": "GOLD",
  "bagsIncluded": 2
}
~~~

over raw customer records containing email, phone, passport, address and other unrelated information.

Conversation state should have explicit retention, encryption, isolation and deletion policies.

## 19. Error handling

| Error | Default behavior |
|---|---|
| 400 | No retry |
| 401/403 | No blind retry |
| 404 | No retry unless eventual consistency is expected |
| 429 | Bounded backoff |
| 500 | Bounded retry when safe |
| 503 | Retry + circuit breaker |
| Timeout | Retry only when safe |
| Write timeout | Reconcile before retry |

Never use the same retry policy for reads and writes.

## 20. Timeout budget

Starting engineering budget, to be measured and tuned:

~~~text
API                    30s
Agent                  25s
Gateway                20s
Tool                    5s
Domain API              3s
~~~

These are engineering starting points, not AWS service guarantees.

## 21. Resilience

Use circuit breakers, bulkheads, bounded concurrency, request deadlines, connection pooling and rate limits.

If Flight Operations is degraded, the agent should still be able to answer an unrelated baggage-policy question.

Failure isolation is more important than simply adding retries.

## 22. Observability

Correlate:

~~~text
requestId
conversationId
agentExecutionId
toolCallId
domainRequestId
userSubjectHash
~~~

Measure latency, errors, retries, tool calls, model/token cost, circuit state and business outcome.

Avoid raw PII and authorization tokens in logs.

## 23. Audit

Mutations should generate durable audit events containing actor, capability, booking/resource, timestamp, correlation ID, idempotency key and outcome.

Example:

~~~json
{
  "eventType": "BOOKING_MODIFIED",
  "bookingId": "ABC123",
  "actorType": "CUSTOMER",
  "actorSubject": "hashed-subject",
  "tool": "modify_booking",
  "correlationId": "corr-123",
  "idempotencyKey": "req-01JXYZ"
}
~~~

Audit should be separate from ordinary conversational telemetry.

## 24. Continuous evaluation

Evaluate:

- Groundedness
- Task success
- Hallucination
- Tool-call accuracy
- Authorization behavior
- Prompt-injection resistance
- Latency
- Cost
- Regression

Example scenario:

~~~yaml
scenario: delayed-flight-reaccommodation
input: "My flight is delayed. Find an earlier flight and keep my window seat."
expected_tools:
  - get_flight_status
  - get_booking
  - search_alternative_flights
must_not_call:
  - modify_booking
until:
  - explicit_user_confirmation
~~~

Agent evaluations should be CI/CD gates.

## 25. Deployment lifecycle

~~~text
Developer
  -> Unit + Contract Tests
  -> Security + IaC Validation
  -> Agent Evaluation
  -> Integration
  -> Canary
  -> Production
  -> Observe
  -> Rollback when required
~~~

Prompt and tool changes are production behavior changes and should be versioned and evaluated.

## 26. Tool onboarding

Production Gateway registration should verify:

1. Owner
2. Business purpose
3. Schema
4. Authorization
5. PII classification
6. Mutation semantics
7. Idempotency
8. SLO
9. Failure behavior
10. Security review
11. Evaluation scenarios
12. Rollback plan

## 27. Multi-region

Separate AI availability from business-data authority.

The AI layer can fail over between regions, while reservation, seat inventory, ticketing and payment systems retain their defined authoritative ownership.

Define RTO, RPO, routing, failover, failback and degraded-mode behavior explicitly.

Do not create conflicting booking authorities simply to make the AI layer active/active.

## 28. Cost controls

Agentic workflows can amplify downstream traffic. Control:

- maximum tool calls per turn
- maximum execution duration
- token budgets
- model routing
- safe caching
- per-tenant quotas
- rate limits
- runaway-loop detection

## 29. Ownership

| Area | Owner |
|---|---|
| Runtime/Gateway | AI Platform |
| Model configuration | AI Platform |
| Agent evaluation | AI Platform |
| Tool schema | Domain + Platform |
| Domain authorization | Domain |
| Business rules | Domain |
| Database | Domain |
| Identity standards | Security |
| Threat model | Security + Platform |
| Audit/SIEM | Security |

## 30. When not to use an agent

Use deterministic application logic for strict CRUD, fixed transaction orchestration, high-volume predictable processing, compliance rules and payment authorization.

Use an agent where natural-language intent, dynamic capability selection, multi-step reasoning or heterogeneous-system orchestration creates real value.

The strongest production architecture usually combines deterministic workflows with agentic reasoning.

## 31. Production checklist

- [ ] Authenticated API and Gateway
- [ ] Explicit authorization
- [ ] No Gateway bypass
- [ ] Least-privilege IAM
- [ ] Cross-account trust reviewed
- [ ] PII minimized
- [ ] Tool output validation
- [ ] Prompt-injection controls
- [ ] Mutation confirmation
- [ ] Idempotency
- [ ] Rate limits and tool-call budgets
- [ ] Audit
- [ ] Distributed tracing
- [ ] Agent evaluation gates
- [ ] Multi-region/DR plan
- [ ] Incident runbooks

## 32. Final mental model

~~~text
API boundary
  -> Can this request enter?

Reasoning boundary
  -> What should the agent do?

Capability governance boundary
  -> Which tools may be invoked?

Business authority boundary
  -> Is this operation valid and authorized?

Data authority boundary
  -> What is the current truth?
~~~

The model decides what to ask for. Authoritative systems decide what is true and what is allowed.

## References

AWS: Build a multi-account AI agent with AgentCore Gateway and MCP:
https://aws.amazon.com/blogs/machine-learning/build-a-multi-account-ai-agent-with-agentcore-gateway-and-mcp/

See diagrams.md for the architecture views and implementation.md for Java, MCP, IAM, security, resilience and CI/CD patterns.
