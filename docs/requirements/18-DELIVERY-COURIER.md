# 18 - Delivery Specification (Local Handover + Courier)

## 1. Modes
LOCAL, COURIER. Listing has allowed modes; order picks one.

## 2. Local handover
1. After seller accepts, both chat/propose meetup: place (suggest public spots: mall, metro station area, police-supervised "safe meetup point" list per city later), time.
2. Meetup agreed -> status HANDOVER_PENDING; app shows each other's verified name and photo, share-location toggle.
3. At meetup both tap "I'm here" (check-in, optional geo proximity check).
4. Buyer inspects/tests item. Checklist screen (powers on, screen, battery, accessories, serial matches).
5. Buyer shows OTP / enters it -> seller confirms -> COMPLETED -> release.
   If problem: "Reject item" with reason + photos -> DISPUTED or mutual cancel (refund).
6. No-show: after meetup time + grace 60 min, other party reports no-show; reschedule once; after 72 h from accept, auto-cancel and refund per BR-024.
Safety: advise day-time public places, bring a friend; SOS/report button.

## 3. Courier
Adapters: Steadfast, RedX, Pathao (interface `CourierProvider`).
```
interface CourierProvider {
  checkCoverage(fromUpazila, toUpazila): Coverage
  quote(parcel, from, to): Money
  createParcel(order): { parcelId, trackingCode, labelUrl }
  cancelParcel(parcelId)
  getStatus(trackingCode): CourierStatus
  verifyWebhook(req): Event
}
```
Flow:
1. Checkout: buyer enters address; system quotes delivery fee (by weight band and zone) added to total.
2. Seller accepts, packs item, records packing video/photos (required for >= configured value), creates parcel in-app (pickup request or drop at hub). Tracking saved.
3. Courier status events mapped: PICKED_UP -> SHIPPED, IN_TRANSIT, OUT_FOR_DELIVERY, DELIVERED, RETURNED, FAILED.
4. DELIVERED starts inspection window; buyer prompted to open parcel on video.
5. If delivery fails/returned: refund buyer, return to seller, delivery cost per fault.
6. Courier charged by prepaid wallet/or fee included; **no COD** (conflicts with escrow).
7. Insurance flag for high-value; claim process for damaged/lost.

## 4. Rules
- DLV-001 Delivery fee determined server side using courier quote cache.
- DLV-002 Seller must input tracking within 48 h or order cancels.
- DLV-003 Parcel weight/size limits; oversize (furniture, bikes) = LOCAL_ONLY or special freight (later).
- DLV-004 Address validated against location hierarchy; phone required.
- DLV-005 Polling fallback every 15 min for parcels in transit if webhook missing.
- DLV-006 Courier proof (POD image/OTP) stored on shipment.
- DLV-007 Buyer cannot mark "not received" if courier shows delivered with OTP proof unless disputed with evidence.

## 5. Delivery fee display
Item price + delivery fee + buyer protection fee = total; breakdown visible before payment.
