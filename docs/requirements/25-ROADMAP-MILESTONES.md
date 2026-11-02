# 25 - Roadmap and Milestones

Estimates assume a small team (2-4 devs) with AI agents; adjust. Each milestone ends with demo + acceptance tests.

| M | Scope | Acceptance |
|---|---|---|
| M0 Foundations (wk 1) | Monorepo, CI, docker-compose, env validation, logging, error framework, shared package, DB replica set, seed locations/categories | `pnpm dev` runs all; CI green; locations queryable |
| M1 Auth & Users (wk 2) | Register/OTP/login/refresh/logout, sessions, RBAC middleware, profile, rate limits | Auth tests incl. token reuse, lockout |
| M2 Verification (wk 3) | Upload (presigned), NID encryption, admin review queue, badges, blacklist checks | Seller can reach L2; audit logs on NID view |
| M3 Listings & Media (wk 4-5) | CRUD, image pipeline, pHash, moderation, expiry, favourites | Listing lifecycle tests; images EXIF-stripped |
| M4 Search & Location (wk 5-6) | Filters, cascading location, geo radius, caching, SEO pages | p95 search < 300 ms on 100k listings seed |
| M5 Chat & Offers (wk 6-7) | Socket.IO chat, masking, offers, notifications in-app | Masking tests; multi-instance socket test |
| M6 Orders & State machine (wk 8) | Order domain, history, timers, outbox, listing reservation | Concurrency test passes (scenario 1) |
| M7 Payments & Ledger (wk 9-10) | Gateway sandbox integration, webhooks, ledger, refunds, wallet | Ledger property tests; webhook idempotency |
| M8 Delivery (wk 11-12) | Local handover OTP, 1 courier adapter, tracking sync | E2E local + courier journeys |
| M9 Disputes & Reviews (wk 13) | Dispute flow, admin arbitration UI, returns, reviews | Dispute E2E, SLA timers |
| M10 Payouts & Finance (wk 14) | Payout accounts, holds, admin approvals, reconciliation | Payout safety tests, dual control |
| M11 Admin & Trust (wk 15) | Reports, risk scoring rules, analytics, audit viewer | Fraud rules trigger tests |
| M12 Hardening & Launch prep (wk 16-18) | Load test, pentest, backups drill, monitoring, legal docs, closed beta | Go-live checklist signed |

## Go-live checklist
Legal: escrow structure approved, ToS, privacy policy, refund policy, prohibited items, NID consent text. Technical: pentest fixed, load test targets met, backup restore tested, alerts live, runbooks. Operations: support team + dispute SOP trained, fraud review rota, SMS/courier/gateway production credentials, customer support channels. Pilot: single city, invite-only, order caps, daily reconciliation, review after 4 weeks.

## Post-launch backlog
Mobile apps, storefronts, promoted listings, price suggestions, AI image checks, more couriers, safe-meetup partner points, wallet top-up, referral system, seller analytics, Bangla voice search.
