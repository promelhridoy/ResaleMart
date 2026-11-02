# 21 - Testing Strategy

## Levels
| Level | Tools | Scope |
|---|---|---|
| Unit | Vitest | Money utils, fee calc, state machine, policies, masking, reputation score, OTP logic |
| Integration | Vitest + Testcontainers (Mongo replica set, Redis) | Repos, transactions, ledger invariants, outbox, queues |
| API | Supertest | Every endpoint: auth, validation, RBAC, error codes |
| Contract | Zod schemas shared; mock gateways/couriers with recorded fixtures | Webhook payloads, courier mapping |
| E2E | Playwright | Critical journeys below |
| Load | k6 | Browse/search 10k VU, chat 2k sockets, checkout burst |
| Security | ZAP/Burp, dependency scan, manual pentest | OWASP ASVS L2 checklist |
| Chaos/Resilience | Kill worker/API mid-flow | No lost or duplicated money |

## Must-pass scenarios
1. Two buyers order the same listing simultaneously: exactly one succeeds, other gets LISTING_NOT_AVAILABLE.
2. Duplicate webhook delivered 5 times: one ledger posting.
3. Webhook with bad signature or wrong amount: no state change, alert raised.
4. Payment succeeds after order expired: auto-refund path.
5. Seller never accepts: auto-refund at 24 h, listing released.
6. Inspection timer ends: auto-complete and release; timer job runs twice: no double release.
7. Dispute opened at last second of window vs timer firing: only one wins (optimistic lock), funds never both released and frozen.
8. Ledger property test: random sequences of valid transitions always keep sum(debits)=sum(credits), no negative wallet, escrow zero in terminal states.
9. Payout double-click with same Idempotency-Key: one payout.
10. IDOR: user A cannot read/modify user B's order, NID, wallet, conversation (test every route).
11. NoSQL injection payloads (`{"$gt":""}`) rejected.
12. Rate limit and OTP lockout behave.
13. Refresh token reuse revokes family.
14. Contact masking catches Bangla-digit phone numbers and spaced digits.
15. Local handover OTP wrong 5 times locks.
16. Admin NID view always produces audit log.

## E2E journeys
Register+verify -> list item -> other user finds by upazila -> chat -> offer -> pay (sandbox) -> accept -> ship (mock courier) -> delivered -> confirm -> release -> payout -> reviews.
Dispute journey with admin refund. Local handover journey with OTP. Failed-payment journey.

## Quality gates (CI)
Lint, typecheck, unit+integration pass, coverage thresholds (domain 80%, ledger/state machine 100% branches), security scan no high, E2E smoke on PR, full E2E nightly.

## Test data
Seed script: locations, categories, 50 users across roles, 500 listings, sample orders in each state. Sandbox accounts for gateway and courier.
