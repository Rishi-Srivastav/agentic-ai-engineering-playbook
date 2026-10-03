# Human-in-the-Loop

Human-in-the-loop (HITL) introduces an explicit approval boundary before an agent performs an action that is material, irreversible, regulated, financially significant, security-sensitive, privacy-sensitive, or externally visible.

The important distinction is that the human approves a **structured action**, not merely an LLM-generated explanation.

## Architecture

~~~mermaid
sequenceDiagram
    participant U as User
    participant A as Agent
    participant P as Policy Engine
    participant H as Human Approver
    participant T as Tool / Business API
    participant AU as Audit Store

    U->>A: Request
    A->>P: Proposed action + exact parameters
    P-->>A: Risk / policy decision
    A->>H: Approval request
    Note over H: Show action, parameters, impact and policy checks
    H-->>A: Approve / Reject
    A->>AU: Record decision and evidence
    alt Approved
        A->>T: Execute authorized action
        T-->>A: Result
    else Rejected
        A-->>U: Action not executed
    end
~~~

## What belongs in the approval record

Capture:

- authenticated actor/user
- proposed action
- exact tool and parameters
- affected resource
- policy checks and risk classification
- approver identity
- approval timestamp
- expiration, if applicable
- execution result
- correlation/request ID

## Design rules

1. **Do not approve opaque text.** Render the exact structured operation.
2. **Re-authorize at execution time.** Approval should not bypass normal authorization.
3. **Bind approval to the exact action.** Changing parameters should require a new approval.
4. **Expire stale approvals.** Do not execute an old approval against changed state.
5. **Make rejection terminal unless explicitly re-planned.**
6. **Audit both approval and execution.**

## Example

For an airline agent, changing a seat may be low risk while issuing a large refund or cancelling a complex itinerary may require approval. The classification should come from explicit business policy, not from the model's opinion.
