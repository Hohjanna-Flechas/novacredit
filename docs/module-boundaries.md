# NovaCredit - Module Boundaries

## Purpose

This document defines which NovaCredit architectural boundaries are represented as physical software modules.

The goal is to keep the codebase understandable and modular without introducing unnecessary layers or abstractions.

---

## Backend Modules

The NovaCredit backend will initially contain four primary modules:

```text
backend/app/
├── domain/
├── application/
├── infrastructure/
└── api/
```

Each module owns a distinct responsibility.

---

## Domain

The `domain` module represents NovaCredit's core business concepts and rules.

It may contain:

- entities,
- value objects,
- domain rules,
- state transition rules,
- domain errors,
- domain-specific calculations.

The domain must remain independent from frameworks, persistence technologies, external APIs, AI providers, and transport mechanisms.

---

## Application

The `application` module coordinates NovaCredit use cases and workflows.

It may contain:

- use cases,
- workflow orchestration,
- application-level contracts,
- ports required from infrastructure,
- coordination between domain capabilities.

The application layer may depend on the domain.

It must not depend on concrete infrastructure implementations.

---

## Infrastructure

The `infrastructure` module contains technical implementations that connect NovaCredit to external systems or runtime technologies.

Examples include:

- database repositories,
- external AI providers,
- machine learning risk implementations,
- dataset adapters,
- event publishers,
- configuration loaders.

Infrastructure may implement contracts defined by inner layers.

Infrastructure must not define NovaCredit business behavior.

---

## API

The `api` module exposes NovaCredit capabilities through HTTP.

FastAPI will belong to this boundary.

It is responsible for:

- routes,
- HTTP request and response schemas,
- transport validation,
- authentication boundaries,
- HTTP error translation.

API handlers invoke application use cases.

Business rules must not be implemented in API handlers.

---

## Frontend

The frontend is an independent application:

```text
frontend/
```

It will use React, Vite, and TypeScript.

The frontend communicates with NovaCredit through the API and must not duplicate or own core business rules.

---

## Configuration

Runtime configuration files may live outside application code:

```text
config/
```

Examples may include:

- application settings,
- risk thresholds,
- policy configuration.

Configuration loading and validation belong to the backend infrastructure boundary.

Secrets must not be stored in YAML configuration files.

---

## Organization Principle

NovaCredit should prefer organization around domain capabilities rather than global technical buckets.

Preferred:

```text
domain/
├── application/
├── risk/
├── review/
├── policy/
└── audit/
```

Avoid:

```text
models/
services/
utils/
helpers/
validators/
```

as global catch-all modules.

Functions, rules, validators, and calculations should remain close to the domain capability they support.

---

## Dependency Direction

```text
API ─────────────┐
                 v
           Application
                 |
                 v
               Domain

Infrastructure
      |
      | implements contracts
      v
Application / Domain boundaries
```

Concrete infrastructure must remain replaceable without forcing changes throughout the core workflow.

---

## Abstraction Principle

NovaCredit will introduce abstractions when they protect a meaningful point of change.

Examples:

- multiple risk implementations,
- multiple persistence implementations,
- external AI providers,
- policy retrieval implementations.

The project will avoid abstractions that exist only for hypothetical future complexity.

Simple code is preferred until a real variation or dependency boundary justifies additional structure.