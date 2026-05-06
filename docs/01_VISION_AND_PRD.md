# 01 — Vision & Product Requirements

## 1. Vision

> **Deliver world-class, agentic coaching to every learner — especially those without geography or budget on their side — through a tribe-based, cognitively-coached, gamified mobile experience.**

### 1.1 Mission statements

| Pillar | Statement |
|--------|-----------|
| **Equity** | A learner in a small village must receive the same quality of coaching as a Stanford MBA. |
| **Agency** | The platform never gives answers. It enables the learner to find their own. |
| **Belonging** | Solo learning is lonely; tribes of 3–4 build trust, momentum, and accountability. |
| **Empathy** | Failure is information, not punishment. The system rescues, never shames. |
| **Privacy** | A tribe's data, goals, and struggles belong only to that tribe. |

### 1.2 Anti-vision (what AGORA explicitly is *not*)

- ❌ A ChatGPT wrapper that answers questions.
- ❌ A LMS with playlists of videos.
- ❌ A productivity app with todos.
- ❌ A social network with public feeds.
- ❌ A subscription pyramid optimised for engagement metrics.

---

## 2. Target audience

### Primary
- **Underserved adult learners** (18–55) in rural areas, minority communities, post-conflict regions, and low-income urban districts of EMEA, LATAM, and Southeast Asia.
- Devices: budget Android (Snapdragon 4-series), 320–414 px viewports, throttled 3G.
- Languages at launch: **English, French, Spanish, Arabic** (RTL).

### Secondary
- Small professional pods (3–4) inside SMBs that cannot afford corporate L&D platforms.
- Self-improvement individuals seeking goal-oriented coaching (fitness, habit, language, career).

