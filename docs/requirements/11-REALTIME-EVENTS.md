# 11 - Real-Time Events (Socket.IO)

## Connection
- Namespace `/` ; auth via access cookie/token in handshake; reject if invalid.
- Redis adapter for multi-instance. Heartbeat 25 s. Reconnect with backoff; client refetches missed data via REST using last cursor.
- Max 5 sockets per user; per-socket rate limit 30 events/10 s.

## Rooms
| Room | Who joins | Used for |
|---|---|---|
| `user:{userId}` | the user (auto) | notifications, order updates, unread counts |
| `conversation:{id}` | buyer + seller on open | messages, typing, read receipts |
| `order:{id}` | buyer + seller on open | live order timeline |
| `admin:disputes`, `admin:verifications`, `admin:reports` | staff by role | queue updates |

Join requests are authorized server side (membership check) every time.

## Server -> client events
| Event | Payload |
|---|---|
| `notification.created` | {id,type,title,body,data} |
| `message.new` | {conversationId,message} |
| `message.read` | {conversationId,readBy,at} |
| `typing` | {conversationId,userId,isTyping} |
| `presence.update` | {userId,online} (only inside conversation) |
| `offer.updated` | {offer} |
| `order.updated` | {orderId,status,version,timeline entry} |
| `order.timer` | {orderId,timer,dueAt} |
| `dispute.updated` | {disputeId,status} |
| `listing.status` | {listingId,status} (for open listing page) |
| `verification.updated` | {status} |
| `payout.updated` | {payoutId,status} |

## Client -> server events
`conversation.join {id}`, `conversation.leave`, `message.send {conversationId,text,clientMsgId}` (ack with saved message), `message.read {conversationId,lastMessageId}`, `typing {conversationId,isTyping}`, `order.join {id}`.

## Rules
- RT-001 Messages are persisted before broadcast; `clientMsgId` dedupes retries.
- RT-002 Order state is never changed through sockets; sockets only push state, REST mutates.
- RT-003 Payloads contain no sensitive data (no NID, no full phone numbers).
- RT-004 Masking applies to message text before saving/broadcasting.
