# 10 — Gamification & Token Economy

> Tokens are a feedback channel, not a currency. Every line of code in this layer must serve learning, not engagement-for-engagement's-sake.

## 1. Principles

1. **Abundant by design.** Tokens are easy to earn; the goal is positive reinforcement, not scarcity.
2. **Empathy outranks streaks.** Recovery rewards are *higher* than standard rewards.
3. **Privacy first.** Leaderboards exist only inside a tribe; cross-tribe ranking is forbidden.
4. **Append-only ledger.** Every awarded token is an immutable transaction. The "balance" is a derived view.
5. **No purchase, no fiat, no crypto.** Tokens are non-monetary. Goal completion mints a Verifiable Credential, not coins.

---

## 2. The token ⭐

Single denomination: **stars (⭐)**. UI shows them as filled glyphs.

### Earning rates (v1 defaults — configurable)

| Action | ⭐ |
|--------|---:|
| Daily action completed (any sentiment) | 10 |
| Daily action completed with `flow` sentiment | +2 bonus |
| Recovery action completed | **15** (intentionally higher) |
| Goal criterion verified (Yes) | 50 |
| Full goal achieved (all criteria Yes) | 200 |
| Peer-review approval given | 5 |
| 7-day streak | 25 (capped weekly) |
| Joining a tribe | 20 (one-time per tribe) |

### Why are recovery tokens *more* than standard?

Because the moment a learner is struggling is precisely when external validation matters most. Penalising struggle is the EdTech industry's biggest mistake. AGORA inverts it: **the reward for showing up after a setback is the largest of the daily rewards.**

---

## 3. Ledger architecture

```sql
-- See 05_DATA_MODEL.md
-- Append-only via SECURITY DEFINER RPC `award_tokens(...)`
-- No UPDATE / DELETE permitted via RLS
```

The ledger is the source of truth. The materialised view `token_balances` is refreshed on each award. Auditors can replay the entire economy at any past timestamp.

### Double-entry option (future)

When/if AGORA introduces *spending* (e.g., unlocking a stretch goal), the ledger transitions to true double-entry with `from_account` and `to_account` columns. Today, every entry credits a single user.

---

## 4. Leaderboard

A tribe leaderboard is a `SELECT user_id, balance FROM token_balances WHERE tribe_id = ?` rendered as a list with avatars and bars.

### Anti-toxicity rules (UI-enforced)

- No "rank" column — only relative bars.
- No public shaming; the lowest balance is *not* highlighted.
- "You're catching up to <name>!" copy is forbidden.
- The user can hide their balance from the tribe at any time (replaced by `—`).

---

## 5. Streaks

A streak is **shown** but never weaponised. Specifically:

- Losing a streak does not produce a notification.
- "You broke your streak" copy is banned.
- Recovery actions count toward streaks, identically to standard actions.
- Streaks reset gently after 48 h of inactivity, not 24 h.

---

## 6. Goal completion → Verifiable Credential

When all `success_criteria.met = true`:

1. The Curator agent compiles a "completion summary" (artefacts, evidence, mastery).
2. The system mints a **W3C Verifiable Credential** following the **Open Badges 3.0** spec.
3. The badge is stored in `verifiable_credentials` table and presented to the user with a download (JSON-LD + signed JWT) and a public verification URL.
4. Rendering uses the standard Open Badges PNG template with metadata embedded.

```json
{
  "@context": ["https://www.w3.org/ns/credentials/v2", "https://purl.imsglobal.org/spec/ob/v3p0/context.json"],
  "type": ["VerifiableCredential","OpenBadgeCredential"],
  "issuer": "https://agora.app/issuers/agora-v1",
  "validFrom": "2026-04-30T12:00:00Z",
  "name": "Career Switcher: REST API Achievement",
  "credentialSubject": {
    "id": "did:key:z6Mk...",
    "achievement": {
      "name": "Built and deployed a CRUD REST API",
      "criteria": { "narrative": "..." },
      "alignments": [],
      "skills": ["python","flask","rest","jwt"]
    }
  },
  "proof": { "type":"DataIntegrityProof","cryptosuite":"eddsa-rdfc-2022", "...":"..." }
}
```

The user owns the credential — exportable, presentable on LinkedIn, verifiable without AGORA.

---

## 7. Friendly competitions (opt-in, v2)

In v1, competition is **intra-tribe only**. In v2, two tribes may opt in to a paired challenge:

- Both tribes set the same goal class.
- A neutral metric (e.g., number of completed actions over 30 days) is compared.
- Both tribes earn a shared bonus on completion regardless of who "wins" — the team called this "friendly Olympics" in the discovery sessions.
- Either tribe can opt out at any time without penalty.

**Cross-tribe data sharing is whitelisted to aggregates only — never per-user data.**

---

## 8. Anti-pattern audit (what we explicitly do NOT do)

| Anti-pattern | Why we reject it |
|--------------|------------------|
| Daily-streak FOMO push | Punishes life events, harms mental health |
| Pay-to-skip token packs | Turns motivation extrinsic |
| Public global leaderboard | Privacy + comparison harm |
| Streak-loss notifications | Shame, well-documented dropout cause |
| Variable-ratio reward dopamine loops (slot-machine style) | Engagement ≠ learning; ethically dubious |
| Tokens that decay over time | Punitive, conflicts with empathy principle |

---

## 9. Configuration

All token amounts and triggers live in `config/economy.json`, version-controlled, reviewed by an "Economic Steward" (a designated team role). No code change required to retune.

```json
{
  "earn": {
    "action_complete": 10,
    "flow_bonus": 2,
    "recovery_action": 15,
    "criterion_met": 50,
    "goal_complete": 200,
    "peer_review": 5,
    "weekly_streak": 25
  },
  "caps": { "daily_max": 100, "weekly_streak_cap": 25 }
}
```

---

## 10. Telemetry

Every award is traced (Langfuse → eval). KPIs:

- Median ⭐/user/day (target ~30).
- Recovery / standard award ratio (target 0.10–0.20 — empathy is real but not the majority).
- Streak length distribution (target: long tail with no cliff at 7).
- Goal-completion-to-VC-mint median latency (target < 60 s).

---

## 11. Future explorations (carefully)

- **Soulbound tokens** (non-transferable on a permissioned chain) for tamper-evident credentials, only if it materially benefits learners (today: probably not).
- **Token gifting** within tribe — peer recognition.
- **Causes / charity** — convert ⭐ to a real-world positive externality (e.g., a partner NGO contribution per N tokens). Requires legal review.

---

See [11_EMPATHY_LOOP.md](11_EMPATHY_LOOP.md) for the recovery-token mechanism in motion.
