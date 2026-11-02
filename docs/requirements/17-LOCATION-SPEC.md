# 17 - Location Specification (Division > District > Upazila/Area)

## 1. Data
- Seed from Bangladesh administrative data: 8 divisions, 64 districts, ~495 upazilas/thanas (verify against latest official BBS/LGED dataset; store BBS codes). Bangla + English names. City-corporation thanas/areas (e.g. Mirpur, Dhanmondi, Uttara, Kandirpar) added as AREA level under upazila/thana.
- Seed script `seed/locations.ts` idempotent by code; admin can add AREA entries.
- Centroid coordinates for each unit (for radius search fallback).

## 2. Model
`locations` collection per 09. Listing and user store IDs of division/district/upazila plus optional AREA text and approximate geo point.

## 3. UX
- Cascading dropdowns with search (type to filter in Bangla/English).
- "Use my location" (browser geolocation, optional) reverse-maps to nearest upazila; user confirms.
- Default from profile; remembered in cookie for guests.
- Listing card shows "Upazila, District" only. Exact address never public.

## 4. Search behaviours
| Mode | Query |
|---|---|
| Area | filter by upazilaId |
| District | filter by districtId |
| Division | filter by divisionId |
| Nearby | `$geoNear`/`$geoWithin $centerSphere` on listing.geo within radius (1-50 km), sorted by distance |
| Nationwide courier | only listings with deliveryMode BOTH/COURIER_ONLY |
- Default for logged-in user: nearby district first, expand suggestions "also in nearby districts".
- Facet counts per district/upazila cached.

## 5. Rules
- LOC-001 Location required for all listings.
- LOC-002 Local-only listing visible nationwide but marked; checkout blocks local order if buyer district differs, offering chat/meetup agreement only if within same district or user acknowledges travel.
- LOC-003 Courier availability depends on courier coverage by upazila (courier coverage table synced from courier API).
- LOC-004 Public geo point jittered 300-500 m for privacy.
- LOC-005 Location names are matched in Bangla spelling variants via alias list (e.g. Comilla/Cumilla, Chittagong/Chattogram).

## 6. Admin
CRUD aliases, add AREA, deactivate; import updates with diff review.
