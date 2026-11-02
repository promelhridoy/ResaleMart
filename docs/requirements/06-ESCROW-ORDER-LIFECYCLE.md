# 06 - Escrow Order Lifecycle (State Machine)

## 1. States
| State | Meaning | Terminal |
|---|---|---|
| PENDING_PAYMENT | Order created, listing reserved, awaiting payment | no |
| PAYMENT_FAILED | Gateway failed / cancelled by buyer | yes |
| PAYMENT_EXPIRED | 15 min elapsed | yes |
| FUNDS_HELD | Money in escrow, awaiting seller accept | no |
| SELLER_REJECTED | Seller declined | yes (refund) |
| SELLER_EXPIRED | Seller did not accept in 24 h | yes (refund) |
| CANCELLED_BY_BUYER | Before accept | yes (refund) |
| ACCEPTED | Seller accepted; prepare delivery | no |
| CANCELLED_BY_SELLER | After accept | yes (refund) |
| SHIPPED | Courier parcel handed over with tracking | no |
| HANDOVER_PENDING | Local meetup being arranged | no |
| DELIVERED | Courier delivered (starts inspection) | no |
| INSPECTION | Inspection window running | no |
| COMPLETED | Buyer confirmed or auto-confirmed; funds releasing | no |
| RELEASED | Seller wallet credited (fee deducted) | yes |
| DISPUTED | Dispute open; funds frozen | no |
| RETURN_REQUESTED / RETURN_IN_TRANSIT / RETURN_RECEIVED | Return sub-flow | no |
| RESOLVED_RELEASED | Admin ruled for seller | yes |
| RESOLVED_REFUNDED | Admin ruled for buyer | yes |
| RESOLVED_PARTIAL | Split | yes |
| FAILED_DELIVERY | Courier could not deliver, returned to seller | no -> refund |
| CANCELLED_NO_HANDOVER | Local handover not completed in time | yes (refund) |

## 2. Diagram
```
PENDING_PAYMENT -> PAYMENT_FAILED | PAYMENT_EXPIRED | FUNDS_HELD
FUNDS_HELD -> ACCEPTED | SELLER_REJECTED | SELLER_EXPIRED | CANCELLED_BY_BUYER
ACCEPTED -> SHIPPED (courier) | HANDOVER_PENDING (local) | CANCELLED_BY_SELLER
SHIPPED -> DELIVERED | FAILED_DELIVERY
DELIVERED -> INSPECTION
HANDOVER_PENDING -> INSPECTION (meetup started) | CANCELLED_NO_HANDOVER
INSPECTION -> COMPLETED | DISPUTED
COMPLETED -> RELEASED
DISPUTED -> RETURN_REQUESTED | RESOLVED_RELEASED | RESOLVED_REFUNDED | RESOLVED_PARTIAL
RETURN_REQUESTED -> RETURN_IN_TRANSIT -> RETURN_RECEIVED -> RESOLVED_REFUNDED | RESOLVED_PARTIAL
```

## 3. Transition table
| From | To | Actor | Guard | Side effects |
|---|---|---|---|---|
| (new) | PENDING_PAYMENT | Buyer | Listing ACTIVE, buyer != seller, no other open order | Reserve listing, snapshot price/fees, start 15 min timer |
| PENDING_PAYMENT | FUNDS_HELD | System (webhook) | Signature valid, amount matches | Ledger: gateway -> escrow; notify seller |
| PENDING_PAYMENT | PAYMENT_EXPIRED/FAILED | System | timer / gateway fail | Release listing |
| FUNDS_HELD | ACCEPTED | Seller | within 24 h | Start delivery deadlines |
| FUNDS_HELD | SELLER_REJECTED / SELLER_EXPIRED | Seller / System | - | Refund, release listing, seller score |
| FUNDS_HELD | CANCELLED_BY_BUYER | Buyer | not yet accepted | Refund |
| ACCEPTED | SHIPPED | Seller | tracking ID + packing proof | Notify buyer |
| ACCEPTED | HANDOVER_PENDING | Seller or Buyer | local order | Open meetup flow, generate OTP |
| ACCEPTED | CANCELLED_BY_SELLER | Seller | - | Refund, strike |
| SHIPPED | DELIVERED | System (courier webhook) / Buyer | courier status delivered | Start inspection timer |
| DELIVERED | INSPECTION | System | immediate | timer 24 h (48 h high value) |
| HANDOVER_PENDING | INSPECTION | System | both checked-in at meetup | inspection happens on-site |
| INSPECTION | COMPLETED | Buyer (confirm / enter OTP) or System (timer end) | no open dispute | enqueue release |
| INSPECTION | DISPUTED | Buyer | within window, evidence attached | Freeze funds, notify seller + admin |
| COMPLETED | RELEASED | System | payout hold rules ok | Ledger: escrow -> seller wallet + platform fee |
| DISPUTED | RESOLVED_* | Admin | decision recorded | Ledger per decision |
| SHIPPED | FAILED_DELIVERY | System | courier returned parcel | Refund buyer, parcel back to seller |

## 4. Invariants
- INV-001 Exactly one open order per listing.
- INV-002 Terminal states never change (except admin correction with audit entry and compensating ledger).
- INV-003 Escrow account balance for an order is >0 only in FUNDS_HELD..DISPUTED/RETURN states.
- INV-004 Every transition writes `OrderStatusHistory(from,to,actor,reason,at)`.
- INV-005 Transitions run in a DB transaction with optimistic version check (`version` field).
- INV-006 Timers are driven by a job queue with idempotent handlers; handler re-checks current state before acting.
- INV-007 Sum of all ledger entries per transaction = 0.

## 5. Local handover OTP
- OTP generated when HANDOVER_PENDING starts; shown to buyer in app, valid for the handover window.
- At meetup buyer inspects item, then tells/enters OTP; seller enters it in their app to confirm handover.
- Wrong OTP 5 times locks order action and alerts support.
- If buyer rejects item on-site, seller/buyer press "Item not as described", order moves to DISPUTED with on-site evidence prompts.
