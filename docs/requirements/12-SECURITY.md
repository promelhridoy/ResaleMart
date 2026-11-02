# 12 - Security Specification

Baseline: OWASP ASVS L2, OWASP Top 10, API Security Top 10.

## 1. Threat model (summary)
| Threat | Examples | Controls |
|---|---|---|
| Account takeover | credential stuffing, SIM swap, OTP abuse | Argon2id, lockout, OTP limits, device fingerprint, new-device alerts, 2FA for staff, session revocation |
| Payment tampering | fake webhook, client price edit, double payout | HMAC verification, server-derived amounts, idempotency, ledger invariants, dual control |
| Data breach | DB dump, bucket leak, log leak | Field-level encryption, private buckets + signed URLs, minimal logs, least-privilege IAM, backups encrypted |
| Injection/XSS | NoSQL operator injection, stored XSS in chat/listing | Zod strict schemas, `$`-key stripping, output escaping, DOMPurify, CSP |
| CSRF | cookie auth abuse | SameSite=Lax/Strict, CSRF token on mutating cookie requests, Origin check |
| Abuse/DoS | scraping, OTP bombing, fake listings | Redis rate limits, WAF/CDN, CAPTCHA on risky flows, bot detection |
| Insider risk | admin snooping, wrong refund | RBAC, audit logs, dual control, NID view logging, 2FA |
| Marketplace fraud | stolen photos, fake NID, advance-payment scam | See 16 |
| IDOR | access to others' orders | Ownership policy checks on every read/write |

## 2. Controls
**Authentication**
- SEC-001 Argon2id (memory 64 MB, t=3, p=1 or tuned); pepper from secret manager optional.
- SEC-002 Access JWT 15 min (HS256 min 256-bit secret or RS256), refresh token random 256-bit, stored hashed, rotation + reuse detection.
- SEC-003 Cookies: HttpOnly, Secure, SameSite, `__Host-` prefix where possible.
- SEC-004 OTP: 6 digits, hashed, 5 min, 5 attempts, per-phone and per-IP caps; SMS pumping protection (country allowlist BD, daily budget alerts).

**Authorization**
- SEC-010 Central policy functions; deny by default; tests for each role x action.
- SEC-011 Object-level checks on every id param.

**Data protection**
- SEC-020 TLS 1.2+ everywhere, HSTS, internal traffic TLS in production.
- SEC-021 NID number AES-256-GCM encrypted; unique lookup via HMAC-SHA256 with separate key; key IDs stored for rotation; keys in KMS/secret manager.
- SEC-022 NID/selfie/dispute evidence in PRIVATE bucket, server-side encryption, signed URLs TTL <= 5 min, access logged.
- SEC-023 Payout account numbers encrypted; only last4 displayed.
- SEC-024 Image pipeline: validate MIME by magic bytes, re-encode, strip EXIF, virus scan (ClamAV) for evidence files.
- SEC-025 Backups encrypted, access restricted, restore drills.

**Input & output**
- SEC-030 Zod validation, body size limits (JSON 100 KB, uploads via presigned only).
- SEC-031 Mongo sanitization: reject keys beginning with `$` or containing `.` in user input.
- SEC-032 CSP (no inline scripts, nonce), X-Content-Type-Options, Referrer-Policy, frame-ancestors none, Permissions-Policy.
- SEC-033 Chat masking and link policy (no clickable external links before order).

**Payments**
- SEC-040 Verify webhook signature + timestamp window + `webhook_events` dedupe.
- SEC-041 Cross-check payment with gateway validation API (server-to-server) before FUNDS_HELD.
- SEC-042 Amount, currency, orderNo must match; mismatch -> flag, no state change, alert.
- SEC-043 Payout: verified account, name match, holds, limits, dual approval above threshold.
- SEC-044 Reconciliation job daily: gateway report vs ledger; mismatches alert finance.
- SEC-045 No card data stored; card handled on gateway hosted page (PCI scope minimal).

**Operations**
- SEC-050 Secrets in secret manager; rotation plan; no secrets in git (pre-commit scan).
- SEC-051 Dependency scanning (npm audit/Dependabot), SAST, container scan in CI.
- SEC-052 Least-privilege DB users; DB not publicly reachable; IP allowlist.
- SEC-053 Security headers verified in CI; penetration test before launch and yearly.
- SEC-054 Incident response runbook: detect, contain, rotate keys, notify users/authority per law, postmortem.
- SEC-055 Alerts: spike in failed logins, OTP sends, payment mismatches, admin NID views, refunds, payout volume.

## 3. Audit log events (minimum)
login/fail, password change, verification submit/approve/reject, NID image view, listing moderation, order forced state, ledger adjustment, refund, payout approve/reject, config change, ban/unban, role change, blacklist change, export of data.

## 4. Privacy
- SEC-060 Privacy policy and consent at registration; explicit consent for NID/selfie processing.
- SEC-061 Data subject requests: export and deletion flow (respecting legal retention).
- SEC-062 Seller exact address never shown publicly; show upazila/area only; exact meetup point after order.
- SEC-063 Strip GPS EXIF; listing geo stored with 300-500 m jitter for public use.
