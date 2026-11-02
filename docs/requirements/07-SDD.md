# 07 - Software Design Document

## 1. Architecture style
Modular monolith + background workers + real-time gateway. Split into services only when a module's scale demands it (candidates: chat, search).

```
Browser -> CDN -> Next.js (SSR/ISR) -> API (Express) -> MongoDB (replica set)
                                       |-> Redis (cache, rate limit, socket adapter, BullMQ)
                                       |-> Worker (BullMQ): timers, payouts, notifications, courier sync
                                       |-> Socket.IO gateway (same codebase, Redis adapter)
                                       |-> Object storage (S3 compatible / Cloudinary) public + private buckets
                                       |-> External: payment gateway, SMS, courier APIs, email
Search: MongoDB text/Atlas Search in v1; OpenSearch/Meilisearch optional later.
```

## 2. Tech stack
| Layer | Choice |
|---|---|
| Frontend | Next.js (App Router), TypeScript strict, Tailwind CSS, shadcn/ui or HeroUI, Redux Toolkit (client/global UI state), TanStack Query (server state), React Hook Form + Zod, next-intl (bn/en) |
| Backend | Node.js LTS, Express, TypeScript strict, Zod validation, Pino logging |
| DB | MongoDB 7 replica set, Mongoose |
| Cache/Queue | Redis, BullMQ |
| Realtime | Socket.IO + Redis adapter |
| Auth | JWT access + rotating refresh, Argon2id, HttpOnly cookies |
| Storage | S3-compatible (private bucket for NID/disputes, public via CDN for listings) |
| Payments | SSLCommerz or licensed aggregator; bKash/Nagad via it or direct |
| Courier | Steadfast, RedX, Pathao (adapter pattern) |
| SMS | Local gateway adapter |
| Observability | Pino + OpenTelemetry, Sentry, Prometheus/Grafana or cloud equivalent |
| CI/CD | GitHub Actions, Docker |
| Tests | Vitest, Supertest, Playwright, k6 |

## 3. Repository layout (monorepo)
```
resalemart/
  apps/
    web/                 # Next.js
    api/                 # Express API + socket gateway
    worker/              # BullMQ workers
  packages/
    shared/              # zod schemas, types, constants, error codes
    config/              # eslint, tsconfig
  docs/requirements/     # this pack
  docker-compose.yml
```

### API module layout
```
apps/api/src/
  config/ (env.ts, db.ts, redis.ts)
  modules/
    auth/  users/  verification/  listings/  categories/  locations/
    search/  chat/  offers/  orders/  payments/  ledger/  payouts/
    delivery/  disputes/  reviews/  notifications/  reports/  admin/
      each: routes.ts controller.ts service.ts policy.ts repo.ts schema.ts dto.ts events.ts
  middleware/ (auth, rbac, rateLimit, validate, error, requestId)
  integrations/ (gateway/, courier/, sms/, storage/)
  shared/ (errors, logger, money, idempotency, audit)
  server.ts  socket.ts
```

## 4. Key design decisions
1. **Money as integer poisha**, `Money` helper, no float math.
2. **Ledger-first**: balances derived from entries; cached balance fields are projections.
3. **State machine in domain layer** (`orders/order.machine.ts`), controllers only call `service.transition()`.
4. **Outbox pattern**: events written in same transaction, dispatched by worker (notifications, sockets, courier jobs).
5. **Idempotency keys** on order creation, payment init, payout request, and all webhooks.
6. **Adapters** for gateway, courier, SMS so providers can be swapped.
7. **CQRS-lite**: search uses denormalised listing documents; writes go to canonical collections.
8. **Config in DB** (`SystemConfig`) cached in Redis for fees/timers.

## 5. Critical flows (sequence summaries)
**Checkout**: Buyer POST /orders (idempotency key) -> tx: lock listing(reserve), create order PENDING_PAYMENT, create payment intent -> redirect to gateway -> gateway webhook -> verify signature, validate amount -> tx: payment SUCCESS, ledger entries, order FUNDS_HELD, outbox events -> notifications.

**Release**: Timer or buyer confirm -> tx: order COMPLETED -> worker creates ledger release entries (escrow -> seller wallet, escrow -> platform fee) -> RELEASED -> payout eligibility.

**Payout**: Seller requests -> checks (verified payout account, balance, holds) -> payout record PENDING -> worker calls gateway disbursement/or manual admin batch -> SUCCESS/FAILED with ledger entries.

## 6. Scaling plan
- Stateless API behind LB, >=2 instances; sticky not needed (Redis adapter).
- Read-heavy listing and category endpoints cached in Redis (30-120 s) + CDN for images.
- Mongo indexes per 09; read preference secondary for search/analytics where acceptable.
- Queue concurrency tuned per job type; separate queues: critical (payments), default, bulk (emails).
- Later: shard by listing location? Not planned; vertical + replica scaling first.
