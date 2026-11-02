# 05 - Roles and RBAC

## 1. Roles
GUEST, USER (buyer+seller capabilities gated by verification level), MODERATOR, SUPPORT, FINANCE, ADMIN, SUPER_ADMIN.

## 2. Verification gates (apply to USER)
| Level | Name | Unlocks |
|---|---|---|
| L0 | Registered | Browse |
| L1 | Phone verified | Chat, offers, buy, favourite |
| L2 | NID verified | Create listings, receive payouts |
| L3 | Trusted | Auto-publish, instant payout, higher limits |

## 3. Permission matrix
| Action | Guest | User L1 | User L2 | Mod | Support | Finance | Admin |
|---|---|---|---|---|---|---|---|
| Browse/search listings | Y | Y | Y | Y | Y | Y | Y |
| Chat / make offer | - | Y | Y | - | - | - | - |
| Buy with escrow | - | Y | Y | - | - | - | - |
| Create listing | - | - | Y | - | - | - | Y |
| Edit own listing | - | - | Y | - | - | - | Y |
| Moderate any listing | - | - | - | Y | - | - | Y |
| Review NID submissions | - | - | - | - | - | - | Y |
| View NID images | - | - | - | - | - | - | Y (audited) |
| Ban / unban user | - | - | - | suspend only | - | - | Y |
| View any order | - | own | own | limited | Y | Y | Y |
| Decide dispute | - | - | - | - | recommend | - | Y |
| View ledger | - | own wallet | own wallet | - | - | Y | Y |
| Approve payouts above threshold | - | - | - | - | - | Y | Y |
| Issue manual refund | - | - | - | - | - | Y | Y |
| Edit SystemConfig | - | - | - | - | - | fees only | Y |
| Manage admin accounts | - | - | - | - | - | - | SUPER_ADMIN |
| View audit log | - | - | - | - | - | - | Y |

## 4. Ownership rules
- RBAC-001 A user may only read/modify their own listings, orders, conversations, wallet, addresses.
- RBAC-002 Order actions restricted by role in order: only seller can accept/ship; only buyer can confirm/dispute.
- RBAC-003 Authorization is enforced server side in policy functions, never trusted from client.
- RBAC-004 Staff accounts require 2FA (TOTP), IP allowlist optional, session max 8 h.
- RBAC-005 Dual control: manual refunds and payout overrides above config threshold need two different staff.
- RBAC-006 Every denied admin-area access attempt is logged.
