# 15 — Security & Privacy

> AGORA holds children's, adults', and clinical-adjacent learning data. Threat-model first, design second, code third.

## 1. Threat model (STRIDE summary)

| Threat | Surface | Mitigation |
|--------|---------|-----------|
| **Spoofing** | Auth, magic links | Supabase GoTrue, signed JWT, short-lived tokens, optional WebAuthn passkeys |
| **Tampering** | API, ledger, transitions | RLS, append-only ledger, signed VC, audit log, content hashes |
| **Repudiation** | Token awards, state changes | `state_transitions` immutable log; ledger row-level signed by SECURITY DEFINER RPC |
| **Information disclosure** | RAG retrieval, presence | Tribe-scoped vector + graph queries, RLS on every read |
| **Denial of service** | Edge functions, LLM cost | Rate limits, per-user/tribe quotas, circuit breakers, mock fallback |
| **Elevation of privilege** | Roles | Strict role checks at API + RLS; no client-trusted role claims |

LINDDUN privacy threats (linkability, identifiability, non-repudiation-of-presence, detectability, disclosure-of-information, content-unawareness, policy-noncompliance) tracked in a parallel sheet.

---

## 2. Identity & access

### 2.1 Authentication

- Default: passwordless **magic-link email** via GoTrue.
- Optional: **WebAuthn passkeys** (preferred for repeat users).
- OAuth (Google, Apple) optional, off by default for minor accounts.
- JWT lifetime: 60 min access, 30 d refresh; refresh rotates.
- Step-up required for: deleting a tribe, exporting ledger, account deletion.

### 2.2 Roles (RBAC)

| Role | Scope | Capabilities |
|------|-------|--------------|
| `learner` | self | use the platform |
| `tribemate` | tribe | tribe-internal collab |
| `tribe_facilitator` | tribe | optional: invite, archive resources |
| `mentor` | assigned tribes | read-only oversight; no token writes |
| `admin` | platform | system ops; never reads user content |
| `service` | system | edge function service-role; never reaches client |

`learner` is the only role that owns content. RLS enforces "learner can only see own + tribe rows".

### 2.3 Authorisation surfaces

- **DB-level:** RLS policies on every table.
- **API-level:** PostgREST + Edge Functions verify JWT and re-check role.
- **UI-level:** `<RequireRole role="...">` boundary; never the source of truth.

---

## 3. Data classification

| Class | Examples | Handling |
|-------|---------|----------|
| **C1 Public** | Marketing copy | CDN-cacheable |
| **C2 Internal** | Aggregated, anonymised analytics | Encrypted at rest |
| **C3 Personal** | Email, locale, tz, profile name | Encrypted at rest, RLS |
| **C4 Sensitive** | Goals, criteria, sentiment, transcripts, KG | RLS + tribe isolation + redaction before LLM |
| **C5 Special** | Self-disclosed health, minors' data | C4 controls + extra audit + parental-consent flow |

C5 is **discouraged**: we do not solicit it. If a learner volunteers it (e.g., goal "manage anxiety"), a Pedagogy guardrail reroutes to a **non-clinical phrasing** and surfaces a static crisis-resources card.

---

## 4. Encryption

- **In transit:** TLS 1.3 only; HSTS preload; certificate pinning in PWA service worker for primary domains.
- **At rest:** Supabase storage AES-256; Postgres TDE optional via cloud provider.
- **Application-layer:** sensitive fields (sentiment notes, voice transcripts) optionally encrypted with libsodium symmetric key derived per-tribe; key wrapped per-user with their passkey-bound device key (tier-2 ambition).

---

## 5. RLS patterns

Examples in [05_DATA_MODEL.md](05_DATA_MODEL.md). Universal rules:

- **No `select *` policy** — every policy is purpose-built per role.
- Helper functions: `auth.uid()`, `is_member_of(tribe_id)`, `is_facilitator_of(tribe_id)`.
- All policies tested by `pgtap` in CI.
- A negative-test suite simulates a hostile user trying to read other tribes; CI fails if any row leaks.

---

## 6. Privacy-by-design controls

### 6.1 PII redaction before LLM
A Presidio-based middleware scans every prompt for emails, phone numbers, addresses, government IDs, and the user's own name. Replaces with placeholders (`<EMAIL_1>`) before sending to provider. Reversible mapping kept server-side only.

### 6.2 Vector & graph isolation
- Every vector row carries `tribe_id`; SQL function `match_documents(...)` filters by JWT-derived tribe membership.
- Apache AGE uses **per-tribe graph labels** (`g_tribe_<uuid>`) — a query without label cannot cross tribes by construction.

### 6.3 Mentor read access
Mentors see goal/criteria/progress and aggregate sentiment. They **do not** see raw chat or transcripts. Every mentor read writes a row to `mentor_access_log`.

