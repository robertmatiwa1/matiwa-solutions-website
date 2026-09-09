# Engineering Technical Specification

## Purpose

Define implementable requirements for a regulated verification/risk-assessment service that integrates external providers and preserves an auditable decision trail.

## Components

### Verification API
- Exposes versioned REST endpoints under `/v1/assessments`.
- Validates request schema and mandatory fields.
- Requires authenticated callers.
- Generates/propagates `X-Correlation-Id`.
- Supports `Idempotency-Key` on create operations.

### Orchestration Service
- Owns assessment state transitions.
- Calls provider adapters using bounded timeouts.
- Executes configurable decision rules.
- Persists state before publishing completion events.
- Must not log raw identity documents or secrets.

### Provider Adapters
- Map canonical domain requests to provider-specific contracts.
- Normalize provider responses into the canonical result model.
- Apply timeout, retry and circuit-breaker policy per provider.
- Keep provider credentials outside application source/configuration repositories.

### Event Bus
Representative events:
- `AssessmentCreated`
- `ProviderCheckCompleted`
- `AssessmentDecisioned`
- `AssessmentFailed`

Each event must include `eventId`, `correlationId`, `assessmentId`, `occurredAt`, `schemaVersion` and event-specific payload.

## State model

`RECEIVED -> PROCESSING -> APPROVED | REFERRED | DECLINED | FAILED`

Transitions must be persisted and timestamped. Invalid transitions return an explicit domain error and must not mutate state.

## Security requirements

1. OAuth 2.0/OIDC for client authentication and authorization.
2. TLS for all service and provider traffic.
3. Least-privilege workload identities.
4. Secrets stored in an approved secrets-management facility.
5. Sensitive fields masked from application and gateway logs.
6. Authorization decisions logged using non-sensitive identifiers.
7. Administrative/configuration changes auditable.

## Reliability requirements

- External calls have explicit connection and response timeouts.
- Retries apply only to retry-safe failures and use exponential backoff with jitter.
- Circuit breakers protect the platform from persistently failing providers.
- Consumers are idempotent.
- Poison messages are moved to a dead-letter path with operational alerting.
- The API can return `202 Accepted` for assessments continuing asynchronously.

## Observability

Metrics should include request rate, latency percentiles, error rate, provider latency/error rate, queue lag, decision outcomes and circuit-breaker state. Distributed traces must propagate the correlation ID across gateway, API, orchestration, provider adapter and event consumers.

## Acceptance criteria

- Duplicate create requests with the same idempotency key do not create duplicate assessments.
- Unauthorized callers receive an authentication/authorization failure without protected data leakage.
- A provider timeout does not exhaust API worker capacity.
- Every terminal decision can be traced to its request, ruleset version and provider references.
- Sensitive request fields do not appear in standard application logs.
- Event consumers can safely process duplicate delivery.
- Health/readiness checks distinguish application health from dependency degradation.

## Delivery artefacts

Engineering receives:
- OpenAPI contract;
- event schemas;
- sequence/state flows;
- NFR and security requirements;
- provider mapping specifications;
- architecture decision records;
- production-readiness checklist.
