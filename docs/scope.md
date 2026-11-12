# NovaCredit - Scope and Boundaries

## In Scope

NovaCredit will provide a software workflow for preparing synthetic credit applications for human review.

The system will support:

- receiving credit applications,
- validating application data,
- maintaining application and review state,
- generating a risk assessment,
- evaluating relevant lending policies,
- preparing a structured case assessment,
- presenting cases to a human reviewer,
- recording the reviewer's final decision,
- maintaining an audit trail of relevant actions and state changes,
- handling duplicate requests and recoverable failures safely.

Future iterations may introduce AI-assisted analysis, policy retrieval, ML-based risk scoring, asynchronous processing, and external integrations.

## System Boundary

NovaCredit is responsible for the lifecycle of a credit review case from application intake through human review and audit recording.

NovaCredit is not responsible for:

- originating real loans,
- transferring money,
- managing customer bank accounts,
- performing identity verification,
- accessing credit bureaus,
- collecting real personal or financial information,
- making autonomous lending decisions,
- replacing a financial institution's core banking systems.

## Human Decision Boundary

AI and ML components may provide assessments, risk indicators, retrieved information, and structured recommendations.

They must not independently transition a case into a final approved or rejected credit decision.

A human reviewer remains responsible for the final decision.

## Data Boundary

Development and demonstration environments will use:

- synthetic applicant records,
- publicly licensed datasets when model development requires them,
- synthetic or publicly usable policy documents.

Private client datasets and real applicant information are outside the project scope.

## Engineering Boundary

NovaCredit will be designed as a modular full-stack software system.

Major concerns should remain independently maintainable, including:

- presentation,
- API transport,
- domain logic,
- workflow orchestration,
- persistence,
- configuration,
- external integrations,
- AI and ML capabilities,
- observability,
- automated testing.

New scenarios should preferably be supported through existing domain concepts, configuration, or replaceable implementations rather than scenario-specific architectural layers.