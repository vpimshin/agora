# 06 — API Contracts

## 1. Conventions

- **Base URL:** `https://<project>.supabase.co/functions/v1` for edge functions; `https://<project>.supabase.co/rest/v1` for PostgREST direct queries.
- **Auth:** `Authorization: Bearer <jwt>` (Supabase user JWT). Anonymous access disabled.
- **Content type:** `application/json` (UTF-8). Streams: `text/event-stream`.
- **IDs:** UUID v4. **Timestamps:** RFC 3339 UTC.
- **Errors:** RFC 9457 Problem Details JSON.
- **Versioning:** URL prefix `/v1/`. Breaking changes → `/v2/`.
- **Idempotency:** mutating endpoints accept `Idempotency-Key` header.
- **Rate limits:** 60 req/min/user default, 10 req/min/user for LLM-bound endpoints. 429 with `Retry-After`.

### Standard error shape

```json
{
  "type": "https://agora.app/errors/criteria-not-binary",
  "title": "Criterion is not binary",
  "status": 422,
  "detail": "The criterion 'feel healthier' cannot be evaluated as Yes/No.",
  "instance": "/v1/criteria/abc",
  "code": "CRIT_NOT_BINARY",
  "trace_id": "01HX..."
}
```

---

## 2. Authentication

| Method | Path | Description |
|-------:|------|-------------|
| POST | `/v1/auth/magic-link` | Email magic link |
| POST | `/v1/auth/oauth/callback` | OAuth callback (Google) |
| POST | `/v1/auth/refresh` | Refresh JWT |
| POST | `/v1/auth/logout` | Invalidate refresh token |

```http
POST /v1/auth/magic-link
{
  "email": "amina@example.org",
  "redirect_to": "https://agora.app/auth/callback",
  "locale": "fr"
}
→ 202 Accepted
```

---

## 3. Tribes

```http
POST /v1/tribes                       # create
GET    /v1/tribes/me                  # tribes I belong to
GET    /v1/tribes/:id                 # detail
POST  /v1/tribes/:id/invite           # rotate invite code
POST  /v1/tribes/join                 # redeem code
DELETE /v1/tribes/:id/members/:uid    # admin/mod removal
```

Create:
```json
{ "name":"Career Switchers Q1", "kind":"tribe" }
→ 201 { "id":"...", "invite_code":"K3F-9PX", "expires_at":"..." }
```

Join:
```json
{ "code":"K3F-9PX" }
→ 200 { "tribe_id":"...", "role":"member" }
```

---

## 4. Coaching (S1)

### 4.1 Cognitive turn — SSE stream

```http
POST /v1/coach/turn
Content-Type: application/json
Accept: text/event-stream

{
  "tribe_id": "uuid",
  "user_message": "I want to lose weight",
  "draft_version": 0
}
```

Streamed events:
```
event: token
data: {"chunk":"You said you want to lose weight."}

event: token
data: {"chunk":" "}

event: token
data: {"chunk":"What does success look like in 30 days?"}

event: done
data: {"message_id":"uuid","trace_id":"01HX...","draft_version":1}
```

### 4.2 Alignment vote

```http
POST /v1/coach/align
{ "goal_id":"uuid", "draft_version":3, "aligned":true }
→ 200 { "tally":{"aligned":3,"total":4}, "unanimous":false }
```

When the server receives the unanimous vote, it broadcasts on the realtime channel `tribe:{id}:alignment` and atomically transitions S1 → S2.

---

## 5. Criteria (S2)

```http
POST /v1/criteria/propose
{
  "goal_id":"uuid",
  "question":"Did you run 5 km under 30 min?",
  "verification_method":"external_log"
}
→ 200 { "id":"uuid", "binary":true }

POST /v1/criteria/lock
{ "goal_id":"uuid" }
→ 200 { "state":"S3" }
```

Validation rules: questions must end with `?`, contain a binary verb, parse to Yes/No (LLM-validated and rule-validated). Subjective adjectives (`good`, `better`, `healthier`) trigger 422.

---

## 6. Resources & RAG (S3)

### 6.1 Upload

```http
POST /v1/resources/upload
Content-Type: multipart/form-data

tribe_id=uuid
goal_id=uuid
kind=pdf
file=<binary>
→ 202 { "resource_id":"uuid", "status":"queued" }
```

### 6.2 Add by URL

```http
POST /v1/resources/url
{ "tribe_id":"uuid", "url":"https://...", "kind":"url" }
```

### 6.3 Query

```http
POST /v1/rag/query
{
  "tribe_id":"uuid",
  "goal_id":"uuid",
  "query":"How do JWT refresh tokens work?",
  "depth":"standard",          // summary | standard | deep_dive
  "format":"prose"             // prose | bullets | flashcards | mcq | mindmap
}
→ 200 {
  "answer_md":"...",
  "citations":[{"chunk_id":"uuid","title":"...","score":0.83}],
  "graph_path":[{"from":"JWT","to":"Refresh Token","weight":0.74}]
}
```

### 6.4 Pedagogy generation

```http
POST /v1/pedagogy/generate
{
  "tribe_id":"uuid",
  "scope": { "chunk_ids": ["uuid","uuid"] },  // or { goal_id }
  "format": "flashcards",                      // flashcards | mcq | mindmap | glossary | summary
  "count": 10,
  "language": "en"
}
→ 200 { "items":[ ... ] }
```

---

## 7. Actions (S4)

