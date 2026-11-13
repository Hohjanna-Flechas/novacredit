# NovaCredit - Communication and Deployment Boundaries

## Purpose

This document defines how NovaCredit components communicate and which responsibilities may become independent deployment units.

Deployment boundaries describe runtime isolation.

They do not automatically imply separate services from the first implementation.

---

## Initial System View

```text
React Frontend
      |
      | HTTPS / JSON
      v
FastAPI Backend
      |
      +-------------------+
      |                   |
      v                   v
Persistence          Risk Capability
      |
      v
PostgreSQL

FastAPI Backend
      |
      | HTTPS
      v
External AI Provider
```

The initial implementation should remain as simple as possible while preserving clear boundaries.

---

## Frontend Boundary

Technology:

- React
- Vite
- TypeScript

Responsibilities:

- reviewer-facing presentation,
- user interaction,
- navigation,
- form handling,
- API consumption,
- display of workflow state and errors.

The frontend communicates only through explicit backend API contracts.

The frontend must not directly access the database, ML model, or external AI provider.

---

## Backend Boundary

Technology:

- Python
- FastAPI

Responsibilities:

- exposing HTTP capabilities,
- invoking application use cases,
- coordinating workflow execution,
- enforcing domain rules,
- interacting with infrastructure through contracts,
- translating system outcomes into HTTP responses.

FastAPI is a delivery mechanism.

It does not own NovaCredit business rules.

---

## Persistence Boundary

Planned technology:

- PostgreSQL

Responsibilities may include storing:

- credit applications,
- review cases,
- workflow state,
- human reviews,
- audit events,
- selected assessment results.

Persistence stores NovaCredit state but does not define business behavior.

The domain must not depend directly on PostgreSQL-specific representations.

---

## Risk Capability Boundary

Risk assessment is a replaceable capability.

Initial implementation:

```text
RiskService
     |
     v
MockRiskService
```

Future implementation:

```text
RiskService
     |
     v
MLRiskService
```

The rest of NovaCredit depends on the `RiskAssessment` contract rather than on the underlying model technology.

The risk capability may initially execute inside the backend process.

It should only become a separately deployed service if operational or scaling requirements justify that separation.

---

## AI Capability Boundary

Future AI-assisted analysis will use an external model provider through a controlled integration boundary.

Conceptually:

```text
Application Workflow
       |
       v
AIAnalysisService
       |
       v
Anthropic Adapter
       |
       | HTTPS
       v
Claude API
```

Claude is an external service and is not hosted inside NovaCredit infrastructure.

The application must not depend directly on provider-specific response formats.

Provider responses must be translated into NovaCredit-controlled contracts.

---

## Policy Capability

Policy evaluation may evolve through multiple implementations.

Initial:

```text
PolicyService
      |
      v
DeterministicPolicyService
```

Future:

```text
PolicyService
      |
      v
RAGPolicyService
```

The workflow should remain independent from how policy information is retrieved or evaluated.

---

## Communication Patterns

### Frontend to Backend

```text
HTTP / HTTPS
JSON
REST-style endpoints
```

Examples may eventually include:

```text
POST /applications
GET /applications/{id}
GET /cases
GET /cases/{id}
POST /cases/{id}/review
```

These routes are illustrative and are not yet finalized API contracts.

---

### Backend to Database

The backend will access persistence through repository contracts.

Conceptually:

```text
Application
      |
      v
CaseRepository
      ^
      |
PostgresCaseRepository
```

---

### Backend to External AI

Communication occurs over HTTPS through an infrastructure adapter.

Failures such as:

- timeouts,
- rate limits,
- invalid responses,
- provider errors,

must be translated into NovaCredit-controlled behavior.

---

## Synchronous vs Asynchronous Processing

NovaCredit will begin with synchronous processing where practical.

A queue, worker system, or asynchronous job architecture will not be introduced until a real need exists.

Potential future triggers include:

- long-running AI workflows,
- expensive risk processing,
- background document processing,
- retryable integrations,
- workload spikes.

Conceptually:

```text
FastAPI
   |
   v
Queue
   |
   v
Worker
   |
   v
Risk / AI Processing
```

This is a future option, not an initial requirement.

---

## Docker Strategy

Docker will be introduced after executable application components exist.

Potential container boundaries:

```text
Docker Compose
|
+-- frontend
|
+-- backend
|
+-- postgres
|
+-- optional future worker
|
+-- optional future locally hosted services
```

External hosted services such as Claude are not containerized by NovaCredit.

---

## Developer Experience Goal

The long-term local development goal is:

```text
git clone
    |
    v
configure .env
    |
    v
docker compose up
    |
    v
NovaCredit running
```

This represents a target developer experience, not the current implementation state.

---

## Deployment Principle

A conceptual module does not automatically require its own deployment unit.

NovaCredit will separate services operationally only when there is a concrete reason such as:

- independent scaling,
- different runtime dependencies,
- fault isolation,
- security isolation,
- long-running workloads,
- or independent deployment requirements.

Deployment complexity must be justified by operational value.

---

## Observability Boundary

Observability will be designed around business and technical signals.

Future instrumentation may include:

- request logs,
- workflow state transitions,
- processing duration,
- integration failures,
- retry attempts,
- duplicate request detection,
- human review events,
- AI response validation failures.

Monitoring implementation will be introduced later, but important workflow events should remain observable by design.

---

## Initial Deployment Decision

For the first implementation milestone:

- frontend and backend remain separate application boundaries,
- persistence will be introduced as a dedicated database dependency,
- risk assessment may run inside the backend process,
- AI capabilities remain external integrations,
- no queue or worker architecture is required yet,
- Docker remains planned rather than implemented.

This keeps the system simple while preserving clear paths for future evolution.