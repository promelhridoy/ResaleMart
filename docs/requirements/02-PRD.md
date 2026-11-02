# 02 - Product Requirements Document (PRD)

## 1. Roles
Guest, Buyer, Seller (same account can do both), Moderator, Admin, Support.

## 2. Feature list by module
### M1 Account & Auth
- Register with phone (primary) + email optional; OTP verify.
- Login (phone/email + password), logout, refresh, forgot/reset password.
- Profile: name, avatar, default location, bio.
- Sessions list and "log out all devices".

### M2 Verification & Trust
- Levels: L0 Unverified, L1 Phone verified, L2 NID verified (required to sell), L3 Trusted (history based).
- NID front/back + selfie upload; auto checks + manual admin review.
- Verified badge, completed sales count, rating, response rate, member since.

### M3 Listings
- Create/edit/pause/delete listing: title, description, category, brand, condition, price, negotiable flag, images (min 3, max 10), optional video, location, delivery mode, warranty/invoice info, defects declaration.
- Draft, pending review, active, paused, reserved, sold, rejected, expired.
- Listing auto-expires in 60 days, renewable.
- Duplicate/stolen-image detection.

### M4 Search & Discovery
- Keyword, category tree, condition, price range, location (division/district/upazila), radius, delivery type, verified-seller-only, sort (newest, price, nearest).
- Saved searches and alerts, favourites/wishlist, recently viewed.
- Home: categories, nearby, newly listed, trending.

### M5 Chat & Offers
- Per-listing conversation buyer<->seller, real-time, image sharing.
- Make offer / counter offer / accept offer (becomes order price).
- Contact info masking (phone numbers, links) until order is paid.
- Block and report user.

### M6 Checkout & Escrow
- "Buy with Escrow" at listing price or accepted offer.
- Choose Local Handover or Courier; address; price breakdown (item + delivery + buyer protection fee).
- Pay via bKash / Nagad / card (SSLCommerz or licensed gateway).
- Funds held; seller notified; order timeline visible to both.

### M7 Delivery
- Local: meetup proposal (public place suggestion), handover OTP code.
- Courier: seller creates parcel (Steadfast/RedX/Pathao), tracking, COD not used (prepaid escrow).
- Delivery proof: packing photo/video, courier receipt.

### M8 Inspection, Confirmation, Release
- Local: buyer inspects at meetup, enters handover OTP to accept (release starts) or declines on the spot (cancel handover).
- Courier: inspection window starts at delivery; buyer confirms, disputes, or auto-confirm at window end.
- Release to seller wallet, then payout to bKash/Nagad/bank.

### M9 Disputes & Returns
- Open dispute with reason + evidence; seller response; admin decision: release, refund, partial refund; return shipping flow.

### M10 Reviews & Reputation
- Both sides review after completion; rating 1-5, tags, text; one review per order; edit within 48h.

### M11 Notifications
- In-app, push (web), SMS (critical), email.

### M12 Admin Panel
- Dashboard, user management, verification queue, listing moderation, reports queue, dispute arbitration, finance (ledger, payouts, refunds), config (fees, timers, categories, locations), audit log, blacklist.

### M13 Safety
- Report listing/user/message; blacklist (NID hash, phone, device); fraud score; safety tips.

## 3. Key user stories
- US-01 As a seller I verify my NID once so I can post listings.
- US-02 As a buyer I filter by my upazila so I find items I can collect.
- US-03 As a buyer I pay into escrow so the seller cannot take my money and vanish.
- US-04 As a seller I see "funds secured" before I ship, so I am not risking my item.
- US-05 As a buyer I raise a dispute with photos if the item is not as described.
- US-06 As a buyer I confirm local handover with a code only after I test the item.
- US-07 As an admin I see both parties' evidence in one screen and decide.
- US-08 As a seller I request payout to my bKash after release.
- US-09 As a buyer I negotiate by chat and the agreed price becomes my order price.
- US-10 As a user I report a suspicious listing and see it removed quickly.

## 4. MVP vs later
| Phase | Features |
|---|---|
| MVP | M1, M2, M3, M4 (core), M5 (chat+offers), M6, M7 (local+1 courier), M8, M9 (basic), M10, M12 (core), M11 (in-app+SMS) |
| v1.1 | Saved search alerts, wishlist, push, more couriers, seller storefront |
| v2 | Mobile apps, price suggestions, AI image checks, promoted listings, wallet top-up |

## 5. Acceptance (MVP definition of done)
A verified seller lists an item; a buyer in another district finds it, chats, pays into escrow; seller ships by courier; buyer confirms; seller receives payout; both review. A second path ends in dispute and admin refund. Every money movement appears in the ledger and balances to zero.
