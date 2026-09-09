# Architecture Decision Log

## ADR-001 — Canonical domain model between core and providers
**Decision:** Keep provider-specific request/response models behind adapters and expose a canonical assessment model to the core platform.  
**Reason:** Reduces vendor coupling and allows providers to change without forcing domain/API redesign.  
**Trade-off:** Additional mapping code and contract tests are required.

## ADR-002 — Support asynchronous completion
**Decision:** Permit the create API to return `202 Accepted` when external checks cannot complete inside the synchronous latency budget.  
**Reason:** Protects customer-facing capacity from slow or unreliable downstream systems.  
**Trade-off:** Clients need status polling or an agreed callback/event mechanism.

## ADR-003 — At-least-once events with idempotent consumers
**Decision:** Assume duplicate event delivery and require consumer idempotency.  
**Reason:** Simpler and more resilient than attempting distributed exactly-once processing across heterogeneous systems.  
**Trade-off:** Consumers must maintain deduplication/state logic where side effects are not naturally idempotent.

## ADR-004 — Correlation and decision references are first-class fields
**Decision:** Persist correlation, assessment and decision references rather than relying on log search alone.  
**Reason:** Supports audit reconstruction and operational investigation across distributed components.  
**Trade-off:** Adds metadata to persistence and event contracts.

## ADR-005 — Keep sensitive data out of general-purpose logs
**Decision:** Log identifiers, state and diagnostic metadata rather than raw protected attributes.  
**Reason:** Reduces privacy/security exposure while retaining operational traceability.  
**Trade-off:** Authorized support investigations may require a controlled path to protected source records.
