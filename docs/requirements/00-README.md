# ResaleMart - Requirements Pack (A to Z)

Secured C2C used-product marketplace for Bangladesh. Escrow payment, NID verification, Division > District > Upazila location, local meetup + courier delivery.

## Reading order
| # | File | Purpose |
|---|------|---------|
| 01 | PRODUCT-VISION | Why, who, success metrics |
| 02 | PRD | Features, user stories, MVP scope |
| 03 | SRS | Functional + non-functional requirements (numbered IDs) |
| 04 | BUSINESS-RULES | Fees, limits, timers, policies |
| 05 | USER-ROLES-RBAC | Roles and permission matrix |
| 06 | ESCROW-ORDER-LIFECYCLE | Order state machine |
| 07 | SDD | System design, tech stack, folder layout |
| 08 | ARD | Architecture rules (hard rules for code) |
| 09 | DATABASE-DESIGN | MongoDB collections, indexes |
| 10 | API-SPECIFICATION | REST endpoints |
| 11 | REALTIME-EVENTS | Socket.IO rooms/events |
| 12 | SECURITY | Threat model and controls |
| 13 | NOTIFICATION-SPEC | In-app, SMS, email, push |
| 14 | PAYMENT-ESCROW-LEDGER | Gateway, ledger, payouts, refunds |
| 15 | DISPUTE-RESOLUTION | Dispute flow and admin arbitration |
| 16 | NID-TRUST-SAFETY | Verification, fraud scoring, blacklist |
| 17 | LOCATION-SPEC | Division/District/Upazila + geo search |
| 18 | DELIVERY-COURIER | Local handover + courier integration |
| 19 | CHAT-SPEC | Buyer-seller chat and offers |
| 20 | SEARCH-DISCOVERY | Listing search, filters, ranking |
| 21 | TESTING-STRATEGY | Test levels, concurrency, security tests |
| 22 | CODING-STANDARDS | TypeScript, naming, errors |
| 23 | DEVOPS-DEPLOYMENT | Docker, CI/CD, environments, backups |
| 24 | UI-UX-SPEC | Pages, components, design tokens |
| 25 | ROADMAP-MILESTONES | Phases and acceptance criteria |
| 26 | AGENTS | Rules for AI coding agents |

## Conventions
- Requirement IDs: `FR-<MODULE>-NNN`, `NFR-NNN`, `BR-NNN`, `ARD-NNN`, `SEC-NNN`.
- MUST / SHOULD / MAY follow RFC 2119.
- If two files conflict: 04-BUSINESS-RULES > 03-SRS > others. Raise the conflict, do not guess.

## Important open items (need a real-world decision, not code)
1. **Holding customer money**: operating an escrow wallet in Bangladesh may require a Bangladesh Bank payment-service licence, or a licensed partner (bank/MFS/PSP) holding the funds. Decide with a lawyer BEFORE launch. See 14-PAYMENT-ESCROW-LEDGER section 1.
2. **NID data**: collecting NID images is sensitive personal data. Decide data-protection compliance and retention with a lawyer. See 16-NID-TRUST-SAFETY.
3. **Prohibited items list** must be legally reviewed (see 04-BUSINESS-RULES).
