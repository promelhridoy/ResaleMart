# 24 - UI/UX Specification

## Principles
Trust-first, mobile-first, Bangla-first, fast on weak networks, clear money states, minimal steps.

## Design tokens
Modern dark/light themes (theme toggle, system default). Accent teal/cyan, neutral slate surfaces, subtle glass cards on dark theme (use sparingly for performance and contrast). Radius 12-16 px, spacing 4 px grid. Fonts: Hind Siliguri / Noto Sans Bengali + Inter. Status colours: success green, warning amber, danger red, info blue; never colour alone.

## Public pages
Home (search bar, location selector, categories, nearby, fresh listings, trust strip "Escrow protected"), Category/search results (filters drawer on mobile, map toggle later), Listing detail (gallery, price, condition, seller trust card, location area, delivery options, Buy with Escrow, Chat, Make offer, safety tips, report), Seller profile, How it works/Buyer protection, Help/FAQ, Terms, Privacy.

## Auth pages
Register (phone first), OTP, Login, Forgot password, Verification wizard (consent -> NID -> selfie -> review status).

## User app
Post listing wizard (category -> details -> photos with guidance -> price/location -> delivery -> preview), My listings, Favourites, Chats (list + thread with listing and order card), Offers, Orders (buyer/seller tabs), Order detail (timeline, countdown timers, actions by state, evidence upload, OTP card), Checkout (address/handover, price breakdown, pay), Dispute screens, Wallet (balance, pending, transactions, payout), Payout accounts, Reviews, Notifications, Settings (profile, security, sessions, notification prefs, language).

## Admin panel
Dashboard (KPIs, queues), Verifications (side-by-side NID/selfie viewer, reasons), Listings moderation, Reports, Disputes (case view: timeline, evidence, chat, decision form), Orders, Finance (ledger explorer, refunds, payouts, reconciliation), Users (risk score, history), Config, Blacklist, Audit logs, Content/Categories/Locations.

## Key component behaviours
- Order timeline stepper with current state, next action, deadline countdown.
- Money breakdown component always shows item, delivery, protection fee, total.
- Escrow explainer tooltip: "Seller gets paid only after you confirm."
- Empty/loading/error states for every list; skeletons; offline banner.
- Image uploader with compression client side, progress, reorder, guidance for good photos.
- Confirm dialogs for irreversible actions (confirm received, release).
- Accessibility AA, tap targets >= 44 px, forms with inline errors.

## Content and tone
Simple Bangla, short sentences, explicit warnings against advance payment. Error messages by code with next step.

## Analytics events
view_listing, start_chat, make_offer, begin_checkout, payment_success, order_confirmed, dispute_opened, verification_submitted (no PII).
