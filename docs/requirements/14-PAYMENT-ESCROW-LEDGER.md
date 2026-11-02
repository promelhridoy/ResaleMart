# 14 - Payment, Escrow and Ledger Specification

## 1. Legal and structural prerequisite (decide before building production money flow)
Holding buyer funds until delivery is regulated activity in many jurisdictions. For Bangladesh, confirm with a lawyer/Bangladesh Bank requirements for payment service providers and e-commerce escrow. Options:
A. Partner with a licensed PSP/aggregator/bank that provides held-settlement (marketplace/split settlement) and make ResaleMart the orchestration layer.
B. Obtain required licence and hold funds in a designated trust/escrow bank account.
C. (MVP/pilot only) Limited pilot under legal advice with strict caps.
The ledger below models the funds regardless of who physically holds them. Do not launch with real money before this is settled.

## 2. Payment methods
bKash, Nagad, Rocket, cards (Visa/Master), internet banking via SSLCommerz-class aggregator. Hosted payment page; ResaleMart never touches card/PIN data.

## 3. Money representation
Integer poisha. BDT display with Bangla/English digits. Rounding: half-up on percentage fees, applied once per order, stored in snapshot.

## 4. Ledger model (double entry)
Accounts:
- `GATEWAY_CLEARING` (money in transit from gateway)
- `ESCROW_HOLD` (per-order sub-ledger via refId)
- `USER_WALLET:{userId}` (seller available balance)
- `PLATFORM_REVENUE`
- `FEE_EXPENSE` (gateway cost)
- `REFUND_PAYABLE`
- `PAYOUT_CLEARING`

Transactions (D = debit, C = credit; amounts per order: item I, delivery fee Df, buyer fee Bf, commission Sc):
| Event | Entries |
|---|---|
| Payment success | D GATEWAY_CLEARING (I+Df+Bf) / C ESCROW_HOLD (I+Df) / C PLATFORM_REVENUE_PENDING (Bf) |
| Release (completed) | D ESCROW_HOLD (I+Df) / C USER_WALLET seller (I+Df-Sc) / C PLATFORM_REVENUE (Sc); Bf moved pending->revenue |
| Full refund (pre-release) | D ESCROW_HOLD (I+Df) / C REFUND_PAYABLE (I+Df); Bf policy: refunded when platform fault/seller fault, kept for buyer-fault per BR |
| Partial refund x | D ESCROW_HOLD / C REFUND_PAYABLE (x) / C USER_WALLET seller (rest-minus commission on retained part) |
| Refund paid out | D REFUND_PAYABLE / C GATEWAY_CLEARING |
| Payout request | D USER_WALLET / C PAYOUT_CLEARING |
| Payout success | D PAYOUT_CLEARING / C GATEWAY_CLEARING (cash out) |
| Payout failed | D PAYOUT_CLEARING / C USER_WALLET (reverse) |
| Admin adjustment | explicit pair with reason, dual approval |

Rules:
- PAY-001 Entries append-only; balance = sum of entries; `balanceCache` verified nightly.
- PAY-002 Every ledger transaction has unique idempotencyKey.
- PAY-003 Per transaction debits == credits; enforced in code and tested.
- PAY-004 Wallet can never go negative (conditional update).
- PAY-005 Reconciliation daily against gateway settlement reports; unmatched items go to review queue.

## 5. Payment flow
1. POST /orders creates PENDING_PAYMENT order, Payment INITIATED, calls gateway init with `tran_id = orderNo-attempt`, success/fail/cancel/ipn URLs.
2. Buyer pays on hosted page.
3. IPN/webhook arrives -> verify signature/hash, call gateway validation API, check amount/currency/tranId, dedupe in `webhook_events`.
4. Tx: payment SUCCESS, ledger payment entries, order -> FUNDS_HELD, outbox events.
5. Browser return URL only shows status by reading DB (never marks paid).
6. If webhook delayed: poller checks gateway status for orders PENDING_PAYMENT after 2 min.
7. Late payment after expiry: auto-refund and alert (listing may be gone).

## 6. Refunds
- Trigger sources: seller reject/expire, buyer cancel pre-accept, seller cancel, failed delivery, dispute decision, admin manual.
- Prefer gateway refund API to original method; fallback manual bKash/Nagad send by finance with proof stored.
- Refund status machine: REQUESTED -> PROCESSING -> SUCCESS/FAILED(retry/manual).
- Target SLA: auto-refund within 24 h, disputes within 3 business days after decision.

## 7. Payouts
- Wallet balance available after release (+ hold rules). Min payout 100 BDT; fee per BR (or free in v1).
- Verified payout account (OTP on add, holder name matches NID name fuzzy >= 0.85 or manual review).
- New payout account has 24 h cooling period.
- Auto payout via gateway disbursement API where available; else finance batch export (CSV) + mark paid with transaction ref.
- Limits: daily per user, per-payout threshold requiring finance approval.

## 8. Fees and tax
Fees per BR; invoices/receipts generated per order (PDF) with order no, fees breakdown. VAT/tax treatment on platform fees to be confirmed with accountant.

## 9. Failure handling
- Gateway timeout at init: order stays PENDING_PAYMENT; allow retry with new attempt id; expire at 15 min.
- Duplicate webhook: ignored by dedupe.
- Amount mismatch: mark payment FLAGGED, no escrow, alert finance, refund after review.
- Server crash mid-transaction: Mongo transaction rolls back; webhook retried by gateway; idempotent.
