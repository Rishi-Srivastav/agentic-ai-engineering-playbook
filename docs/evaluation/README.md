# Evaluation

A production AI system needs evaluation at three levels.

## Component

Retrieval, classifier/router, tool selection, policy decisions.

## End-to-end

Final answer quality, citations, task success, refusal behavior.

## Agent trajectory

Was the sequence of actions valid? Did it use unnecessary tools? Did it violate a policy? Did it recover correctly from failure?

Store evaluation datasets as versioned artifacts. Run a regression suite on every major prompt, model, retrieval, or tool-contract change.
