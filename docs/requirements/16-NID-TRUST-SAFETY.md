# 16 - NID Verification, Trust and Safety

## 1. Verification flow (seller L2)
1. User consents (explicit checkbox, purpose and retention shown in Bangla).
2. Enters NID number (10/13/17 digits accepted; validate format), uploads front, back, and takes live selfie (camera capture; gallery upload disallowed for selfie where possible).
3. Server: validates images, runs checks:
   - Duplicate NID hash against other accounts and blacklist.
   - Image quality (blur, glare, crop), OCR of name/DOB/NID number vs entered data (OCR confidence logged, not trusted alone).
   - Face match selfie vs NID photo (provider/ML; score threshold; below -> manual).
   - Phone verified, account age, device fingerprint risk.
4. Admin queue: reviewer sees masked data; "reveal NID image" is audited.
5. Outcome: APPROVED (level 2), REJECTED with reason (blurry, mismatch, suspected fake), NEEDS_RESUBMIT. Max 3 attempts then support.
6. Optional later: integrate official NID verification via an authorised channel (Election Commission API through authorised partners) after legal approval.

## 2. Data handling
- NID number encrypted AES-256-GCM; lookup HMAC; images in private bucket.
- Only fields needed: number, name, DOB, images; do not store father/mother/address unless needed.
- Access: ADMIN verification role only, 2FA, logged. Never in exports, logs, analytics.
- Retention: delete images after approval + retention period (default 12 months) unless fraud case; keep hash for dedupe/blacklist.
- Right to erasure handled except hash kept for fraud prevention where legally allowed.
- Legal review required for collecting NID and for sharing with law enforcement on valid request only.

## 3. Trust signals shown publicly
Verified badge (NID), member since, completed sales, rating, response time, dispute-free %, seller level (Bronze/Silver/Gold), "ID verified" tooltip explaining. Never show NID details.

## 4. Fraud scoring (risk engine v1: rules, v2: ML)
Score 0-100 per user/listing/order; thresholds: <30 allow, 30-60 review/limits, >60 block/hold.
Signals:
| Signal | Weight idea |
|---|---|
| New account + high-value listing | + |
| Price far below market for category/model | ++ |
| Image reuse (pHash match elsewhere/web) | +++ |
| Multiple accounts same device/IP/payout account | +++ |
| Contact info in listing or chat, "pay advance", "bKash first" keywords | ++ |
| Rapid listing bursts, many categories | + |
| Chat pattern: copy-paste messages to many buyers | ++ |
| Many cancelled orders/disputes | +++ |
| Payout account used by other user | +++ |
| Listing description copied from another listing | + |
| Unusual login geography/device | + |
Actions: auto-hold listing for review, require extra proof (photo with handwritten code/date), delay payout, lower limits, notify admin.

## 5. Listing safety
- Proof-of-ownership photo (item with handwritten "ResaleMart + date") required for high-value categories.
- IMEI/serial checks: format validation; check blacklist of reported-stolen devices; BTRC IMEI/DIRBS-style check later if accessible.
- Prohibited-item keyword rules + image moderation (nudity, weapons) via classifier.
- Duplicate listing detection by title/description similarity and image hash.

## 6. Reporting and enforcement ladder
Report (listing/user/message) -> triage (auto priority by risk) -> action: warn, remove listing, suspend, ban + blacklist. SLA: high risk 4 h, normal 24 h. Reporter gets outcome notice. Abuse of reporting is itself penalised.

## 7. Blacklist keys
NID hash, phone hash, device fingerprint, payout account hash, IP (temporary), email. Checked at register, verification, payout account add, order creation.

## 8. User safety education
Safety tips in chat and checkout: "Never pay outside ResaleMart", "Meet in public places", "Test the item before confirming OTP", "Record unboxing video for courier orders". Banner when chat detects off-platform payment talk.

## 9. Law enforcement
Documented process: only on valid legal request, via legal contact, logged, minimal data disclosed.