### Tertiary (future)
- NGO-deployed cohorts (refugee education, women's literacy programs).
- High-school study groups.

---

## 3. Functional requirements

Each requirement carries an ID. Requirements marked **MUST** are tier-0 (hackathon scope). Requirements marked **SHOULD** are tier-1. **MAY** are tier-2.

### 3.1 Onboarding & identity

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-ON-01 | The system **MUST** authenticate via email magic link (passwordless). | 0 |
| FR-ON-02 | The system **MUST** support an anonymous "guest" mode that upgrades to a real account without data loss. | 0 |
| FR-ON-03 | The system **MUST** offer Solo or Tribe mode at first launch. | 0 |
| FR-ON-04 | In Tribe mode, the system **MUST** generate a 6-character invite code with TTL = 24 h. | 0 |
| FR-ON-05 | A tribe **MUST** be limited to between 1 and 4 humans. (Solo = 1 human + Synthetic Companion.) | 0 |
| FR-ON-06 | The system **SHOULD** support Google OAuth as a secondary provider. | 1 |
| FR-ON-07 | The system **MAY** support WebAuthn/passkeys. | 2 |

### 3.2 Cognitive coaching (S1)

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-CC-01 | The Cognitive Coach **MUST** never produce a declarative answer to a goal-formation question. | 0 |
| FR-CC-02 | Each agent turn **MUST** comprise exactly: paraphrase → pause indicator → open question. | 0 |
| FR-CC-03 | The system **MUST** stream agent output token-by-token via SSE. | 0 |
| FR-CC-04 | The system **MUST** support multi-user chat where every member sees every member's contribution in real time. | 0 |
| FR-CC-05 | Each member **MUST** vote "Aligned / Not yet" on the candidate goal; transition to S2 requires 100 % alignment. | 0 |
| FR-CC-06 | The system **MUST** produce the goal in **SMART(ER)** form: Specific, Measurable, Achievable, Relevant, Time-bound, Evaluated, Reviewed. | 0 |
| FR-CC-07 | The system **SHOULD** translate the final goal into the preferred locale of each tribe member. | 1 |

### 3.3 Success criteria (S2)

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-SC-01 | Every criterion **MUST** be a binary, provable Yes/No question. | 0 |
| FR-SC-02 | The system **MUST** reject subjective or unmeasurable criteria with the rationale shown to the user. | 0 |
| FR-SC-03 | A goal **MUST** carry between 1 and 5 success criteria. | 0 |
| FR-SC-04 | Each criterion **MUST** declare its verification method: `self_attestation`, `photo_proof`, `peer_review`, `quiz_score`, `external_log`. | 0 |
| FR-SC-05 | The system **SHOULD** support `photo_proof` with on-device redaction of faces and EXIF stripping. | 1 |

### 3.4 Resource curation (S3)

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-RC-01 | A tribe **MUST** be able to upload PDF, EPUB, plain text, Markdown, and URLs. | 0 |
| FR-RC-02 | The system **MUST** chunk, embed, and graph-link new resources within 60 s of upload. | 0 |
| FR-RC-03 | The Curator agent **MUST** offer at least three resources from the public commons (Wikipedia, OER, OpenStax) per goal. | 0 |
| FR-RC-04 | The user **MUST** be able to choose `summary`, `standard`, or `deep_dive` depth. | 0 |
| FR-RC-05 | All vector queries **MUST** be filtered by `tribe_id`; cross-tribe leakage is a P0 incident. | 0 |
| FR-RC-06 | The system **SHOULD** surface a knowledge-graph mini-map showing concept relationships. | 1 |

### 3.5 Daily actions & feedback (S4)

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-DA-01 | The system **MUST** propose at most 5 action items per day per learner. | 0 |
| FR-DA-02 | Each action **MUST** be completable in ≤ 25 minutes (one Pomodoro). | 0 |
| FR-DA-03 | Action cards **MUST** support swipe-right (accept), swipe-left (struggle), swipe-up (defer). | 0 |
| FR-DA-04 | On struggle, the Empathy Coach **MUST** offer a simpler alternative within 3 s. | 0 |
| FR-DA-05 | Every completion **MUST** append an immutable row to the token ledger. | 0 |
| FR-DA-06 | The system **SHOULD** send a daily push notification at the user's chosen time, configurable per tribe. | 1 |
| FR-DA-07 | The system **SHOULD** learn the user's optimal nudge time via a multi-armed bandit. | 2 |

### 3.6 Pedagogical generators

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-PG-01 | The system **MUST** generate flashcards (front/back) from any selected text. | 0 |
| FR-PG-02 | The system **MUST** schedule flashcard reviews using **FSRS-6**. | 0 |
| FR-PG-03 | The system **MUST** generate multiple-choice questions (MCQ) with 4 options and rationale per option. | 0 |
| FR-PG-04 | The system **SHOULD** generate mind-maps (Mermaid + interactive) and glossaries. | 1 |
| FR-PG-05 | The system **SHOULD** estimate concept mastery via Bayesian Knowledge Tracing (BKT). | 1 |

### 3.7 Gamification & token economy

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-GT-01 | The token ledger **MUST** be append-only and double-entry. | 0 |
| FR-GT-02 | Standard task completion **MUST** award 10 ⭐ tokens. | 0 |
| FR-GT-03 | Recovery tokens **MUST** award 15 ⭐ (higher than standard) to combat shame. | 0 |
| FR-GT-04 | A tribe **MUST** see a leaderboard of tokens within the tribe; cross-tribe leaderboards are forbidden. | 0 |
| FR-GT-05 | Goal completion **SHOULD** mint an Open Badge 3.0 verifiable credential. | 1 |
| FR-GT-06 | The system **MAY** support seasonal "tribe vs tribe" friendly challenges (opt-in). | 2 |

### 3.8 Synthetic Companion (Solo mode)

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-SY-01 | The Companion **MUST** be clearly labelled as AI in every message. | 0 |
| FR-SY-02 | The Companion **MUST NOT** instruct, coach, or correct. | 0 |
| FR-SY-03 | The Companion **MUST** simulate parallel struggles using the same goal context. | 0 |

### 3.9 Video & voice

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-VV-01 | The system **MUST** support voice dictation in chat via the Web Speech API. | 0 |
| FR-VV-02 | The system **SHOULD** support tribe video calls (WebRTC mesh up to 4 peers). | 1 |
| FR-VV-03 | The system **SHOULD** transcribe and summarise calls; summaries auto-ingest into the knowledge base. | 1 |

### 3.10 Empathy loop (cross-cutting)

| ID | Requirement | Tier |
|----|-------------|:---:|
| FR-EM-01 | The system **MUST** capture per-action sentiment via a 3-emoji scale (😞 / 😐 / 😄). | 0 |
| FR-EM-02 | Three consecutive 😞 ratings **MUST** trigger the Empathy Coach. | 0 |
| FR-EM-03 | Recovery tasks **MUST** never reference the user's previous failure in negative language. | 0 |

---

## 4. Non-functional requirements

| Category | Requirement |
|----------|-------------|
| **Performance** | First Contentful Paint < 1.5 s on 3G; LLM first-token latency p95 < 1.2 s. |
| **Availability** | 99.5 % uptime during the hackathon demo window; graceful degradation to mock-mode. |
| **Scalability** | Single Supabase project must support 10 k tribes / 40 k users without redesign. |
| **Accessibility** | WCAG 2.2 AA. EAA 2025 compliant. Full keyboard + screen-reader parity. |
| **Privacy** | GDPR-compliant. Right to export, right to be forgotten, all PII redactable. |
| **Security** | OWASP ASVS Level 2. Every table RLS-protected. CSP, HSTS, SRI enforced. |
| **Observability** | Every LLM call traced in Langfuse; every API call traced in OTel. |
| **Cost** | < $0.10 per active tribe per day at 100 daily messages. |
| **Internationalisation** | English, French, Spanish, Arabic at launch. RTL-correct. ICU MessageFormat. |
| **Mobile** | Installable PWA. Works offline for read + draft. Push notifications. |
| **Browser support** | Last 2 versions of Chrome, Firefox, Safari, Edge. iOS Safari 16+. |

---

## 5. Success metrics (post-launch)

| Metric | Target |
|--------|-------:|
| W1 retention (% returning on day 7) | ≥ 50 % |
| Goal completion rate (% achieving binary criteria) | ≥ 35 % |
| Average tribes per user | ≥ 1.2 |
| Median time-to-first-action-completion | ≤ 36 h |
| Empathy Loop trigger → recovery task completion | ≥ 70 % |
| Net Promoter Score | ≥ 45 |

---

## 6. Out of scope (v1)

- Native iOS / Android apps (PWA only).
- Marketplace for paid coaches.
- Public goal feeds, social following, profile pages.
- Synchronous classroom features (lectures, hand-raise).
- Payment processing — v1 is free with optional Stripe support reserved for v2.
- Blockchain tokens — v1 tokens are off-chain ledger entries. Verifiable Credentials use W3C VC, not crypto.

---

## 7. Glossary anchors used in this doc

See [GLOSSARY.md](GLOSSARY.md) for: *agentic*, *cognitive coaching*, *FSRS*, *GraphRAG*, *RLS*, *empathy loop*, *recovery token*, *tribe*, *synthetic companion*.
