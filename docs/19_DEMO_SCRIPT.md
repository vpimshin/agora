# 19 — Demo Script (Hackathon, ~7 minutes)

> Mock-mode forced on. Network resilient. Three personas, one heart-moment, one technical wow.

## 0. Setup (before stage)

- Demo URL pre-loaded in two browsers (left = Amina, right = Lisbon trio member).
- Laptop on hotspot **and** on local stage Wi-Fi; service worker primed; mocks on.
- Slides pre-positioned at the closing one.
- Reset run: `make demo-reset` — fresh canonical state in < 5 s.

---

## 1. The hook (0:00 – 0:30)

> *"Coaching changes lives. But a great coach costs $200 an hour and lives in the wrong city. AGORA puts a Socratic coach, a curated library, and a tiny accountability tribe in your pocket. Let me show you."*

Open splash → click "Try it".

---

## 2. Amina's first 60 seconds (0:30 – 2:00)

Persona: 17-y-o in Cairo wants to learn Python.

- **Solo onboarding** — pick Arabic; UI flips RTL. (Wow #1: built-in i18n.)
- **Coach (S1):** Amina types "I want to learn Python."
  - Coach paraphrases, breathing-pause dot, then asks **one** question — never answers.
  - Two more turns; the **alignment widget** confirms with the AI Companion.
- Transition badge animates: **S1 → S2**.

Beat: "Notice — the Coach never gave Amina the answer. That's the whole product philosophy."

---

## 3. Criteria → Resources → first action (2:00 – 3:30)

- **S2 Criteria:** Amina drafts "I'll know Python when I can build a calculator."
  - Verifier flags it as too vague; suggests measurable rephrasings.
  - Amina picks one: "I can write a CLI that handles +, −, ×, ÷ and prints results." Verifier: ✓ binary.
- **S3 Resources:** Curator suggests three free open-licence sources. Amina keeps two.
  - KG mini-map renders nodes (Wow #2: GraphRAG visible).
- **S4 Today:** swipe deck appears; first card: "Read the *Python REPL Basics* chunk — 12 min."
  - Amina swipes right → starts.

---

## 4. The empathy moment (3:30 – 5:00)

> *"This is the moment most ed-tech gets wrong."*

- Amina swipes left on the next card (struggle signal). She picks 😔 in the sentiment prompt.
- The palette **shifts to calm cyan**. The Empathy banner slides in.
- Recovery action appears: "Read this gentle one-pager (5 min) — earn **+15 ⭐**."
- Amina taps Start → completes → **+15 ⭐** awarded with a tasteful animation.
- Token ledger pill increments.

Beat: *"Notice — recovery pays **more** than the standard action. We don't shame struggle. We pay for the bravery of trying again."*

---

## 5. Tribe & alignment (5:00 – 6:00)

Switch to second browser: the **Lisbon trio** is converging on a learning goal.

- Three avatars in real-time presence; alignment widget shows 2/3 aligned.
- Type a tweak in browser 2; browser 1 reflects via Realtime in < 200 ms.
- Third member's halo flips green; **convergence animation** plays; transition badge S1→S2.

Beat: *"Aligned, accountable, asynchronous. A coach for them collectively."*

---

## 6. The technical wow (6:00 – 6:30)

Open Grafana panel (cached image acceptable):

- Coach trace with paraphrase span + retrieval span + guardrail span.
- Verifier eval gate: "Subjective FP rate 3.1 % — gate green."
- Token ledger anomaly check: "0 anomalies in 7 d."

Beat: *"Behind every Coach reply: a redacted prompt, a tribe-isolated retrieval, a guardrail chain, and an evaluation that gated this very build."*

---

## 7. Close (6:30 – 7:00)

> *"Free. Open. Audit-friendly. Verifiable Credentials when you finish a goal — yours to take anywhere. A platform that profits when you learn, not when you scroll."*
>
> *"AGORA — world-class coaching for everyone, everywhere. Thank you."*

---

## Risk plan

| Risk | Mitigation |
|------|------------|
| Wi-Fi drops | mocks on; fully local-runnable demo (`make local-demo`) |
| Browser hiccup | second laptop pre-loaded as hot spare |
| Animation jank | `prefers-reduced-motion` toggle works; fall back if needed |
| Time creep | sections 4 + 5 are non-negotiable; cut section 3 detail first, then 6 |

---

## Speaker notes

- **Cadence:** slow on the empathy beat. That's where hearts move.
- **Eyes:** look up after each persona transition, not into the screen.
- **Hands:** away from keyboard during empathy beat; let it land.
- **Numbers to drop only if asked:** 30 % goal achievement target, +15⭐ recovery vs +10⭐ standard, 100 % "never reveal answer" coach gate.

---

See [11_EMPATHY_LOOP.md](11_EMPATHY_LOOP.md), [10_GAMIFICATION_AND_TOKENS.md](10_GAMIFICATION_AND_TOKENS.md), [02_PERSONAS_AND_JOURNEYS.md](02_PERSONAS_AND_JOURNEYS.md) for the substance behind the script.