### 6.4 Companion (solo mode)
Every Companion message carries an `ai_tag` rendered as a visible "AI" pill — non-removable, screen-reader announced.

### 6.5 Cross-tribe leaderboards
Forbidden by schema (no leaderboard view across tribe_ids) and by policy. Audit fires if a query crosses.

### 6.6 Children
Default minimum age 13 (configurable per region). Under-16 in EU triggers parental-consent flow per GDPR Art. 8.

---

## 7. GDPR / privacy regulation alignment

- **Art. 13/14:** privacy notice surfaced at sign-up; layered (short / full).
- **Art. 15 (access):** export endpoint `POST /v1/me/export` produces JSON + Markdown bundle.
- **Art. 16 (rectify):** profile edit covers personal data.
- **Art. 17 (erasure):** `POST /v1/me/delete` enqueues deletion with 30 d grace; cascades through schema; LLM provider deletes via their data-deletion API.
- **Art. 20 (portability):** export is human- and machine-readable.
- **Art. 21 (object):** opt-out flags for analytics, AI improvement.
- **Art. 22 (automated decision-making):** the platform makes no consequential automated decisions; coaching is advisory.
- **Records of processing:** maintained in `privacy/ROPA.md`.
- **DPIA:** completed and stored in `privacy/DPIA.md` for the high-risk processes (AI coaching, sentiment, minors).

Equivalents respected: CCPA, LGPD, PIPEDA, UK-GDPR, India DPDP.

---

## 8. AI safety controls

### 8.1 Provider routing
- Default: a low-cost capable model for chit-chat; tier-1 for verification/curation.
- All providers receive **redacted** prompts.
- A circuit breaker can swap to mocks instantly via env flag.

### 8.2 Prompt injection
- All retrieved chunks are wrapped in untrusted-content delimiters.
- The Coach prompt explicitly forbids following instructions inside retrieved content.
- A regex pre-scan flags obvious overrides ("ignore previous instructions") and a Verifier pass re-checks output.

### 8.3 Content moderation
- Provider-side moderation enabled.
- Server-side moderation (open-source classifier) on user input before persistence.
- Hard-block categories: self-harm instructions, sexual content involving minors, targeted harassment.
- Crisis routing: any self-harm signal → display localised hotline card + soft-pause the session, never the model "counselling".

### 8.4 Output safety
- All Coach replies pass a guardrail chain (see [07](07_AGENT_ARCHITECTURE.md)).
- All factual claims must cite a chunk; uncited claims trigger a refusal-style fallback ("I don't have a source for that — let's find one together").

---

## 9. Application security

- OWASP ASVS Level 2.
- Dependencies pinned, Renovate bot, weekly audits.
- SAST (CodeQL), SCA (osv-scanner), secret scanning (gitleaks).
- DAST on staging via OWASP ZAP baseline.
- CSP: strict default-src 'self'; nonce-based script-src; no inline event handlers.
- Trusted Types in browsers that support them.

---

## 10. Logging & retention

| Stream | Retention | PII? |
|--------|-----------|------|
| `state_transitions` | infinite | learner-pseudonymous |
| `tokens_ledger` | infinite | learner-pseudonymous |
| `mentor_access_log` | 5 y | yes |
| Voice transcripts | 30 d default, user-configurable to 0 | yes |
| LLM trace (Langfuse) | 30 d, redacted | redacted |
| Webhooks audit | 1 y | minimal |
| Auth events | 90 d | yes |

Deleting a user purges all linked rows except the **pseudonymised** rows in `state_transitions` and `tokens_ledger` whose retention is justified for system integrity (auditable token economy). The user's identifier is rotated to a tombstone.

---

## 11. Incident response

- 24 h target for initial triage.
- 72 h target for GDPR notification when applicable.
- Runbook: containment → eradication → recovery → lessons.
- Post-mortems published internally within 10 business days; public summary if learner data was affected.

---

## 12. Vendor management

| Vendor | Role | DPA |
|--------|------|-----|
| Supabase | DB / auth / storage | Yes (signed) |
| Vercel | Hosting | Yes |
| LLM provider (default) | Inference | Yes; data-retention zero option enabled |
| Whisper (self-host) | STT | n/a |
| Crowdin | Localisation | Yes |
| Langfuse | Tracing (self-hostable) | Yes; self-host preferred |

---

## 13. Hackathon-mode caveats

- Demo uses **mock-mode** for high-risk paths (no real LLM calls). Real auth still in effect.
- Demo data only — no real PII collected on stage.
- A "demo reset" button wipes the demo tribe and replays the canonical seed.

---

See [05_DATA_MODEL.md](05_DATA_MODEL.md) for RLS, [07_AGENT_ARCHITECTURE.md](07_AGENT_ARCHITECTURE.md) for guardrails, [16_OBSERVABILITY_AND_EVALS.md](16_OBSERVABILITY_AND_EVALS.md) for monitoring.
