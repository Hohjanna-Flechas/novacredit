# NovaCredit - Core Workflow

## Primary Actor

The primary actor is a credit operations analyst or reviewer responsible for evaluating a credit application and recording the final decision.

## Happy Path

A successful credit review follows this sequence:

1. A new credit application is submitted to NovaCredit.
2. The system validates the application data.
3. A credit review case is created.
4. The application is sent to the risk assessment capability.
5. The system receives a structured risk assessment.
6. Relevant policy checks are evaluated.
7. NovaCredit prepares a structured case assessment for review.
8. The case is marked as ready for human review.
9. A reviewer opens the case and examines:
   - applicant information,
   - risk assessment,
   - relevant flags,
   - applicable policy information,
   - and the case history.
10. The reviewer records one of the supported final outcomes:
    - Approved
    - Rejected
    - More Information Required
11. NovaCredit records the decision and the responsible reviewer.
12. The case transitions to its final state.
13. An audit event is recorded for the decision and relevant state changes.

## Core Workflow

```text
Credit Application
        |
        v
Application Validation
        |
        v
Case Creation
        |
        v
Risk Assessment
        |
        v
Policy Evaluation
        |
        v
Structured Case Assessment
        |
        v
Ready for Human Review
        |
        v
Human Decision
        |
        v
Final Case State
        |
        v
Audit Trail
```

## Workflow Principles

- Each stage has a clear responsibility.
- Business state changes must be explicit and traceable.
- AI and ML components may support the workflow but cannot make the final credit decision.
- The workflow should depend on stable contracts rather than specific implementations.
- A failure in one capability should not leave the case in an undefined state.
- New scenarios should reuse the same workflow concepts whenever possible instead of creating scenario-specific flows.

## Initial Risk Assessment Strategy

Risk assessment is part of the core workflow.

During the first implementation milestone, NovaCredit will use a mock risk assessment implementation behind a stable risk service contract.

This allows the full workflow to be implemented and tested before introducing a real ML-based risk model.