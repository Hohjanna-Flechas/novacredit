# NovaCredit - Initial Data Contracts

## Purpose

This document defines the initial internal data contracts used by NovaCredit.

These contracts represent NovaCredit domain concepts independently from:

- HTTP transport,
- database schemas,
- UI frameworks,
- external datasets,
- machine learning implementations,
- and third-party integrations.

External systems must translate their data into these contracts before interacting with the core workflow.

---

# CreditApplication

Represents a credit application received by NovaCredit.

This is a product-domain contract and must not mirror the South German Credit dataset directly.

## Identity

- `application_id`
- `applicant_id`
- `created_at`

## Requested Credit

- `requested_amount`
- `credit_duration_months`
- `credit_purpose`

## Applicant Profile

- `age`
- `employment_status`
- `employment_duration`

## Financial Profile

- `existing_credit_count`
- `savings_status`
- `installment_commitments`
- `housing_status`

## Credit Context

- `credit_history`
- `guarantor_status`

## Notes

`applicant_id` is an internal synthetic identifier.

NovaCredit does not store real applicant personally identifiable information in development or demonstration environments.

The exact required/optional status and validation constraints of these fields will be defined during implementation.

---

# RiskModelInput

Represents the normalized feature representation consumed by a risk scoring implementation.

This contract is intentionally separated from `CreditApplication`.

A Risk Service or adapter is responsible for translating NovaCredit domain information into the representation expected by the selected risk model.

Conceptually:

```text
CreditApplication
        |
        v
Risk Feature Adapter
        |
        v
RiskModelInput
        |
        v
Risk Model
```

The initial implementation may use a mock representation.

A future ML implementation may introduce additional transformations, categorical mappings, or source-specific feature preparation without changing the NovaCredit application contract.

Sensitive or intentionally excluded source attributes must not be introduced through this contract.

---

# RiskAssessment

Represents the normalized output of the risk assessment capability.

The NovaCredit workflow depends on this contract rather than on a specific model implementation.

## Fields

- `application_id`
- `risk_score`
- `risk_level`
- `risk_factors`
- `assessment_source`
- `assessed_at`

## Risk Level

Initial values:

- `low`
- `medium`
- `high`

The thresholds used to determine these levels belong to configuration and must not be embedded throughout the workflow.

## Assessment Source

Identifies which implementation produced the assessment.

Examples:

- `mock`
- `ml_model`

This allows the system to remain traceable while replacing risk implementations.

---

# PolicyEvaluation

Represents the result of evaluating policies relevant to a credit review case.

## Fields

- `application_id`
- `policy_references`
- `policy_flags`
- `evaluation_source`
- `evaluated_at`

The first implementation may use deterministic rules.

Future implementations may introduce retrieval-based policy evaluation without changing the core workflow contract.

---

# ReviewCase

Represents the operational case that NovaCredit manages throughout the review lifecycle.

## Identity

- `case_id`
- `application_id`

## Assessment

- `risk_assessment`
- `policy_evaluation`
- `review_flags`

## Workflow

- `case_status`
- `created_at`
- `updated_at`

A ReviewCase exists independently from any UI or database representation.

---

# HumanReview

Represents an action performed by an authorized human reviewer.

## Fields

- `case_id`
- `reviewer_id`
- `outcome`
- `review_notes`
- `reviewed_at`

## Supported Outcomes

- `approved`
- `rejected`
- `more_information_required`

AI and ML components must not independently produce a final HumanReview outcome.

The final credit decision originates from a human action.

---

# AuditEvent

Represents a traceable event generated during the lifecycle of a review case.

## Fields

- `event_id`
- `case_id`
- `event_type`
- `actor`
- `timestamp`
- `metadata`

## Example Event Types

- `application_received`
- `application_validated`
- `case_created`
- `risk_assessment_completed`
- `policy_evaluation_completed`
- `case_ready_for_review`
- `human_review_completed`
- `case_status_changed`
- `duplicate_application_detected`
- `workflow_failed`

Audit events describe what happened in the system.

They are not a substitute for application state.

---

# Contract Boundaries

## Domain vs Transport

Domain contracts must not depend on HTTP concepts such as:

- request objects,
- status codes,
- headers,
- route parameters.

FastAPI will translate HTTP requests into NovaCredit domain contracts.

---

## Domain vs Persistence

Domain contracts must not depend directly on database table structures.

Persistence implementations are responsible for translating domain objects into storage representations.

---

## Domain vs External Data

External datasets and providers do not define NovaCredit contracts.

Adapters translate external representations into internal representations.

---

## Domain vs Machine Learning

The NovaCredit workflow depends on `RiskAssessment`.

It does not depend on:

- model libraries,
- training datasets,
- feature encodings,
- prediction array formats,
- or a particular ML algorithm.

---

# Data Principles

1. Contracts represent business concepts, not provider-specific schemas.
2. Components communicate through explicit and stable contracts.
3. External representations must be translated at system boundaries.
4. Monetary values must use exact numeric representations during implementation.
5. Dates and timestamps must use explicit and consistent timezone-aware formats.
6. Sensitive or intentionally excluded historical attributes must not enter the risk workflow accidentally.
7. Training targets must never become runtime prediction inputs.
8. Contracts should evolve through intentional versioning rather than ad-hoc field additions.