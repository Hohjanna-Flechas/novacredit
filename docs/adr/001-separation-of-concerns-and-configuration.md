# ADR-001: Separation of Concerns and Configuration Strategy

## Status

Accepted

## Context

NovaCredit is intended to evolve from a small software project into a modular AI-assisted application containing multiple concerns, including:

- domain logic,
- workflow orchestration,
- HTTP transport,
- persistence,
- machine learning capabilities,
- AI integrations,
- configuration,
- frontend presentation,
- automated testing,
- and observability.

The project must remain maintainable as new capabilities are introduced.

A new scenario should not require duplicating architecture or introducing a new system layer when the existing domain model can support it.

NovaCredit should also allow technical implementations such as databases, AI providers, or risk models to change independently from core business behavior.

---

## Decision

NovaCredit will use a modular architecture with explicit separation of concerns.

The primary backend boundaries are:

```text
API
 |
 v
Application
 |
 v
Domain
```

Infrastructure provides technical implementations required by inner layers.

The frontend remains an independent application communicating with the backend through explicit API contracts.

---

## Domain Independence

Core business logic must remain independent from:

- FastAPI,
- React,
- PostgreSQL,
- Docker,
- machine learning frameworks,
- LLM providers,
- external datasets,
- and third-party services.

The domain represents NovaCredit concepts and rules rather than implementation technologies.

---

## Object-Oriented Design

NovaCredit will use pragmatic object-oriented design.

OOP will be preferred when modeling:

- identity,
- state,
- behavior,
- contracts,
- replaceable implementations,
- and domain concepts.

Composition will be preferred over unnecessary inheritance.

Interfaces or protocols will be introduced when they protect a meaningful point of change.

Examples include:

```text
RiskService
├── MockRiskService
└── MLRiskService
```

and future implementations such as:

```text
AIAnalysisService
├── AnthropicAnalysisService
└── AlternativeProviderAnalysisService
```

The core workflow should depend on the abstraction rather than the concrete implementation.

---

## Functions

Simple functions may be used for stateless transformations, calculations, normalization, or validation when introducing a class provides no meaningful benefit.

Functions must belong to a clear domain or infrastructure concern.

NovaCredit will avoid global catch-all modules such as:

```text
utils/
helpers/
misc/
```

unless a genuinely cross-cutting responsibility emerges.

A calculation related to risk, for example, should remain within the risk capability rather than becoming an orphan utility.

---

## Configuration Strategy

Configuration will remain separate from business behavior.

External configuration may define values such as:

- thresholds,
- feature flags,
- retry limits,
- provider selection,
- and operational settings.

YAML may be used when human-readable configuration provides a clear benefit.

Conceptually:

```text
YAML configuration
       |
       v
Configuration Loader
       |
       v
Validated Configuration Object
       |
       v
Application / Infrastructure
```

Configuration must be validated before entering the rest of the system.

---

## Configuration vs Business Logic

Configuration provides parameters.

Code owns behavior.

For example:

```text
risk threshold = configuration
```

while:

```text
determining whether a case requires review = domain behavior
```

NovaCredit will avoid creating a configuration language that duplicates application logic.

---

## Secrets

Secrets must not be stored in YAML or committed to the repository.

Examples include:

- API keys,
- database passwords,
- authentication secrets,
- private credentials.

Secrets will be supplied through environment variables or an appropriate secret-management mechanism.

The repository may provide an `.env.example` file containing variable names but never real credentials.

---

## Dependency Principle

Outer layers may depend on inner layers.

Inner layers must not depend on outer implementation details.

For example:

```text
Application
     |
     v
RiskService contract
     ^
     |
MLRiskService implementation
```

The application workflow should not know which machine learning library, provider, database, or external service implements the capability.

---

## Scalability Principle

NovaCredit will scale through stable domain concepts, composition, configuration, and replaceable implementations rather than scenario-specific architectural layers.

The system should not evolve into structures such as:

```text
scenario_1/
scenario_2/
scenario_3/
```

when those scenarios can be represented through existing concepts and rules.

---

## Simplicity Principle

NovaCredit will not introduce abstractions solely for hypothetical future needs.

An abstraction should protect:

- a real dependency boundary,
- an expected implementation variation,
- testability,
- or meaningful business behavior.

Simple implementations are preferred until additional complexity is justified.

---

## Consequences

### Positive

This decision should provide:

- clear ownership of responsibilities,
- easier onboarding for new contributors,
- independently testable components,
- replaceable integrations,
- reduced framework coupling,
- easier evolution of AI and ML capabilities,
- and improved maintainability.

### Trade-offs

This architecture introduces more explicit boundaries than a small monolithic script.

Developers must understand where behavior belongs and maintain clear contracts between components.

Some mapping code will be required between:

- API and domain representations,
- domain and persistence,
- external providers and internal contracts.

This duplication is intentional when it protects the core domain from external implementation details.

---

## Guiding Rule

When deciding where new code belongs, ask:

> What responsibility owns this behavior, and what should be allowed to change without forcing this code to change?

If that answer is unclear, the architectural boundary should be reviewed before implementation.