# 01 - Product Vision

## 1. Name
ResaleMart (working name).

## 2. Problem
Used-goods trading in Bangladesh happens on Facebook groups and ad-only sites. Common harms:
- Advance/token payment (bKash/Nagad) then seller disappears.
- Fake or stolen-photo listings, fake profiles.
- Item differs from description (defective, replica, wrong condition).
- No safe payment, no dispute path, no seller accountability.
- Buyers in other districts cannot trust remote deals.

## 3. Vision
The place where buying a used item from a stranger is as safe as buying from a shop: money is held until the buyer is satisfied, every seller is identity-verified, and disputes have a fair, fast resolution.

## 4. Target users
| Persona | Need |
|---|---|
| Student buyer | Cheap books, laptops, phones; fears scams |
| Bachelor/renter | Furniture and appliances locally, hand-to-hand |
| Freelancer | Used gadgets, cameras, monitors |
| Casual seller | Sell unused items quickly, safely get paid |
| Power seller | Many listings, reputation, repeat buyers |
| Admin/moderator | Remove fraud, resolve disputes |

## 5. Value proposition (USP)
1. Escrow: buyer pays platform, seller paid only after inspection.
2. Mandatory identity verification for sellers (phone OTP + NID + selfie).
3. Inspection window and structured dispute resolution.
4. Upazila-level local search plus nationwide courier option.
5. Visible trust: verified badge, completed-sales count, ratings.

## 6. Scope
In scope (v1): web app (responsive), listings, search, chat, offers, escrow checkout, local handover, courier shipping, disputes, reviews, admin panel, notifications.
Out of scope (v1): native mobile apps, auctions, ads marketplace, crypto, international shipping, loans/instalments, AI price estimation (later).

## 7. Success metrics
| Metric | 6-month target |
|---|---|
| Registered users | 100,000 |
| Verified sellers | 15% of sellers at least |
| Completed escrow orders / month | 5,000 |
| Dispute rate | < 5% of orders |
| Confirmed fraud loss | < 0.5% of GMV |
| Median time to first message on listing | < 30 min |
| Listing page p95 load | < 2.5 s on 4G |

## 8. Principles
1. Trust over growth: never weaken verification to get users.
2. Money logic is boring, auditable and reversible.
3. Server decides everything about price, status and permission.
4. Design for scale from day one (indexes, cache, queues) without premature microservices.
5. Bangla-first UX, English supported.

## 9. Risks
| Risk | Mitigation |
|---|---|
| Regulatory (escrow licence) | Licensed partner; legal review pre-launch |
| Off-platform deals to dodge fees/escrow | Chat contact-info masking, safety nudges, fee fairness |
| Fake NID / identity rental | Selfie match, manual review, device fingerprint, limits for new sellers |
| Friendly fraud (buyer false dispute) | Evidence rules, handover OTP, dispute history scoring |
| Cold start (no listings) | City-by-city launch, seed communities |
| Courier losses | Insured courier flow, tracking, packaging proof |
