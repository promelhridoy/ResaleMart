# 10 - REST API Specification

Base: `/api/v1`. JSON. Auth by HttpOnly cookie (web) or `Authorization: Bearer` (future mobile).
Success: `{ "data": ..., "meta": { "nextCursor": "..." } }`
Error: `{ "error": { "code": "ORDER_INVALID_STATE", "message": "...", "details": {}, "requestId": "..." } }`
Headers: `Idempotency-Key` on marked endpoints. Rate limits per section 12.

## Auth
| Method | Path | Auth | Notes |
|---|---|---|---|
| POST | /auth/register | - | phone, name, password |
| POST | /auth/otp/send | - | purpose: REGISTER/LOGIN/RESET |
| POST | /auth/otp/verify | - | |
| POST | /auth/login | - | sets cookies |
| POST | /auth/refresh | cookie | rotates |
| POST | /auth/logout | user | |
| POST | /auth/logout-all | user | |
| POST | /auth/password/forgot, /reset | - | |
| GET | /auth/sessions, DELETE /auth/sessions/:id | user | |

## Users
GET /users/me, PATCH /users/me, GET /users/:id/public (profile, rating, badges), GET /users/:id/listings, GET /users/:id/reviews, POST /users/:id/block, DELETE /users/:id/block.

## Verification
POST /verification/upload-urls (presigned PUT for nid-front/back/selfie), POST /verification (submit), GET /verification/me.
Admin: GET /admin/verifications?status=PENDING, GET /admin/verifications/:id (signed URLs, audited), POST /admin/verifications/:id/approve, POST /admin/verifications/:id/reject.

## Catalog & Location
GET /categories (tree), GET /categories/:id/attributes, GET /locations/divisions, GET /locations/districts?divisionId=, GET /locations/upazilas?districtId=, GET /locations/search?q=.

## Listings
| Method | Path | Notes |
|---|---|---|
| GET | /listings | search: q, categoryId, minPrice, maxPrice, condition[], divisionId, districtId, upazilaId, lat, lng, radiusKm, delivery, verifiedOnly, sort, cursor, limit |
| GET | /listings/:id | public detail (increments view async) |
| POST | /listings/upload-urls | presigned image upload |
| POST | /listings | create (L2) |
| PATCH | /listings/:id | owner; blocked fields when order open |
| POST | /listings/:id/pause, /resume, /renew, /mark-sold | owner |
| DELETE | /listings/:id | owner soft delete |
| POST | /listings/:id/favorite, DELETE | user |
| GET | /me/listings?status= | owner |
| GET | /me/favorites | |
| POST/GET/DELETE | /me/saved-searches | |

## Conversations, Messages, Offers
POST /listings/:id/conversations (get or create), GET /conversations, GET /conversations/:id/messages?cursor=, POST /conversations/:id/messages, POST /conversations/:id/read, POST /conversations/:id/offers, POST /offers/:id/accept, /reject, /counter.

## Orders
| Method | Path | Notes |
|---|---|---|
| POST | /orders | body: listingId, offerId?, deliveryMethod, address?; Idempotency-Key; returns order + payment redirect URL |
| GET | /orders/:id | party or staff |
| GET | /me/orders?role=buyer|seller&status= | |
| POST | /orders/:id/cancel | buyer (pre-accept) |
| POST | /orders/:id/accept | seller |
| POST | /orders/:id/reject | seller |
| POST | /orders/:id/cancel-by-seller | seller, reason |
| POST | /orders/:id/shipment | seller: courier, packing proof keys; creates parcel |
| POST | /orders/:id/meetup/propose, /meetup/accept, /meetup/checkin | local |
| POST | /orders/:id/handover/confirm | OTP |
| POST | /orders/:id/confirm-received | buyer, ends inspection OK |
| POST | /orders/:id/dispute | buyer, see below |
| GET | /orders/:id/timeline | status history |

## Payments
POST /payments/init (internal via order), GET /payments/:id, POST /webhooks/sslcommerz, POST /webhooks/bkash, POST /webhooks/courier/:provider (signature verified, no cookie auth), GET /payments/return/success|fail|cancel (browser redirect, never trusted for state).

## Wallet & Payouts
GET /me/wallet (balance, pending), GET /me/wallet/transactions, POST/GET /me/payout-accounts, POST /me/payouts (Idempotency-Key), GET /me/payouts.

## Disputes
POST /orders/:id/dispute {reason, description, evidenceKeys[]}, GET /disputes/:id, POST /disputes/:id/respond (seller), POST /disputes/:id/messages, POST /disputes/:id/return-shipment (buyer).
Admin: GET /admin/disputes?status=, POST /admin/disputes/:id/decision {type, refundPct, reasonText}, POST /admin/disputes/:id/request-info.

## Reviews & Reports
POST /orders/:id/reviews, GET /users/:id/reviews, PATCH /reviews/:id (48h), POST /reports.

## Notifications
GET /notifications, POST /notifications/:id/read, POST /notifications/read-all, PATCH /me/notification-preferences, POST /me/push-subscriptions.

## Admin
GET /admin/dashboard, GET/PATCH /admin/users/:id, POST /admin/users/:id/ban|unban|suspend, GET /admin/listings?status=, POST /admin/listings/:id/approve|reject|remove, GET /admin/reports, POST /admin/reports/:id/action, GET /admin/orders, POST /admin/orders/:id/force-state (SUPER_ADMIN, audited), GET /admin/ledger/transactions, GET /admin/payouts, POST /admin/payouts/:id/approve|reject, POST /admin/refunds (dual control), GET/PATCH /admin/config, GET /admin/audit-logs, POST /admin/blacklist, GET /admin/analytics/*.

## Error codes (sample)
AUTH_INVALID_CREDENTIALS, AUTH_OTP_EXPIRED, AUTH_LOCKED, FORBIDDEN, VALIDATION_FAILED, NOT_FOUND, VERIFICATION_REQUIRED, LISTING_NOT_AVAILABLE, LISTING_LIMIT_REACHED, ORDER_INVALID_STATE, ORDER_ALREADY_EXISTS, PAYMENT_SIGNATURE_INVALID, PAYMENT_AMOUNT_MISMATCH, IDEMPOTENCY_CONFLICT, RATE_LIMITED, DISPUTE_WINDOW_CLOSED, PAYOUT_NOT_ALLOWED, BLACKLISTED, UPLOAD_INVALID.

## Rate limits (per IP + user)
Login 10/15 min; OTP send 5/hour per phone; register 5/hour/IP; listing create 20/day; chat start 50/day; message 60/min; search 120/min; order create 10/hour; payout request 5/day; default 300/min.
