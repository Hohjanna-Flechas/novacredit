# NovaCredit - Data Source

## Purpose

This document defines the external data source used to support NovaCredit's risk assessment capability and establishes how external data is allowed to enter the NovaCredit domain.

The external dataset does not define NovaCredit's internal domain model.

---

## Selected Dataset

NovaCredit will initially use the **South German Credit** dataset from the UCI Machine Learning Repository.

### Source Information

- Dataset: South German Credit
- UCI Repository ID: 573
- Instances: 1,000
- Predictor variables: 20
- Target: credit risk
- Distribution: 700 good credit cases and 300 bad credit cases
- Missing values: none
- License: Creative Commons Attribution 4.0 International (CC BY 4.0)
- DOI: 10.24432/C5QG88

The dataset contains historical credit records collected between 1973 and 1975.

---

## Why This Dataset

The South German Credit dataset was selected because it provides a publicly licensed credit-risk dataset with attributes that map naturally to the type of information handled by NovaCredit.

It also provides a corrected representation of the widely used Statlog German Credit dataset, whose original coding documentation contains known issues.

The dataset is suitable for learning, experimentation, testing, and development of a demonstration risk-scoring capability.

It is not intended to represent modern lending populations or production lending behavior.

---

## Role Inside NovaCredit

The dataset is not the runtime application database.

Its initial role is to support:

- development of the risk assessment capability,
- experimentation with risk models,
- generation of test fixtures,
- validation of mapping logic,
- and creation of realistic synthetic application scenarios.

Runtime NovaCredit applications will use synthetic applicant records.

The distinction is:

```text
Public Dataset
      |
      v
Risk Model Development
      |
      v
Risk Assessment Capability


Synthetic Application
      |
      v
NovaCredit API
      |
      v
Credit Review Workflow
```

---

## Programmatic Access

The dataset can be retrieved programmatically from the UCI Machine Learning Repository.

The ingestion implementation will be introduced later and must remain outside the core NovaCredit domain.

Conceptually:

```text
UCI Data Source
      |
      v
External Data Adapter
      |
      v
NovaCredit Representation
```

The rest of the system must not depend directly on UCI-specific column names, encodings, or retrieval mechanisms.

---

## Source Variables

The dataset includes concepts such as:

- checking account status,
- credit duration,
- credit history,
- credit purpose,
- credit amount,
- savings,
- employment duration,
- installment rate,
- other debtors or guarantors,
- residence duration,
- property,
- age,
- other installment plans,
- housing,
- number of existing credits,
- job information,
- number of financially dependent people,
- telephone information,
- foreign worker status,
- and credit risk outcome.

These external fields will not automatically become NovaCredit domain fields.

A mapping decision must be made before a source attribute is introduced into the domain.

---

## Mapping Strategy

NovaCredit will use an adapter boundary between external data and internal domain contracts.

```text
South German Credit Record
          |
          v
SouthGermanCreditAdapter
          |
          v
NovaCredit Domain Contract
          |
          v
Risk Assessment Workflow
```

This allows NovaCredit to change its external data source without requiring changes throughout the workflow.

The adapter is responsible for translating source-specific names, categories, and representations into concepts understood by NovaCredit.

---

## Responsible Feature Selection

Some attributes in the historical dataset contain demographic or historically contextual information.

NovaCredit will not automatically use every available field as a risk-assessment input.

Attributes must be reviewed for:

- relevance to the business domain,
- potential fairness implications,
- historical context,
- explainability,
- and whether they are necessary for the demonstration use case.

Source availability alone is not sufficient justification for using an attribute.

---

## Data Privacy

NovaCredit will not use private customer datasets or real applicant information.

Development and demonstrations will rely on:

- publicly licensed datasets,
- synthetic applicant records,
- and synthetic or publicly usable policy documents.

No personally identifiable information from real credit applicants will be stored in the project repository.

---

## Architectural Principle

External data sources are integrations.

They do not own the NovaCredit domain.

The NovaCredit workflow depends on stable internal contracts, while adapters translate external representations into those contracts.

This allows data providers, risk implementations, and other integrations to evolve independently from the core application.