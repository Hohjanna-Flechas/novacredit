# NovaCredit - External Data Mapping

## Purpose

This document defines how attributes from the South German Credit dataset may be used by NovaCredit.

The external dataset schema must not become the NovaCredit domain model directly.

Source-specific fields are translated through an adapter into model-facing representations, while the product domain remains independently defined.

---

## Mapping Categories

Each source attribute is classified as one of:

- **Model Candidate**: may be considered as an input to the demonstration risk model.
- **Domain Candidate**: represents a concept that may also exist in NovaCredit's runtime application domain.
- **Excluded**: intentionally not used by NovaCredit.
- **Training Target**: used only when training/evaluating the model and never supplied as runtime input.

---

## Source Mapping

| Source Variable | NovaCredit Concept | Classification | Initial Decision |
|---|---|---|---|
| `status` | checking account status | Model Candidate | Keep for model experimentation |
| `duration` | credit duration | Domain + Model Candidate | Keep |
| `credit_history` | credit history | Domain + Model Candidate | Keep |
| `purpose` | credit purpose | Domain + Model Candidate | Keep |
| `amount` | historical credit amount | Model Candidate | Keep only in source/model representation |
| `savings` | savings status | Model Candidate | Keep |
| `employment_duration` | employment duration category | Domain + Model Candidate | Keep as categorical/ordinal, not exact months |
| `installment_rate` | installment burden category | Model Candidate | Keep using the source representation |
| `personal_status_sex` | personal status / sex | Excluded | Do not use |
| `other_debtors` | co-debtor / guarantor status | Domain + Model Candidate | Keep |
| `present_residence` | residence duration category | Model Candidate | Keep provisionally |
| `property` | property status | Model Candidate | Keep provisionally; review fairness implications |
| `age` | applicant age | Domain Candidate | May support eligibility validation; do not initially use for risk scoring |
| `other_installment_plans` | external installment plans | Domain + Model Candidate | Keep |
| `housing` | housing status | Model Candidate | Keep provisionally |
| `number_credits` | existing credit count | Domain + Model Candidate | Keep |
| `job` | employment/job category | Model Candidate | Keep provisionally; review fairness implications |
| `people_liable` | financial dependents | Domain + Model Candidate | Keep |
| `telephone` | historical landline ownership | Excluded | Do not use |
| `foreign_worker` | foreign worker status | Excluded | Do not use |
| `credit_risk` | observed credit outcome | Training Target | Training/evaluation only |

---

## Important Separation

The South German Credit dataset is historical training data.

It must not dictate the shape of a modern NovaCredit application.

For example:

```text
South German Credit
        |
        v
Source Adapter
        |
        v
Risk Feature Representation
        |
        v
Risk Model
        |
        v
RiskAssessment
```

is separate from:

```text
Synthetic CreditApplication
        |
        v
NovaCredit Workflow
        |
        v
Risk Service
```

The Risk Service is responsible for translating relevant NovaCredit application information into whatever representation the selected risk model requires.

---

## Target Leakage Rule

`credit_risk` represents the historical outcome and is used only as a training or evaluation target.

It must never be accepted as an input to a runtime risk assessment request.

---

## Historical Data Limitations

The dataset represents credit data from the 1970s.

NovaCredit must therefore avoid treating its distributions, categorical values, monetary values, or historical relationships as representative of modern lending populations.

The dataset exists to demonstrate software integration and ML risk-scoring capabilities, not to create a production lending model.

---

## Responsible Use

Feature availability does not automatically justify feature usage.

Before a source field becomes part of the risk model, it should be reviewed for:

- relevance,
- explainability,
- historical context,
- fairness implications,
- and whether the feature is necessary for the demonstration.

Sensitive or historically inappropriate attributes should be excluded even when they are available in the source data.