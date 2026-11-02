# 19 - Chat and Offers Specification

## 1. Behaviour
- Start chat from listing ("Chat with seller"); one thread per listing/buyer/seller; seller sees listing header and offer shortcut.
- Real-time via Socket.IO; REST fallback; unread badges; typing; read receipts; image attachments (max 3 per message, 5 MB each, scanned).
- Quick replies: "Is it still available?", "Last price?", "Can you share more photos?".
- System messages for order events inserted in thread.
- Block/report from thread.

## 2. Masking and safety
- Detect Bangladeshi phone patterns (01XXXXXXXXX with spaces/Bangla digits/words), emails, URLs, "bkash/nagad/rocket/send money/advance/token" keywords.
- Pre-order: replace contact info with `[hidden - use ResaleMart chat]`; show warning banner for payment keywords.
- Repeat violations increase fraud score; third violation auto-mute 24 h.
- After order PAID: allow phone sharing for meetup coordination inside order screen only (masked proxy number optional later).

## 3. Offers
- Buyer offers amount >= 50% of list price (configurable) if listing negotiable; max 5 per listing.
- Seller accepts/rejects/counters; accepted offer locks price for 24 h for that buyer; "Buy now at agreed price" button creates order with offerId.
- Server validates offer belongs to buyer, not expired, listing still ACTIVE.

## 4. Limits
Message 1,000 chars; 60 msgs/min; 50 new conversations/day; spam heuristics (same text to many users).

## 5. Retention and privacy
Messages kept 2 years (or longer if dispute); users can delete conversations from their view only; staff access only for reports/disputes, audited.

## 6. Scaling
Messages collection indexed by conversation; sockets via Redis adapter; unread counters in Redis, synced to DB; fan-out through user rooms; separate chat service possible later.
