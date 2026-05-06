# 20 — Build Roadmap

> Tier 0 = ship. Tier 1 = next. Tier 2 = vision. Cut from the top under pressure.

## Phase 0 — Foundation

**Goal:** developer environment, infra, CI green on a "hello tribe" walking skeleton.

- Repo bootstrap: monorepo (`apps/web`, `supabase/`, `packages/design-system`, `packages/agents`).
- Supabase project (local + staging).
- Auth (magic link) + a single dummy table with RLS.
- CI pipeline: lint, type-check, unit, deploy preview.
- Storybook + Tailwind v4 tokens.
- Decision log seeded with ADRs 0001–0005.

**Done when:** PR merges trigger preview deploy and a logged-in user can see their own row.

---

## Phase 1 — State Machine + Coach (Tier 0)

**Goal:** S0 → S5 with the **Coach (S1)** and **Verifier (S2)** functioning, mock-mode capable.

- Schema for tribes, members, goals, criteria, messages, transitions.
- RLS policies + pgtap suite.
- Edge function `state.transition` with guards.
- Coach prompt + agent + Langfuse tracing.
- Verifier prompt + agent.
- Front-end: onboarding, S1 chat, S2 criteria editor, transition UI.
- Eval suites: "never gives the answer", "binary criterion FP rate".

**Done when:** Amina demo flow runs end-to-end on staging in mock-mode.

---

## Phase 2 — RAG + S3 Resources (Tier 0)

- Resource upload + ingestion pipeline (chunk + embed + extract entities).
- pgvector + AGE tables.
- Curator agent + retrieval API.
- KG mini-map (Cytoscape).
- Public-commons suggester.

**Done when:** S3 unlocks once ≥ 1 resource is ready; Curator citations precision ≥ 0.9 in evals.

---

## Phase 3 — Pedagogy + S4 Actions + Tokens (Tier 0)

- Pedagogy generator agent.
- Action card swipe deck + completion flow.
- Token ledger + RPC + materialised balance.
- Standard award path (+10 ⭐).
- FSRS-6 scheduler + review screen.

**Done when:** action acceptance rate dashboard wired; ledger anomaly check passes.

---

## Phase 4 — Empathy Loop (Tier 0)

- Sentiment picker + struggle-signal triggers.
- Empathy agent + recovery action template.
- Palette shift; recovery card UI.
- +15 ⭐ recovery RPC + audit.

**Done when:** synthetic struggle journey resolves with +15 ⭐ awarded; "no toxic positivity" eval passes.

---

## Phase 5 — Tribe collaboration & Realtime (Tier 0)

- Tribe creation + invite codes.
- Realtime presence + alignment widget.
- Multi-user S1 convergence.
- Tribe leaderboard (in-tribe only).

**Done when:** Lisbon trio journey passes E2E with two browsers.

---

## Phase 6 — Polish, A11y, i18n (Tier 0 ship gate)

- Locale catalogues for `en`, `es-419`, `ar`, `sw`.
- RTL audit; reduced-motion audit; NVDA + VoiceOver pass.
- Lighthouse a11y ≥ 95 on critical paths.
- Privacy notice, GDPR endpoints (`/me/export`, `/me/delete`).
- Demo seed + reset script.

**Done when:** demo script (doc 19) runs cleanly, twice in a row, on stage Wi-Fi simulation.

---

## Phase 7 — Tier 1 expansion

- WebRTC Visio sessions + captions + summary.
- Voice mode (Web Speech default, Whisper fallback).
- Mentor role + read-only oversight.
- Verifiable Credentials issuance (Open Badges 3.0).
- Public-commons curation portal.

---

## Phase 8 — Tier 2 (vision)

- Federated learning across tribes (privacy-preserving).
- Self-host bundle (docker compose).
- Plugin SDK for community-built agents.
- Plain-language toggle (cognitive a11y).
- Mobile native shell (Capacitor) for stores where web isn't enough.

---

## Cut-line discipline

If schedule slips, cut **in this order**, never above:
1. Tier 2 items.
2. Tier 1 items.
3. Locale parity beyond `en` and `ar`.
4. Verifier richness (keep binary check; defer suggestion quality).
5. KG mini-map (degrade to a flat list).

**Never cut:**
- The Empathy loop.
- The "never gives the answer" Coach gate.
- RLS / tribe isolation.
- A11y critical paths.
- Mock-mode for the demo.

---

## Hackathon-window plan

If only one hackathon window is available, build in this **strict** order:

1. Foundation + Phase 1 (Coach + Verifier in mock-mode, single user).
2. Phase 4 (Empathy) — because it is the **emotional payoff** of the demo.
3. Phase 3 (Tokens, minimal) — to make Empathy resolve with a visible +15 ⭐.
4. Phase 5 (Realtime + tribe), only as far as a 2-browser alignment demo.
5. Phase 6 polish — RTL Arabic, mock-mode hardening, demo reset.

Skip RAG, FSRS, Visio, mentor, VC for the demo. Stub them in copy and roadmap.

---

## Definition of "shippable" for prod

- All Tier 0 phases done.
- All Tier 0 evals at threshold.
- A11y AA on critical paths.
- DPIA signed; ROPA filled.
- 7-day soak on staging with synthetic users; no critical incidents.
- On-call rota established; runbooks current.

---

See [02_PERSONAS_AND_JOURNEYS.md](02_PERSONAS_AND_JOURNEYS.md), [01_VISION_AND_PRD.md](01_VISION_AND_PRD.md) for tier definitions, [19_DEMO_SCRIPT.md](19_DEMO_SCRIPT.md) for the staged story.
