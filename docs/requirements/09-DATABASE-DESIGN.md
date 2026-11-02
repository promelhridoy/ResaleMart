# 09 - Database Design (MongoDB + Mongoose)

Conventions: `_id` ObjectId, `createdAt/updatedAt`, money = integer poisha, enums UPPER_SNAKE, `version` on state-bearing docs.

## users
phone(unique), phoneVerifiedAt, email(unique sparse), passwordHash, name, avatarUrl, role(USER|MODERATOR|SUPPORT|FINANCE|ADMIN|SUPER_ADMIN), verificationLevel(0-3), status(ACTIVE|SUSPENDED|BANNED), defaultLocation{divisionId,districtId,upazilaId}, reputation{score,completedSales,completedBuys,ratingAvg,ratingCount,responseRate}, strikes, lastLoginAt, twoFactor{enabled,secretEnc}, deviceIds[].
Indexes: phone unique; email unique sparse; {status:1}; {verificationLevel:1}.

## sessions
userId, refreshTokenHash, familyId, userAgent, ip, deviceFingerprint, expiresAt(TTL), revokedAt, replacedBy.
Index: {userId:1}, {familyId:1}, TTL expiresAt.

## otp_codes
phone, purpose, codeHash, attempts, expiresAt(TTL), consumedAt.

## verifications
userId, status(PENDING|APPROVED|REJECTED|NEEDS_RESUBMIT), nidNumberEnc, nidNumberHash(HMAC keyed, unique when APPROVED), nidFrontKey, nidBackKey, selfieKey (private bucket keys), fullNameOnNid, dob, submittedAt, reviewedBy, reviewedAt, rejectReason, attemptCount, retentionUntil.
Indexes: {userId:1}, {status:1,submittedAt:1}, unique partial on nidNumberHash where status=APPROVED.

## categories
name{bn,en}, slug, parentId, path[], attributesSchema[] (e.g. brand, model, IMEI required), isHighValue, prohibited, sortOrder, active.

## locations (Division > District > Upazila)
type(DIVISION|DISTRICT|UPAZILA|AREA), name{bn,en}, slug, parentId, code (BBS code), centroid GeoJSON Point, active.
Indexes: {type:1,parentId:1}, 2dsphere centroid, unique slug.

## listings
sellerId, title, description, categoryId, categoryPath[], brand, model, attributes{}, condition(NEW|LIKE_NEW|GOOD|FAIR|FOR_PARTS), defectsNote, price(int), negotiable, images[{key,url,thumbUrl,pHash,order}], videoUrl, location{divisionId,districtId,upazilaId,areaText,geo GeoJSON Point(approximate)}, deliveryMode(LOCAL_ONLY|COURIER_ONLY|BOTH), courierWeightKg, courierSize, status(DRAFT|PENDING_REVIEW|ACTIVE|PAUSED|RESERVED|SOLD|REJECTED|EXPIRED|REMOVED), rejectReason, activeOrderId, serialOrImei (encrypted), stats{views,favs,chats}, publishedAt, expiresAt, searchText.
Indexes: {status:1,publishedAt:-1}; {status:1,categoryPath:1,price:1}; {status:1,'location.districtId':1,publishedAt:-1}; {status:1,'location.upazilaId':1}; 2dsphere 'location.geo'; text(title,description,brand,model); {sellerId:1,status:1}; {images.pHash:1}.

## favorites
userId, listingId, createdAt. Unique {userId,listingId}.

## saved_searches
userId, query{}, notify(bool), lastNotifiedAt.

## conversations
listingId, buyerId, sellerId, lastMessage{text,at,senderId}, unread{buyer,seller}, status(OPEN|CLOSED|BLOCKED), orderId?.
Unique {listingId,buyerId,sellerId}; {buyerId:1,updatedAt:-1}; {sellerId:1,updatedAt:-1}.

## messages
conversationId, senderId, type(TEXT|IMAGE|OFFER|SYSTEM), text, maskedText, attachments[], offerId?, readAt, createdAt.
Index {conversationId:1,createdAt:-1}.

## offers
listingId, conversationId, buyerId, sellerId, amount, status(PENDING|ACCEPTED|REJECTED|COUNTERED|EXPIRED), parentOfferId, expiresAt(TTL-job).

