# Implementation Task Template

Future implementation work must be divided into small, contract-driven, independently auditable tasks. Broad instructions such as `implement inventory` are not acceptable: split them until scope, invariants, failure semantics, and verification are explicit.

Do not activate this template for product coding while [`00_BLUEPRINT_STATUS.md`](00_BLUEPRINT_STATUS.md) says product coding is prohibited.

```text
Task ID:
Phase:
Status:

Objective:

Architecture references:

Decision references:

Evidence references:

Scope:

Contract:

Inputs:

Outputs:

Invariants:

Failure/error semantics:

Explicit exclusions:

Dependencies:

Database effects:

Concurrency requirements:

Idempotency requirements:

Security / tenant-isolation requirements:

Observability requirements:

Unit tests:

Integration tests:

Property/randomized tests:

Edge cases:

Completion criteria:

Completion report:
- files changed
- commands run
- tests run
- results
- deviations from contract
- unresolved issues
```

An implementation task is not ready if its required contract is `UNKNOWN`, its exclusions are implicit, or its completion cannot be verified by named tests/checks.
