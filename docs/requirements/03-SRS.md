# 03 - Software Requirements Specification (SRS)

## 1. Functional requirements

### Auth (FR-AUTH)
- FR-AUTH-001 System MUST register users with unique phone number (BD format +8801XXXXXXXXX).
- FR-AUTH-002 System MUST verify phone with 6-digit OTP, 5 min expiry, max 5 attempts, resend cooldown 60 s.
- FR-AUTH-003 Passwords MUST be hashed with Argon2id; min 8 chars, not in common-password list.
- FR-AUTH-004 Access token (15 min) + rotating refresh token (30 days) in HttpOnly Secure SameSite cookies.
- FR-AUTH-005 Refresh token reuse MUST revoke the whole token family.
- FR-AUTH-006 Account lock 15 min after 5 failed logins.
- FR-AUTH-007 User MUST be able to list and revoke sessions.

### Verification (FR-VER)
- FR-VER-001 Selling, receiving payouts and creating listings require level L2.
- FR-VER-002 Submission includes NID number, NID front, NID back, live selfie.
- FR-VER-003 System MUST reject duplicate NID (via keyed hash) already linked to another account.
- FR-VER-004 Admin MUST approve or reject with reason; rejected user can resubmit max 3 times.
- FR-VER-005 NID images MUST be stored privately and viewed only via short-lived signed URL, every view audit-logged.

### Listings (FR-LST)
- FR-LST-001 Listing requires title(10-100), description(30-5000), category, condition, price(>0), >=3 images, location, delivery mode.
- FR-LST-002 New sellers' first 3 listings go to moderation (PENDING_REVIEW); trusted sellers auto-publish.
- FR-LST-003 Prohibited items MUST be blocked (keyword rules + report review).
- FR-LST-004 Images: JPEG/PNG/WebP, max 5 MB each, server re-encodes, strips EXIF GPS, generates thumbnails.
- FR-LST-005 Perceptual hash of images MUST be compared to detect reuse across accounts.
- FR-LST-006 Sold/reserved items MUST NOT be purchasable.
- FR-LST-007 Price edits on a listing with an active order MUST NOT alter that order.
- FR-LST-008 High-value categories (phone, laptop, bike, camera) require extra fields (IMEI/serial/model) and the seller to confirm ownership.

### Search (FR-SRCH)
- FR-SRCH-001 Search by text, category, price, condition, location, radius, delivery mode, verified seller.
- FR-SRCH-002 Results paginated by cursor, default 24, max 60.
- FR-SRCH-003 Nearby search uses GeoJSON 2dsphere, radius 1-50 km.
- FR-SRCH-004 Only ACTIVE listings appear in public search.

### Chat & Offers (FR-CHAT)
- FR-CHAT-001 One conversation per (listing, buyer, seller).
- FR-CHAT-002 Messages delivered in real time and persisted; offline users get notification.
- FR-CHAT-003 Phone numbers, emails, URLs and payment-app keywords MUST be masked before order payment.
- FR-CHAT-004 Offers: amount, expiry (default 24 h), states PENDING/ACCEPTED/REJECTED/COUNTERED/EXPIRED.
- FR-CHAT-005 Blocked user MUST NOT be able to message the blocker.

### Orders & Escrow (FR-ORD)
- FR-ORD-001 Order is created only from an ACTIVE listing by a verified-phone buyer who is not the seller.
- FR-ORD-002 Server computes all amounts; client-sent amounts are ignored.
- FR-ORD-003 Creating an order MUST atomically reserve the listing (one order per listing at a time).
- FR-ORD-004 Unpaid order expires after 15 min and listing is released.
- FR-ORD-005 Funds are held in escrow after verified gateway webhook (never client redirect alone).
- FR-ORD-006 Seller must accept within 24 h or order auto-cancels with full refund.
- FR-ORD-007 State changes only via the lifecycle in 06; each change writes status history.
- FR-ORD-008 Auto-release occurs when inspection window ends with no dispute.

