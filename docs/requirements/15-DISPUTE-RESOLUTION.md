# 15 - Dispute Resolution Specification

## 1. Goals
Fair, evidence-based, fast. Protect honest buyers from bad items and honest sellers from false claims.

## 2. Eligibility
- Opened only by the buyer, only in INSPECTION (or at local meetup rejection).
- Valid reasons: NOT_AS_DESCRIBED, NOT_RECEIVED (courier shows delivered but buyer denies, needs courier proof), DAMAGED, FAKE (replica/counterfeit), MISSING_PARTS, OTHER.
- Mandatory: description (>=30 chars) and at least 1 photo/video evidence (unboxing video encouraged for high-value).
- Seller-initiated dispute: allowed for "buyer no-show" and "return not as sent" cases.

## 3. Flow
1. Buyer opens dispute -> order DISPUTED, funds frozen, timers paused.
2. Seller notified; 48 h to respond with text + evidence (packing video, invoice, IMEI proof, chat history auto-attached).
3. Optional automated mediation: offer partial refund/keep item discount; if both accept, auto-resolve PARTIAL.
4. Admin review (target 72 h): sees order, listing snapshot (as listed), chat log, evidence from both, courier tracking, both users' history/score.
5. Decision options:
   - FULL_RELEASE (claim rejected)
   - FULL_REFUND without return (cheap items, item destroyed/fake)
   - FULL_REFUND after RETURN (item must be sent back; refund when return delivered or confirmed)
   - PARTIAL_REFUND (x%) keep item
6. Ledger executes per decision; order -> RESOLVED_*.
7. Both parties may appeal once within 48 h to a different admin (senior).

## 4. Evidence rules
- Evidence stored private; both parties see each other's evidence except personal data.
- Chat history snapshot frozen at dispute open.
- Metadata (upload time, EXIF stripped for privacy but server-recorded time kept) used to detect last-minute fabricated proof.
- Video preferred for functional defects ("does not power on").

## 5. Decision guideline (for admins)
| Situation | Typical outcome |
|---|---|
| Listing hid a clear defect, buyer has evidence | Refund (seller pays return shipping) |
| Item exactly as described, buyer regret | Release to seller |
| Counterfeit/replica sold as genuine | Full refund, seller strike, possible ban |
| Courier damage, seller packed well | Refund via courier claim, no seller penalty |
| Parcel empty / wrong item | Refund; investigate seller; ban if repeated |
| Both partially right (minor undisclosed scratch) | Partial refund 10-30% |
| Buyer claims not received, courier proof of delivery with OTP | Release unless strong counter-evidence |
| Seller non-responsive | Rule on buyer evidence |

## 6. Anti-abuse
- Buyer dispute rate > 30% over 5+ orders triggers review; repeated false claims restrict buying.
- Seller lost-dispute rate > 10% reduces limits; 3 fraud findings = ban + blacklist.
- Dispute messages are moderated; threats escalate to support.

## 7. SLAs and metrics
Seller response 48 h; admin decision 72 h; refund execution 24 h after decision. Track: dispute rate, win rate by reason, time to resolve, appeal reversal rate.

## 8. Return sub-flow
RETURN_REQUESTED -> buyer ships within 72 h with tracking -> RETURN_IN_TRANSIT -> seller confirms receipt or 5 days after courier delivered auto-confirm -> RESOLVED_REFUNDED. If buyer never ships, dispute re-opens for admin to release funds to seller.
