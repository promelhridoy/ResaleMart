# 13 - Notification Specification

## Channels
In-app (always), Web push (opt-in), SMS (critical only, cost-controlled), Email (receipts, summaries; optional since phone-first).

## Templates (bn + en, variables via ICU)
| Event | Buyer | Seller | Channels |
|---|---|---|---|
| Order created, awaiting payment | reminder at 10 min | - | in-app |
| Funds held | "Payment secured" | "New order - accept within 24h" | in-app, SMS(seller), push |
| Seller accepted | yes | - | in-app, push |
| Seller rejected/expired | refund started | - | in-app, SMS |
| Shipped with tracking | yes | - | in-app, push |
| Courier delivered / inspection started | "Inspect within 24h" + countdown | - | in-app, SMS, push |
| Inspection ending soon (6h, 1h) | yes | - | in-app, push |
| Handover OTP ready | OTP shown in app | meetup details | in-app |
| Order completed/released | - | "Funds added to wallet" | in-app, SMS |
| Dispute opened | - | "Respond within 48h" | in-app, SMS, email |
| Dispute decision | yes | yes | in-app, SMS |
| Refund processed | yes | - | in-app, SMS |
| Payout success/fail | - | yes | in-app, SMS |
| New message (offline) | yes | yes | push, digest email |
| Offer received/accepted | yes | yes | in-app, push |
| Verification approved/rejected | - | yes | in-app, SMS |
| Listing approved/rejected/expiring | - | yes | in-app |
| Saved-search match | yes | - | push, in-app (daily cap) |
| Security: new device login, password change | yes | yes | SMS/in-app |

## Rules
- NTF-001 Notifications are created from outbox events; delivery by BullMQ with retries.
- NTF-002 User preferences per category; critical security/financial cannot be disabled.
- NTF-003 SMS budget cap per day with alert; SMS template IDs approved by gateway.
- NTF-004 Dedupe key per (user,event,entity) within 5 min.
- NTF-005 Quiet hours (22:00-08:00 Dhaka) for non-critical push.
- NTF-006 Never include OTP/handover codes in push or email body when avoidable; SMS OTP only for auth.