### Payments (FR-PAY)
- FR-PAY-001 Every money movement MUST be a balanced double-entry ledger transaction.
- FR-PAY-002 Webhooks MUST be signature-verified and idempotent.
- FR-PAY-003 Refunds go back to the original payment method where gateway supports it, else to buyer wallet.
- FR-PAY-004 Seller payout to bKash/Nagad/bank only for verified payout account owned by verified user (name match).
- FR-PAY-005 Payout hold: first payout for a new seller delayed 24 h; large amounts require admin approval above configured threshold.

### Delivery (FR-DLV)
- FR-DLV-001 Listing delivery mode: LOCAL_ONLY, COURIER_ONLY, BOTH.
- FR-DLV-002 Local orders use 6-digit handover OTP shown to buyer, entered/confirmed at meetup.
- FR-DLV-003 Courier orders require tracking ID within 48 h of seller acceptance.
- FR-DLV-004 Courier status updates sync via webhook/polling.

### Disputes (FR-DSP)
- FR-DSP-001 Buyer may open dispute inside inspection window with reason and >=1 evidence file.
- FR-DSP-002 Seller has 48 h to respond; silence allows admin to rule on buyer evidence.
- FR-DSP-003 Admin decision: FULL_RELEASE, FULL_REFUND, PARTIAL_REFUND(x%), with written reason.
- FR-DSP-004 Funds stay frozen during dispute.

### Reviews (FR-REV)
- FR-REV-001 Only parties of a COMPLETED/RESOLVED order may review, once each, within 14 days.
- FR-REV-002 Reviews can be reported; admin may remove abusive ones.

### Admin (FR-ADM)
- FR-ADM-001 All admin actions MUST write immutable audit log entries.
- FR-ADM-002 Finance actions require role FINANCE or SUPER_ADMIN.
- FR-ADM-003 Admin can ban user, which cancels active listings and freezes payouts pending review.

### Notifications (FR-NTF)
- FR-NTF-001 Every order state change notifies affected party in-app; critical ones also SMS.

## 2. Non-functional requirements
| ID | Requirement |
|---|---|
| NFR-001 | Support 100,000 registered users, 10,000 concurrent browsing, 2,000 concurrent chat sockets per node |
| NFR-002 | API p95 < 300 ms for reads (cached), < 800 ms for writes |
| NFR-003 | Availability 99.5% monthly (v1), payment endpoints 99.9% target |
| NFR-004 | No single point of failure for API (>=2 instances behind load balancer) |
| NFR-005 | Daily encrypted DB backups, restore tested monthly, RPO 15 min, RTO 4 h |
| NFR-006 | All traffic HTTPS (TLS 1.2+), HSTS |
| NFR-007 | WCAG 2.1 AA for main flows; mobile-first responsive |
| NFR-008 | Bangla and English UI (i18n), BDT currency, Asia/Dhaka timezone display, UTC storage |
| NFR-009 | Pages usable on slow 3G/4G: LCP < 2.5 s p75, images lazy-loaded WebP/AVIF via CDN |
| NFR-010 | Structured logs, metrics, tracing, alerting on payment and error-rate anomalies |
| NFR-011 | Money stored as integers in poisha (1 BDT = 100 poisha); never floats |
| NFR-012 | Audit logs retained 5 years; financial records per legal requirement |
| NFR-013 | 80% unit-test coverage on domain logic; 100% of ledger and state-machine paths tested |
| NFR-014 | OWASP ASVS L2 as security baseline |

## 3. Assumptions
- A licensed payment partner holds the escrow funds, or the business holds the licence.
- SMS gateway available (e.g. local bulk SMS provider).
- NID verification is manual-assisted in v1; automated NID API integration is a later option.

## 4. Constraints
- MongoDB MUST run as a replica set (transactions required).
- Single region deployment in v1 (Singapore/Mumbai nearest cloud region or a BD host); CDN for media.