```http
GET  /v1/actions/today?tribe_id=uuid    → 200 { "actions":[ ≤5 ] }
POST /v1/actions/complete
{
  "action_id":"uuid",
  "sentiment":"neutral",
  "effort_minutes":22,
  "evidence_url":null,
  "notes":"Got it on second try."
}
→ 200 {
  "tokens_awarded":10,
  "balance":120,
  "next_action": { ... } | null
}
```

### 7.1 Empathy recovery

```http
POST /v1/actions/struggle
{ "action_id":"uuid" }
→ 200 {
  "recovery_action":{ ... },
  "message_md":"That accent is genuinely tricky..."
}
```

The server inserts an `empathy_events` row, generates a simpler action via the Empathy Coach agent, awards `recovery_token` reservation (paid on completion).

---

## 8. Tokens & leaderboard

```http
GET /v1/tokens/balance?tribe_id=uuid          → { "balance": 230 }
GET /v1/tokens/ledger?tribe_id=uuid&limit=50  → { "items":[...] }
GET /v1/tribes/:id/leaderboard                → { "rows":[{user_id,balance}] }
```

Cross-tribe leaderboards do not exist. By design.

---

## 9. Visio (WebRTC)

```http
POST /v1/visio/sessions
{ "tribe_id":"uuid", "scheduled_for":"2026-05-08T10:00:00Z" }
→ 201 { "id":"uuid", "join_token":"..." }

POST /v1/visio/:id/start
→ 200 { "ice_servers":[...], "room":"..." }

POST /v1/visio/:id/transcripts
{ "segments":[{"start_ms":0,"end_ms":4500,"text":"..."}] }
→ 202

POST /v1/visio/:id/summarise
→ 202 { "trace_id":"..." }   # async; result in visio_sessions.summary_md
```

WebRTC signalling uses Supabase Realtime channel `visio:{id}:signal`. ICE servers via Twilio/Cloudflare TURN.

---

## 10. Companion (Solo only)

```http
POST /v1/companion/turn
{ "tribe_id":"uuid", "context":"action_struggle", "ref_id":"uuid" }
→ 200 { "message_md":"I bombed mine yesterday too 😅...", "thread":"companion:..." }
```

The companion never appears in `coach.turn` outputs and vice versa.

---

## 11. Realtime channels (Supabase Realtime / Phoenix)

| Channel | Events |
|---------|--------|
| `tribe:{id}:chat` | `message.new`, `message.token` (streaming) |
| `tribe:{id}:alignment` | `vote.update`, `state.transition` |
| `tribe:{id}:presence` | Phoenix presence diff |
| `tribe:{id}:kg_updates` | `node.added`, `edge.added` |
| `user:{id}:notifications` | `nudge`, `recovery_offered`, `badge_minted` |
| `visio:{id}:signal` | `offer`, `answer`, `candidate` |

Client snippet:
```ts
const channel = supabase.channel(`tribe:${id}:chat`);
channel.on("broadcast", { event: "message.token" }, p => append(p));
await channel.subscribe();
```

---

## 12. Admin & GDPR

```http
GET  /v1/me                                    # profile
PATCH /v1/me                                   # update profile
GET  /v1/me/export                             # GDPR export (zip)
POST /v1/me/delete                             # GDPR delete (async, with grace period)
GET  /v1/admin/orgs/:id/dashboard              # aggregate, anonymised
```

Admin endpoints require `app_role = 'admin'` and explicit `X-Admin-Reason` header (audit-logged).

---

## 13. Rate limiting & quotas

| Endpoint group | Limit |
|----------------|------:|
| `/auth/*` | 10/min/IP |
| `/coach/turn` | 30/min/user, 200/day/tribe |
| `/rag/query` | 60/min/user |
| `/resources/upload` | 20/day/tribe, max 25 MB/file, max 200 MB/tribe |
| `/visio/summarise` | 5/day/tribe |
| Default | 60/min/user |

429 responses include `RateLimit-*` headers and `Retry-After`.

---

## 14. Streaming protocol details

- All LLM-bound endpoints use **Server-Sent Events** with retry envelope (`id:` + `retry:` lines).
- Each `event: token` carries `<= 80 chars`. Receivers must handle out-of-order delivery with `seq`.
- `event: error` provides Problem Details JSON + recoverable flag.
- The same response is mirrored to the realtime `tribe:{id}:chat` channel (broadcast) so other tribe members watch the typing.

---

## 15. Webhooks (outgoing, optional)

If a tribe is part of an organisation with `webhook_url` configured, AGORA POSTs:

| Event | Payload |
|-------|---------|
| `tribe.state_transition` | `{tribe_id, from, to, at}` |
| `goal.achieved` | `{tribe_id, goal_id, criteria, vc_url}` |
| `user.empathy_triggered` | `{tribe_id, user_id, trigger}` (anonymised) |

Webhooks are signed with HMAC-SHA256 (`X-AGORA-Signature`) and retried with exponential backoff up to 24 h.

---

## 16. Mock-mode

All endpoints honour `?mock=1` (or `USE_MOCKS=true` env var) to return deterministic fixtures from `supabase/functions/_mocks/`. SSE streams are simulated with realistic timing. This guarantees zero external dependencies during demos.

---

See [07_AGENT_ARCHITECTURE.md](07_AGENT_ARCHITECTURE.md) for what runs *behind* `/coach/turn`, `/rag/query`, `/actions/struggle`, and `/companion/turn`.
