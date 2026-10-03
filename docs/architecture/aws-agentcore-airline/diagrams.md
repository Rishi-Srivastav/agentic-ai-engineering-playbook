
# Airline AgentCore Architecture Diagrams

These diagrams focus on trust boundaries, identity, failure isolation and mutation safety.

## 1. Executive architecture

~~~mermaid
flowchart LR
 U["Airline Client / Internal Consumer"] --> EDGE["API Gateway + WAF"]
 EDGE --> API["Spring Boot Airline AI API"]
 API --> AG["AgentCore Runtime"]
 AG --> GW["AgentCore Gateway"]
 GW --> R["Reservation"]
 GW --> F["Flight Operations"]
 GW --> C["Customer / Loyalty"]
 GW --> S["Seats / Ancillaries"]
 GW --> B["Baggage"]
 GW --> D["Disruption"]
 AG --> M["Amazon Bedrock"]
~~~

## 2. Trust boundaries

~~~mermaid
flowchart LR
 subgraph USER["User Trust Boundary"]
   U["Customer / Employee"]
   IDP["OIDC / Enterprise IdP"]
 end
 subgraph PLATFORM["AI Platform AWS Account"]
   EDGE["API Gateway + WAF"]
   API["Spring Boot API"]
   AGENT["AgentCore Runtime"]
   GATEWAY["AgentCore Gateway"]
   POLICY["Authorization / Policy"]
 end
 subgraph RES["Reservation AWS Account"]
   RMCP["Reservation MCP / OpenAPI Target"]
   RS["Reservation Services"]
   RDB[("Reservation DB")]
 end
 subgraph OPS["Flight Ops AWS Account"]
   OMCP["Flight Ops MCP / OpenAPI Target"]
   OS["Flight Operations Services"]
   ODB[("Flight Ops DB")]
 end
 U --> IDP
 U --> EDGE
 IDP --> EDGE
 EDGE --> API
 API --> AGENT
 AGENT --> GATEWAY
 GATEWAY --> POLICY
 POLICY --> RMCP
 RMCP --> RS
 RS --> RDB
 POLICY --> OMCP
 OMCP --> OS
 OS --> ODB
~~~

## 3. Request sequence

~~~mermaid
sequenceDiagram
 autonumber
 participant C as Customer
 participant A as Spring Boot API
 participant R as AgentCore Runtime
 participant G as AgentCore Gateway
 participant F as Flight Ops
 participant B as Reservation
 participant L as Baggage
 C->>A: Natural-language request
 A->>R: Authenticated request
 R->>G: get_flight_status
 G->>F: Authorized tool call
 F-->>G: Flight status
 G-->>R: Flight status
 R->>G: get_booking
 G->>B: Authorized tool call
 B-->>G: Booking summary
 G-->>R: Booking summary
 R->>G: get_baggage_allowance
 G->>L: Authorized tool call
 L-->>G: Allowance
 G-->>R: Grounded result
 R-->>A: Response
 A-->>C: Response
~~~

## 4. Identity propagation

~~~mermaid
sequenceDiagram
 participant U as User
 participant IDP as Identity Provider
 participant API as Spring Boot API
 participant R as Agent Runtime
 participant G as Gateway
 participant D as Domain Target
 U->>IDP: Authenticate
 IDP-->>U: Access token
 U->>API: Request + token
 API->>API: Validate issuer/audience/scopes
 API->>R: Trusted subject context
 R->>G: Tool invocation
 G->>G: Authenticate + authorize
 G->>D: User-bound / delegated context
 D->>D: Business authorization
 D-->>G: Result
 G-->>R: Result
 R-->>API: Response
 API-->>U: Response
~~~

## 5. Read versus write

~~~mermaid
flowchart TD
 Q["User Request"] --> R["Agent"]
 R --> READ{"Read?"}
 READ -->|Yes| RT["Read tool"]
 RT --> AUTH1["Authorization"]
 AUTH1 --> DATA["Authoritative data"]
 DATA --> R
 READ -->|No| WRITE["Mutation tool"]
 WRITE --> AUTH2["Authorization"]
 AUTH2 --> CONF{"Confirmation required?"}
 CONF -->|Yes| USER["Explicit confirmation"]
 CONF -->|No| IDEM["Idempotency check"]
 USER --> IDEM
 IDEM --> EXEC["Execute mutation"]
 EXEC --> AUDIT["Audit event"]
 AUDIT --> R
~~~

