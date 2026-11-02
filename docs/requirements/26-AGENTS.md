# 26 - Guide for AI Coding Agents

## Prime directives
1. Read the relevant requirement files BEFORE coding. Do not invent behaviour; if a rule is missing or conflicts, stop and ask or record an assumption in `docs/assumptions.md`.
2. Work in small vertical slices (one module/endpoint at a time) with tests.
3. Obey 08-ARD. Never place business logic in controllers; never access another module's models.
4. Money = integer poisha; ledger append-only; transitions only via the order state machine.
5. Never trust client input for price, status, role, owner, balance.
6. Never log or return secrets, tokens, OTPs, NID data.
7. Never commit secrets, `.env`, or real credentials. Use `.env.example`.
8. Every endpoint needs: Zod schema, policy check, error codes, tests (happy, validation, forbidden, not-found).
9. Every state-changing money/order action needs idempotency and a transaction.
10. Update docs when behaviour changes.

## Workflow per task
1. Identify requirement IDs (FR/BR/ARD/SEC) the task covers.
2. Write/adjust shared Zod schema and types.
3. Write failing tests.
4. Implement repo -> service -> policy -> controller -> route.
5. Run lint, typecheck, tests; fix.
6. Summarise: files changed, requirement IDs satisfied, tests added, open questions.

## Suggested agent roles
- Planner: breaks milestone into tasks citing requirement IDs.
- Backend agent: modules per 07-SDD layout.
- Frontend agent: pages per 24-UI-UX-SPEC with TanStack Query + RTK rules.
- Security reviewer: checks 12-SECURITY list on every PR (IDOR, injection, secrets, webhooks).
- Test agent: scenarios in 21-TESTING-STRATEGY.
- Finance reviewer: verifies ledger invariants for any payment-touching change.

## Definition of done
Typecheck, lint, tests pass; no TODO for security; docs updated; migration + rollback considered; metrics/logs added for new flows.

## Forbidden
Floats for money; direct ledger updates; deleting audit/ledger rows; client-side authorization only; storing tokens in localStorage; disabling validation to "make it work"; skipping tests for payment, state machine, or auth code; real NID/PII in fixtures or logs.

## Starter prompt for first task
"Read docs/requirements/00-README.md, 07-SDD.md, 08-ARD.md, 22-CODING-STANDARDS.md. Implement milestone M0: monorepo scaffold (pnpm workspaces), env validation, Pino logger, AppError + global handler, health endpoints, docker-compose with Mongo replica set and Redis, CI workflow. Add tests. Report assumptions."
