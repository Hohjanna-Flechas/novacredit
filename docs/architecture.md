# NovaCredit - Software Architecture

## Purpose

This document defines the software architecture of NovaCredit, including system responsibilities, dependency boundaries, communication patterns, and deployment boundaries.

The architecture should allow the system to evolve without coupling core business behavior to frameworks, external providers, infrastructure, or presentation technologies.

---

## Architectural Goal

NovaCredit should be:

- maintainable,
- modular,
- testable,
- understandable by new contributors,
- configurable where appropriate,
- and extensible without introducing scenario-specific architectural layers.

The architecture should favor stable domain concepts and explicit contracts over framework-specific implementations.

---

## Core Dependency Rule

Dependencies should point toward the NovaCredit domain.

The core business rules must not depend directly on:

- FastAPI,
- React,
- PostgreSQL,
- Docker,
- Anthropic or other LLM providers,
- machine learning libraries,
- external datasets,
- or third-party APIs.

Conceptually:

```text
                    External World
                         |
       +-----------------+-----------------+
       |                 |                 |
       v                 v                 v
   Frontend           FastAPI        Integrations
                                          |
                                          v
                                   Infrastructure
                                          |
                                          v
                                    Application
                                          |
                                          v
                                       Domain
```

The domain sits at the center of the system.

Outer layers may depend on inner layers.

Inner layers must not depend on outer layers.

---

## Architectural Layers

### Domain

Owns NovaCredit's core business concepts and rules.

Examples:

- CreditApplication
- ReviewCase
- RiskAssessment
- HumanReview
- case state transitions
- domain validation
- business rules

The domain must remain independent from frameworks and infrastructure.

---

### Application

Coordinates use cases and workflows.

Examples:

- submit application,
- prepare review case,
- request risk assessment,
- evaluate policies,
- record human review,
- recover a failed workflow.

The application layer orchestrates domain behavior but does not own infrastructure implementations.

---

### Infrastructure

Provides technical implementations required by the application.

Examples:

- database repositories,
- ML risk service implementation,
- external AI providers,
- policy retrieval implementation,
- event publishing,
- configuration loading.

Infrastructure implements contracts required by the application or domain.

---

### API

Translates HTTP interactions into application use cases.

FastAPI belongs here.

Responsibilities include:

- HTTP request parsing,
- transport-level validation,
- authentication boundary,
- response serialization,
- HTTP status handling.

Business rules must not live in API endpoints.

---

### Frontend

Provides the reviewer-facing user interface.

React, Vite, and TypeScript belong here.

The frontend may:

- display application and case information,
- request actions through the API,
- display workflow state,
- present errors and results.

The frontend must not define NovaCredit business rules or directly mutate business state.

---

## Dependency Direction

```text
Frontend
   |
   v
API
   |
   v
Application
   |
   v
Domain
```

Infrastructure connects through abstractions rather than controlling the workflow:

```text
Application
     |
     | depends on contract
     v
RiskService
     ^
     |
     | implements
     |
MLRiskService
Infrastructure
```

This allows implementations to be replaced without changing the core workflow.

---

## Primary Principle

NovaCredit architecture should be organized around stable system responsibilities rather than individual scenarios.

New scenarios should preferably be supported through:

- existing domain concepts,
- configuration,
- composition,
- or replaceable implementations.

A new scenario should not automatically require a new architectural layer.