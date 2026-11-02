# 04 - Business Rules

All values are defaults stored in an admin-editable `SystemConfig` collection.

## 1. Fees (BR-FEE)
- BR-001 Seller commission: 5% of item price, min 20 BDT, max 2,000 BDT, deducted at release.
- BR-002 Buyer protection fee: 1.5% of item price, min 10 BDT (shown at checkout).
- BR-003 Payment gateway fee is absorbed by platform (included in above), shown in finance reports.
- BR-004 Listing is free; promoted listings are a later feature.
- BR-005 Fee snapshot is stored on each order and never recalculated.

## 2. Limits
- BR-010 Max single order: 200,000 BDT in v1 (raise per tier).
- BR-011 New seller (no completed sales): max 3 active listings, max 30,000 BDT per item until 3 successful sales.
- BR-012 Trusted seller (>=10 sales, rating >=4.5, no lost disputes in 90 days): auto-publish, instant payout after release.
- BR-013 Listing images 3-10; description 30-5000 chars.
- BR-014 Daily limits: 20 new listings per seller, 50 chat conversations started per buyer, 5 offers per buyer per listing.

## 3. Timers
| Event | Duration |
|---|---|
| Payment window | 15 min |
| Seller accept window | 24 h |
| Courier tracking upload after accept | 48 h |
| Local handover window after accept | 72 h |
| Courier inspection window (after delivered) | 24 h (48 h for high-value categories) |
| Local inspection | At meetup, confirmed with OTP |
| Seller dispute response | 48 h |
| Admin target resolution | 72 h |
| Return shipping after approved return | 72 h to ship; 5 days to arrive |
| Review window | 14 days |
| Offer expiry | 24 h default |
| Listing expiry | 60 days |
| Payout hold (new seller first payout) | 24 h |

## 4. Cancellation rules
- BR-020 Buyer cancels before seller accepts: full refund, no penalty.
- BR-021 Seller rejects or lets it expire: full refund; repeated expiry lowers seller score.
- BR-022 Seller cancels after accepting: full refund to buyer, seller strike; 3 strikes in 30 days = listing restriction.
- BR-023 Buyer cancels after seller accepted: only allowed if seller agrees, else must dispute; abuse lowers buyer score.
- BR-024 Local handover buyer no-show (72 h): order cancels, buyer refunded minus buyer protection fee if the seller kept the item reserved and proved attendance by seller OTP check-in (configurable).

## 5. Inspection and return
- BR-030 Item "not as described" qualifies for refund: wrong model/spec, undisclosed defect, replica sold as genuine, missing parts listed, severe condition mismatch.
- BR-031 NOT valid grounds: change of mind, minor cosmetic wear consistent with stated condition, buyer-caused damage.
- BR-032 Return shipping cost: seller pays if claim upheld; buyer pays if rejected.
- BR-033 Seller receives item back inspected by admin/photo before refund release when return approved (refund held until return delivered or 5 days).
- BR-034 Courier damage in transit: claim via courier insurance; platform refunds buyer first for insured parcels (rule configurable).

## 6. Verification
- BR-040 Selling requires L2 verification. Buying requires L1 (phone) only.
- BR-041 One NID per account. One phone per account.
- BR-042 Seller legal name must match payout account holder name (fuzzy match, manual override).
- BR-043 Account sharing or selling accounts is prohibited, leads to ban.

## 7. Prohibited items (needs legal review)
Weapons and ammunition, drugs and prescription medicine, alcohol, tobacco/vape, counterfeit goods and replicas sold as genuine, stolen goods, items with blocked/untraceable IMEI, adult content, live animals, human organs/blood, government IDs or documents, counterfeit currency, explosives and chemicals, pirated software/media, anything illegal under Bangladeshi law.

## 8. Conduct
- BR-050 No off-platform payment solicitation ("send advance to bKash"). First offence warning, second offence suspension.
- BR-051 Contact info in chat masked until order is paid.
- BR-052 Harassment, hate speech, spam: warning, mute, then ban.

## 9. Blacklist
- BR-060 Confirmed fraud leads to permanent ban; NID hash, phone, device fingerprint, payout account added to blacklist.
- BR-061 Banned user's funds: pending payouts frozen until investigation; legal handover to authorities on valid request.
- BR-062 Appeals reviewed within 7 days by a different admin.

## 10. Reputation score (0-100)
Start 50 at L2. + completed sale (+2), + 5-star review (+1), - lost dispute (-15), - seller cancel (-5), - confirmed report (-10), - slow shipping (-2). Score drives limits, search ranking boost and badges (Bronze/Silver/Gold seller).
