# NovaCredit - Review Case State Machine

## Purpose

This document defines the valid lifecycle states of a NovaCredit `ReviewCase`, the transitions allowed between them, and the main failure conditions that may affect workflow execution.

The state machine represents business state.

Infrastructure errors, logs, and audit events do not replace business state.

---

## States

### RECEIVED

A credit application has been received by NovaCredit but has not yet completed validation.

### VALIDATED

The application has passed the required structural and business validation checks.

### ASSESSING

NovaCredit is preparing the case by executing the required assessment capabilities, such as risk assessment and policy evaluation.

### READY_FOR_REVIEW

The case assessment has been successfully prepared and the case is ready for a human reviewer.

### MORE_INFORMATION_REQUIRED

A human reviewer determined that additional information is required before a final decision can be recorded.

This is a non-terminal state.

### APPROVED

A human reviewer approved the credit application.

This is a terminal state.

### REJECTED

A human reviewer rejected the credit application.

This is a terminal state.

### FAILED

The workflow encountered a failure that prevented the case from progressing safely.

This state must contain enough failure context to support diagnosis and, when appropriate, recovery.

`FAILED` is not automatically terminal.

---

## Happy Path

```text
RECEIVED
    |
    v
VALIDATED
    |
    v
ASSESSING
    |
    v
READY_FOR_REVIEW
    |
    +----------------------+
    |          |           |
    v          v           v
APPROVED    REJECTED    MORE_INFORMATION_REQUIRED
```

---

## Information Recovery Path

A case requiring more information may return to the normal workflow after the missing information is provided.

```text
MORE_INFORMATION_REQUIRED
          |
          v
      VALIDATED
          |
          v
      ASSESSING
          |
          v
   READY_FOR_REVIEW
```

The workflow must not create a new credit case when the existing case can continue safely.

---

## Failure Path

Failures may occur during validation, risk assessment, policy evaluation, persistence, or external integrations.

Conceptually:

```text
RECEIVED
    |
    +---- failure ----> FAILED

VALIDATED
    |
    +---- failure ----> FAILED

ASSESSING
    |
    +---- failure ----> FAILED
```

A recoverable failure may resume from an explicitly supported state.

Recovery must never depend on guessing what completed before the failure.

---

## Invalid Transitions

Examples of invalid transitions include:

```text
RECEIVED -> APPROVED

VALIDATED -> REJECTED

ASSESSING -> APPROVED

FAILED -> APPROVED

READY_FOR_REVIEW -> VALIDATED
```

A final lending outcome must originate from an authorized human review action.

The system must reject invalid state transitions rather than silently correcting them.

---

## Terminal States

The initial terminal states are:

- `APPROVED`
- `REJECTED`

Terminal cases cannot return to active processing through the normal workflow.

Future requirements may introduce explicit reopening or appeal capabilities, but these are outside the initial scope.

---

## Duplicate Application Handling

Duplicate request detection is not represented as a separate business state.

If NovaCredit receives a duplicate request for an application that already exists, the system should identify the existing application or case and avoid creating a second independent workflow.

A corresponding audit event may be recorded:

```text
duplicate_application_detected
```

The idempotency strategy will be defined during implementation.

---

## Failure Principles

1. A failure must not leave a case in an unknown state.
2. Recoverable failures should preserve enough context to resume safely.
3. Retrying a workflow must not duplicate previously completed operations.
4. External service failures must be translated into NovaCredit-controlled failure behavior.
5. Terminal decisions cannot be produced by automated recovery.
6. Failure details used for debugging must remain separate from reviewer-facing business state.

---

## State Transition Principles

1. State transitions must be explicit.
2. Valid transitions must be centrally defined.
3. State changes must produce audit events.
4. UI components must not directly mutate case state.
5. API endpoints request business actions; the domain decides whether the resulting transition is valid.
6. Persistence stores state but does not define transition rules.
7. AI and ML components cannot directly transition a case into `APPROVED` or `REJECTED`.

---

## Conceptual Ownership

```text
Frontend
    |
    | requests action
    v
API
    |
    v
Application / Workflow Layer
    |
    v
Domain State Rules
    |
    +------ valid ------> new ReviewCase state
    |
    +------ invalid ----> domain error
```

This ensures that business state remains controlled by the NovaCredit domain rather than by presentation, transport, persistence, AI, or external services.