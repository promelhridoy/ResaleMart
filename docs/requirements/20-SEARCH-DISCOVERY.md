# 20 - Search and Discovery

## 1. v1 implementation
MongoDB indexes + text index; Atlas Search or Meilisearch when relevance/typo-tolerance needed (decision gate: if p95 search > 400 ms or relevance complaints).
Bangla search: normalise Unicode (NFC), handle common spelling variants, transliteration (Banglish) aliases for brands/categories (e.g. "laptop", "ল্যাপটপ").

## 2. Filters
Category tree, brand, price range, condition, location (division/district/upazila/radius), delivery mode, verified sellers, posted date (24h/7d/30d), has warranty, negotiable.

## 3. Sorting
Relevance, newest, price low/high, nearest, trusted-seller boost.

## 4. Ranking formula (default feed)
`score = freshness decay + seller reputation + image quality + proximity + engagement - risk score`. Promoted listings later, clearly labelled.

## 5. Performance
- Cursor pagination on (sortKey, _id).
- Popular queries and category pages cached in Redis 60 s; CDN cache for anonymous category pages (ISR 60 s).
- Facet counts computed async and cached.
- Hide inactive/sold from index immediately via status filter.

## 6. SEO
SSR/ISR listing and category pages, clean slugs (`/l/{id}-{slug}`), JSON-LD Product schema, sitemap (split by 50k URLs), canonical tags, noindex on filtered combos beyond depth, Open Graph images, Bangla titles.

## 7. Personalisation (later)
Recently viewed, saved search alerts, "similar items" by category+price band.

## 8. Search analytics
Log queries (no PII), zero-result queries, click-through, to improve categories/aliases.
