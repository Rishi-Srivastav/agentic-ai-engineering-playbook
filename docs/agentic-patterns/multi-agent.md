# Multi-Agent Systems

A multi-agent architecture separates responsibilities across specialized agents instead of giving one agent every tool, instruction and context source.

Use it when specialization, isolation, independent evaluation, or different security boundaries provide a measurable benefit. Do not split an application into agents simply because multiple agents are possible.

## Preferred topology

~~~mermaid
flowchart TD
    U[User Request] --> S[Supervisor / Orchestrator]
    S --> R{Route by capability}
    R --> A[Reservation Agent]
    R --> F[Flight Operations Agent]
    R --> C[Customer / Loyalty Agent]
    R --> B[Baggage Agent]
    A --> RA[Structured Result]
    F --> RF[Structured Result]
    C --> RC[Structured Result]
    B --> RB[Structured Result]
    RA --> S
    RF --> S
    RC --> S
    RB --> S
    S --> V[Validate / Synthesize]
    V --> U
~~~

## Why specialization helps

A specialist can have:

- a smaller tool catalogue
- narrower instructions
- domain-specific retrieval
- domain-specific authorization
- a smaller context window
- independent evaluation metrics
- a clear owner and SLO

## Coordination contract

| Contract | Example |
|---|---|
| Role | Flight disruption specialist |
| Inputs | flight number, date, user context |
| Allowed tools | flight status, reaccommodation search |
| Output schema | typed JSON result |
| Timeout | bounded per request |
| Termination | result, refusal, or explicit failure |
| Owner | Flight Operations platform/team |

## Avoid unconstrained peer-to-peer agents

Prefer a supervisor delegating to a specialist and receiving a structured result over unconstrained peer-to-peer conversations.

The supervisor should control delegation, deadlines, retries and final synthesis.
