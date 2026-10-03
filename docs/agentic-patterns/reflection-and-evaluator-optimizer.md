# Reflection and Evaluator–Optimizer

These patterns add a critique or evaluation stage after generation.

They are useful when quality can be measured against explicit criteria such as correctness, policy compliance, groundedness, formatting, or task success.

## Basic loop

~~~mermaid
flowchart LR
    I[Task / context] --> G[Generate]
    G --> E[Evaluate against explicit rubric]
    E --> D{Meets threshold?}
    D -->|Yes| F[Return result]
    D -->|No| R[Revise]
    R --> G
~~~

## Reflection vs evaluator–optimizer

**Reflection** usually means the same agent critiques and improves its own output.

**Evaluator–optimizer** makes the evaluator an explicit component, which can be another model, deterministic validator, or a hybrid.

~~~mermaid
flowchart TD
    X[Input] --> O[Optimizer / Generator]
    O --> A[Artifact]
    A --> V[Evaluator]
    V --> Q{Pass?}
    Q -->|Yes| OUT[Approved artifact]
    Q -->|No| FB[Structured feedback]
    FB --> O
~~~

## Prefer deterministic criteria

A useful evaluator should check observable properties:

- schema validity
- factual/groundedness requirements
- required fields
- policy compliance
- tool-call correctness
- regression against a known evaluation set
- latency and cost budgets

Avoid relying only on vague prompts such as “critique your answer until it is excellent.”

## Production controls

- Cap revision rounds.
- Define a measurable threshold.
- Preserve initial and final artifacts for debugging.
- Detect evaluator instability.
- Version prompts, models and rubrics.
- Run offline evaluation before deployment.
- Sample production traces for human review.

For agent systems, evaluate not only the final answer but also **tool selection, arguments, trajectory length, safety policy compliance and task success**.
