# 22 - Coding Standards

## General
- TypeScript `strict: true`, `noUncheckedIndexedAccess`, no `any` (use `unknown` + Zod).
- ESLint + Prettier + commit hooks (lint-staged); Conventional Commits; PRs need 1 review, CI green.
- Files kebab-case, types/classes PascalCase, functions/vars camelCase, constants UPPER_SNAKE, DB fields camelCase, enums UPPER_SNAKE strings.
- Feature-based folders; no giant `utils`. Shared contracts in `packages/shared`.
- Prefer pure functions in domain layer; inject dependencies (repo, clock, gateway) for testing.
- Use `Clock` abstraction; never call `Date.now()` in domain logic.

## Backend
- Controller: `validate(schema) -> service.method(ctx,dto) -> res`.
- Service signature takes `ctx {userId, role, requestId}`.
- Errors: throw `AppError`; no `res.status` in services.
- Mongoose: lean() for reads, explicit projections, never return documents with sensitive fields; `select:false` on secrets.
- Transactions via `withTransaction(session => ...)` helper with retry on TransientTransactionError.
- Money: `Money.fromBdt(…)`, `Money.add`, `Money.percent(rounding)`.
- Logging: Pino with redaction paths (password, token, otp, nid*, authorization).
- Config: `env.ts` Zod-validated at start; fail fast.

## Frontend
- App Router; server components for public listing pages; client components for interactive parts.
- TanStack Query keys factory per module (`listingKeys.list(filters)`), `staleTime` tuned, optimistic updates only for non-money actions.
- Redux Toolkit slices: `auth` (user snapshot), `ui`, `checkoutDraft`. No server data in Redux.
- Forms: React Hook Form + Zod resolver.
- API client: single typed fetch wrapper, handles 401 refresh once, normalises errors to codes, i18n messages by error code.
- Accessibility: semantic HTML, focus management, aria labels, contrast, keyboard nav.
- Performance: next/image with CDN loader, route-level code splitting, skeletons, avoid layout shift.
- i18n: all strings in message files (bn, en); no hard-coded text; Bangla numerals via formatter.

## Error code convention
`<DOMAIN>_<REASON>` upper snake. Frontend maps code to localised message.

## Git
`main` protected; `develop` optional; feature branches `feat/<module>-<short>`; squash merge; release tags semver.

## Documentation
Public functions with TSDoc where non-obvious; ADRs in `docs/adr/` for major decisions; update requirement file when behaviour changes (same PR).
