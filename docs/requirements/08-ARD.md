# 08 - Architecture Rules (ARD)

Hard rules. Violations fail code review.

## Layering
- ARD-001 Controllers are thin: parse request, call service, shape response. No business logic, no DB calls.
- ARD-002 Business rules live in services/domain/policy functions.
- ARD-003 Only repositories talk to Mongoose models. Cross-module access goes through the other module's public service interface, never its models.
- ARD-004 Modules may not import each other's internals; use `index.ts` public API or events.

## Validation and errors
- ARD-010 Every route validates params, query, body with Zod; unknown fields stripped.
- ARD-011 All errors are `AppError(code, httpStatus, message, details?)`; global handler formats `{ error: { code, message, details, requestId } }`.
- ARD-012 Never leak stack traces or internal ids in production responses.

## Data integrity
- ARD-020 Multi-document changes (order+listing+ledger+history) MUST use a Mongo transaction.
- ARD-021 State changes use optimistic concurrency (`version`) or conditional `findOneAndUpdate` with expected status.
- ARD-022 No float for money. Use integer poisha and `Money` utilities.
- ARD-023 Ledger entries are append-only: no update, no delete. Corrections are new compensating entries.
- ARD-024 Timestamps stored UTC; created/updated on every collection.
- ARD-025 Soft-delete for user content; hard-delete only via privacy-erasure job that respects legal retention.

## Security
- ARD-030 Authorization is checked in policy functions in the service layer for every operation, not only in routes.
- ARD-031 Client never supplies price, fee, status, role, ownerId, or balance fields. They are derived server side.
- ARD-032 Secrets only from environment/secret manager; never committed; typed env validation at boot.
- ARD-033 NID and sensitive fields are encrypted at field level; keys outside DB.
- ARD-034 Rate limit all public and auth endpoints; stricter on OTP, login, payment, chat start.
- ARD-035 Webhooks verify signature and replay window, then are processed idempotently.

## Async work
- ARD-040 Side effects (notifications, courier calls, payouts) run via outbox + queue, never inline in request path.
- ARD-041 Jobs are idempotent and re-check state before acting.
- ARD-042 Failed jobs retry with exponential backoff, then dead-letter queue with alert.

## API
- ARD-050 Versioned `/api/v1`; breaking changes require `/v2`.
- ARD-051 List endpoints use cursor pagination with max limit.
- ARD-052 Mutations accept `Idempotency-Key` where money or inventory is involved.
- ARD-053 Responses never include sensitive fields (passwordHash, nid*, tokens).

## Frontend
- ARD-060 Server state in TanStack Query only; Redux Toolkit only for auth session snapshot, UI state, and checkout draft.
- ARD-061 No tokens in localStorage/sessionStorage. Cookies only.
- ARD-062 Forms validated with shared Zod schemas from `packages/shared`.
- ARD-063 All user-generated text rendered escaped; any HTML sanitized via DOMPurify allowlist.

## Observability
- ARD-070 Every request has `requestId`; propagated to logs and jobs.
- ARD-071 Log structured JSON; never log passwords, OTPs, tokens, NID, full card/wallet numbers.
- ARD-072 Every money or admin action emits an audit event.