## orders
orderNo(unique, human readable e.g. RM-240101-AB12), listingId, listingSnapshot{title,image,category,condition}, buyerId, sellerId, status, version,
amounts{itemPrice,deliveryFee,buyerFee,sellerCommission,total},
feeSnapshot{commissionPct,buyerFeePct,minMax},
deliveryMethod(LOCAL|COURIER), deliveryAddress{name,phone,divisionId,districtId,upazilaId,addressLine,geo}, meetup{place,time,proposedBy,sellerCheckedInAt,buyerCheckedInAt},
handover{otpHash,attempts,expiresAt,confirmedAt}, shipment{courier,trackingId,parcelId,status,packingProofKeys[],shippedAt,deliveredAt},
inspection{startsAt,endsAt}, timers{paymentDueAt,sellerAcceptDueAt,shipDueAt,inspectionEndsAt}, disputeId, idempotencyKey(unique per buyer), paymentId, completedAt, releasedAt.
Indexes: {buyerId:1,createdAt:-1}, {sellerId:1,createdAt:-1}, {status:1,'timers.*':1} (per timer), unique orderNo, unique {listingId} partial where status in open states.

## order_status_history
orderId, from, to, actorType(BUYER|SELLER|SYSTEM|ADMIN), actorId, reason, meta, at. Index {orderId:1,at:1}.

## payments
orderId, userId, provider(SSLCOMMERZ|BKASH|NAGAD|...), providerTxnId, valId, method, amount, currency(BDT), status(INITIATED|SUCCESS|FAILED|CANCELLED|REFUNDED|PARTIAL_REFUND), rawResponse(redacted), webhookEvents[], idempotencyKey.
Unique providerTxnId.

## webhook_events
provider, eventId (unique), signatureValid, payloadHash, processedAt, result. Ensures idempotency.

## ledger_accounts
type(GATEWAY_CLEARING|ESCROW_HOLD|USER_WALLET|PLATFORM_REVENUE|REFUND_PAYABLE|PAYOUT_CLEARING|FEE_EXPENSE), ownerUserId?, currency, balanceCache, version.

## ledger_transactions
txnId, type(ESCROW_FUND|RELEASE|REFUND|PARTIAL_REFUND|PAYOUT|PAYOUT_FAIL|ADJUSTMENT), refType, refId (order/payout), idempotencyKey(unique), createdBy, createdAt.

## ledger_entries (append-only)
txnId, accountId, direction(DEBIT|CREDIT), amount(>0), balanceAfter, createdAt. Invariant: per txnId sum(debit)=sum(credit). Indexes {accountId:1,createdAt:-1}, {txnId:1}.

## payout_accounts
userId, type(BKASH|NAGAD|ROCKET|BANK), accountNumberEnc, accountLast4, holderName, verifiedAt, status.

## payouts
userId, payoutAccountId, amount, fee, status(REQUESTED|HOLD|APPROVED|PROCESSING|SUCCESS|FAILED|CANCELLED), holdUntil, approvedBy, providerRef, failureReason, idempotencyKey.

## disputes
orderId, openedBy, reason(NOT_AS_DESCRIBED|NOT_RECEIVED|DAMAGED|FAKE|MISSING_PARTS|OTHER), description, buyerEvidence[], sellerResponse{text,evidence[],at}, status(OPEN|AWAITING_SELLER|UNDER_REVIEW|RETURN_REQUIRED|RESOLVED), decision{type,refundPct,reasonText,decidedBy,at}, slaDueAt, messages[]. Index {status:1,slaDueAt:1}.

## reviews
orderId, reviewerId, revieweeId, role(BUYER|SELLER), rating(1-5), tags[], comment, status(VISIBLE|HIDDEN), editableUntil. Unique {orderId,reviewerId}.

## reports
reporterId, targetType(LISTING|USER|MESSAGE|REVIEW), targetId, reason, details, status(OPEN|ACTIONED|DISMISSED), handledBy.

## blacklist
type(NID_HASH|PHONE|DEVICE|PAYOUT_ACCOUNT|IP), valueHash, reason, addedBy, addedAt, expiresAt?.

## notifications
userId, type, title, body, data, channels[], readAt, createdAt. TTL 180 days.

## outbox_events
type, aggregateId, payload, status(PENDING|SENT|FAILED), attempts, nextAttemptAt.

## audit_logs (append-only)
actorId, actorRole, action, targetType, targetId, before, after, ip, userAgent, requestId, at. Retain 5 years.

## system_config
key, value, updatedBy, updatedAt (fees, timers, limits, feature flags).

## Relationship summary
user 1-N listings; listing 1-N conversations; conversation 1-N messages/offers; listing 1-0..1 open order; order 1-1 payment; order 1-N ledger transactions; order 0..1 dispute; order 0..2 reviews.

## Retention
NID images: delete 12 months after verification approval unless law/fraud case requires hold (decide with lawyer). Chats: 2 years. Ledger/audit: 5+ years.