## 6. Gateway bypass prevention

~~~mermaid
flowchart LR
 A["Agent"] --> G["AgentCore Gateway"]
 G --> T["Authorized Tool Target"]
 T --> D["Domain Service"]
 A -. "Forbidden direct path" .-> D
 X["IAM / Network / Resource Policies"] -. "blocks" .-> D
 X -. "enforces" .-> T
~~~

## 7. Failure isolation

~~~mermaid
flowchart TB
 A["Agent"] --> G["Gateway"]
 G --> R["Reservation"]
 G --> F["Flight Ops"]
 G --> B["Baggage"]
 G --> S["Seats"]
 F --> X["Flight Ops degraded"]
 R --> R2["Reservation healthy"]
 B --> B2["Baggage healthy"]
 S --> S2["Seat inventory healthy"]
 X -. "isolated failure" .-> G
~~~

## 8. Mutation safety state machine

~~~mermaid
stateDiagram-v2
 [*] --> GatherFacts
 GatherFacts --> PresentOptions
 PresentOptions --> AwaitConfirmation
 AwaitConfirmation --> Execute: User confirms
 AwaitConfirmation --> [*]: User declines
 Execute --> IdempotencyCheck
 IdempotencyCheck --> ExistingResult: Key exists
 IdempotencyCheck --> Transaction: New key
 Transaction --> Audit
 Audit --> Success
 ExistingResult --> Success
 Success --> [*]
~~~

## 9. Deployment pipeline

~~~mermaid
flowchart LR
 DEV["Developer"] --> TEST["Unit + Contract Tests"]
 TEST --> SEC["Security + IaC"]
 SEC --> EVAL["Agent Evaluation"]
 EVAL --> INT["Integration"]
 INT --> CAN["Canary"]
 CAN --> PROD["Production"]
 PROD --> OBS["Observe"]
 OBS --> RB["Rollback"]
 RB --> CAN
~~~

## 10. Tool onboarding

~~~mermaid
flowchart TD
 TEAM["Domain Team"] --> SPEC["Tool Specification"]
 SPEC --> OWNER["Owner + SLO"]
 OWNER --> AUTH["Authorization Review"]
 AUTH --> PII["PII Classification"]
 PII --> IDEM["Mutation / Idempotency"]
 IDEM --> SEC["Security Review"]
 SEC --> EVAL["Agent Evaluation"]
 EVAL --> REG["Gateway Registration"]
 REG --> CAN["Canary"]
 CAN --> PROD["Production"]
~~~

## 11. Data movement

~~~mermaid
flowchart LR
 U["User"] --> API["API"]
 API --> AG["Agent"]
 AG --> GW["Gateway"]
 GW --> T1["Tool Target"]
 T1 --> D1["Domain Service"]
 D1 --> DB1[("Authoritative DB")]
 DB1 --> D1
 D1 --> T1
 T1 --> GW
 GW --> AG
 AG --> API
 API --> U
 D1 -. "metadata only" .-> OBS["Observability"]
~~~

The model should receive the minimum useful representation of domain data, not unrestricted database records.

## 12. Production reference

~~~mermaid
flowchart TB
 subgraph EDGE["Enterprise Edge"]
   WAF["WAF"]
   APIGW["API Gateway"]
   IDP["Enterprise IdP"]
 end
 subgraph AI["AI Platform Account"]
   API["Spring Boot API"]
   RT["AgentCore Runtime"]
   GW["AgentCore Gateway"]
   EVAL["Evaluation"]
   OBS["Observability"]
 end
 subgraph DOM["Domain Accounts"]
   R["Reservation"]
   F["Flight Operations"]
   C["Customer / Loyalty"]
   S["Seats / Ancillaries"]
   B["Baggage"]
   D["Disruption"]
 end
 subgraph SEC["Security / Governance"]
   IAM["IAM / SCP"]
   KMS["KMS / Secrets"]
   SIEM["Security / Audit"]
 end
 IDP --> APIGW
 WAF --> APIGW
 APIGW --> API
 API --> RT
 RT --> GW
 GW --> R
 GW --> F
 GW --> C
 GW --> S
 GW --> B
 GW --> D
 IAM -.-> GW
 IAM -.-> R
 KMS -.-> AI
 SIEM -.-> OBS
 RT --> EVAL
~~~

## Diagram principle

This is deliberately not a single "LLM talks to everything" box. Each boundary has an owner, authorization decision and failure mode.
