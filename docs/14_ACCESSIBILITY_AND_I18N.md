# 14 — Accessibility & Internationalisation

> AGORA targets **WCAG 2.2 Level AA** and the **EU European Accessibility Act (EAA, June 2025)**. Accessibility is a non-negotiable ship gate.

## 1. Conformance targets

| Standard | Level | Status |
|----------|-------|--------|
| WCAG 2.2 | AA | Must pass on all critical paths |
| EN 301 549 v3.2.1 | — | Aligned where stricter than WCAG |
| ATAG 2.0 (authoring tools) | A | The Coach UI is an authoring surface |
| ARIA 1.2 | — | Authoring guidance |
| EAA 2025 | — | Annex I + Annex IV implementation |

Critical paths that must pass on every release: onboarding, S1→S5 happy path, action completion, review session, empathy recovery, voice flow.

---

## 2. POUR principles → AGORA practice

### 2.1 Perceivable

- All non-text content has `alt`. Decorative images use `alt=""`.
- Streaming chat exposes `aria-live="polite"` per message bubble; no announcement on every token (we coalesce by sentence).
- Captions on every WebRTC session (auto via Web Speech, manual edit allowed).
- Token reward animation has a non-motion equivalent: a short text confirmation "+10 ⭐ awarded".
- Empathy state palette switch passes contrast at AA+ (verified via build-time checks).

### 2.2 Operable

- Full keyboard parity: every gesture has a keyboard equivalent (see swipe deck in [12](12_UI_UX_SPEC.md)).
- Visible focus rings (3 px, offset 2 px, brand colour, never removed).
- No keyboard traps.
- 48 × 48 px minimum touch target on mobile (WCAG 2.5.5 Level AAA — we adopt anyway).
- Pointer gestures: any drag has a button equivalent.
- Time limits: there are no automatic time-outs except auth (idle 60 min, with a "Stay signed in" 30 s prompt).

### 2.3 Understandable

- Reading level: 8th grade in English; equivalent in localisations.
- Form errors are: identified (`aria-invalid`), described (`aria-describedby`), and recoverable.
- Page language: `<html lang="...">` set per locale.

### 2.4 Robust

- Semantic HTML first; ARIA only when no native element fits.
- Tested with NVDA, VoiceOver (iOS, macOS), TalkBack, and JAWS at minimum.

---

## 3. Specific feature contracts

### 3.1 Voice mode

- 🎤 push-to-talk on mobile, toggle on desktop.
- Live transcript displayed in real time and editable.
- Speech rate / pitch: configurable for TTS playback.
- Full keyboard alternative: spacebar press-and-hold mirrors push-to-talk.

### 3.2 Sentiment picker

Buttons are `<button>` with `aria-pressed`. Labels: "Struggle", "Neutral", "Flow". Emoji is decorative (`aria-hidden="true"`).

### 3.3 Swipe deck

Each card is a focusable region. While focused:
- `Enter` → accept
- `?` → struggle
- `Esc` → defer
On-screen instructions visible to keyboard users; hidden on touch.

### 3.4 KG mini-map

Provides a list-mode toggle: a flat, sorted list of nodes with parent-child indentation. Cytoscape canvas itself is `aria-hidden="true"`; the list is the accessible surface.

### 3.5 Realtime presence

Avatar status carries `aria-label="Amina, active now"` etc. Updates throttled to ≤ 1 announcement per 10 s per user.

---

## 4. Internationalisation

### 4.1 Locales — launch matrix

| Locale | Code | RTL | Notes |
|--------|------|-----|-------|
| English | `en` | No | Source-of-truth |
| Spanish (Latin America) | `es-419` | No | Demo (Lisbon trio) |
| Portuguese (Brazil) | `pt-BR` | No | Hackathon strategic |
| French | `fr` | No | EAA market |
| Arabic | `ar` | **Yes** | Demo (Amina) |
| Hindi | `hi` | No | Strategic |
| Swahili | `sw` | No | Demo (Mateo) |
| Indonesian | `id` | No | Strategic |

Day-1 must include `en`, `es-419`, `ar`, `sw`. Others stub-translated.

### 4.2 Stack

- **Format:** ICU MessageFormat for plurals, gender, ordinals.
- **Library:** `lingui` or `react-intl` (decision recorded in implementation phase).
- **Tooling:** Strings extracted via babel macro; translated via [Crowdin](https://crowdin.com) free open-source plan. AI machine translation as first pass; human review gate before launch.
- **Storage:** JSON catalogues per locale, lazy-loaded by route.

### 4.3 Right-to-left

- All component CSS uses logical properties (`margin-inline-*`, `padding-inline-*`, `inset-inline-*`).
- Icons that imply direction (arrows, back/next) flip via `[dir="rtl"]` selector.
- Numerals: keep western digits by default; allow user override to native digits.
- Testing: Arabic locale is a tier-0 test path in CI.

### 4.4 Text expansion

German, Russian translations can run +35 %. All container components must accommodate without truncation. Visual regression suite includes `de` snapshots.

### 4.5 Cultural & pedagogical adaptation

- Examples and metaphors localised (not just translated).
- Avoid idioms; if used, include a localiser note for substitutions.
- Empathic copy localised by native speakers — empathy rarely survives MT.
- Date/time/number formatting via `Intl.*`.

---

## 5. Accessibility testing

### 5.1 Automated

- `axe-core` runs in Storybook for every component.
- Playwright E2E injects axe and asserts no violations on critical paths.
- Lighthouse CI gate: a11y score ≥ 95.

### 5.2 Manual

- Per release: NVDA + Firefox, VoiceOver + Safari, TalkBack + Chrome.
- Keyboard-only smoke test (no mouse, no touch).
- 200 % zoom check.
- Forced-colours mode (Windows High Contrast).

### 5.3 With users

- Recurring testing with disabled users — paid, not free labour.

---

## 6. EAA 2025 documentation

We maintain a public [accessibility statement](https://example.com/a11y) covering:
- conformance level claimed,
- known issues with workarounds,
- contact route for issues (commitment: response within 5 business days),
- compatible AT,
- last audit date.

---

## 7. Cognitive accessibility

- Plain-language toggle (planned tier-2): rewrites all UI copy at a 5th-grade level.
- Pause-on-hover for any animated content.
- Time-out warnings 30 s before any auto-action.
- "Distraction-free" mode hides leaderboards and notifications.

---

See [12_UI_UX_SPEC.md](12_UI_UX_SPEC.md) for screens, [13_DESIGN_SYSTEM.md](13_DESIGN_SYSTEM.md) for tokens, [15_SECURITY_AND_PRIVACY.md](15_SECURITY_AND_PRIVACY.md) for related privacy obligations.
