# 12 — UI/UX Specification

> Mobile-first, one-thumb friendly, voice-capable, screen-reader-friendly. WCAG 2.2 AA throughout.

## 1. Information architecture

```
/                        → marketing splash (public)
/auth/login              → magic link
/auth/callback
/onboarding              → mode select, locale, tz
/tribe/new
/tribe/join
/t/:tribeId              → tribe shell (authenticated)
  /t/:id/coach           → S1 chat + alignment widget
  /t/:id/criteria        → S2 form + verifier feedback
  /t/:id/resources       → S3 upload, KG mini-map
  /t/:id/today           → S4 daily action deck
  /t/:id/review          → flashcard FSRS deck
  /t/:id/library         → resources, notes, glossary
  /t/:id/visio/:sid      → WebRTC room
  /t/:id/leaderboard
  /t/:id/settings
/me                      → profile, badges, GDPR
```

Routing: file-based, type-safe (TanStack Router). Authenticated routes wrapped in a `RequireAuth` boundary.

---

## 2. Layouts

### 2.1 Mobile shell (default)

```
┌──────────────────────────┐
│  ☰   AGORA  Tribe ▾   👤 │  ← top bar (sticky, glassmorphic)
├──────────────────────────┤
│                          │
│        VIEWPORT          │
│                          │
├──────────────────────────┤
│ 🏠 💬 ✅ ⭐ ⚙              │  ← bottom tab nav (5 tabs)
└──────────────────────────┘
```

- 5 tabs: Home (today), Coach (S1/S2 chat), Library (resources/review), Tokens, Settings.
- Top bar opens a tribe switcher (≤ 1.2 tribes per user expected; switcher is a simple sheet).
- Bottom nav uses 48 px tap targets (WCAG 2.5.5).

### 2.2 Tablet / desktop

Three-column with persistent sidebar (tribes), main column (active state's primary view), context column (alignment widget / leaderboard / KG mini-map). Collapses gracefully to mobile shell at < 900 px.

---

## 3. Screen-by-screen

### 3.1 Onboarding

- Step 0: locale picker (auto-detect + override).
- Step 1: "Solo or Tribe?" — two huge cards with emoji + 1-line description.
- Step 2 (Tribe): generate code OR enter code.
- Step 3 (Solo): name your Companion (optional, suggestion provided).
- Step 4: tz + daily anchor time.
- Total target: ≤ 90 s on 3G.

### 3.2 Coach (S1 / S2)

```
┌─ avatar grid (real-time presence) ──────┐
│  🟢 Amina   🟢 Yusuf   🟡 Sara   ⚪ Bot   │
├──────────────────────────────────────────┤
│  message bubbles                          │
│  agent messages render: paraphrase line, │
│  breathing pause dot (1.2 s),             │
│  question line                            │
│                                           │
├──────────────────────────────────────────┤
│  [ alignment widget — current draft v3 ]  │
│  ◉ ◉ ◯ ◯  3/4 aligned                    │
│  [I'm aligned]  [Not yet]                 │
├──────────────────────────────────────────┤
│  [ chat input + 🎤 voice + 📎 attach ]    │
└──────────────────────────────────────────┘
```

- Streaming SSE; tokens appear in real time for all members via Realtime broadcast.
- Voice: tap-and-hold mic, releases on lift; on-device Web Speech API for transcription, fallback to /v1/voice/stt.
- Empty state: a single, kind sentence inviting the first message.

### 3.3 Criteria (S2)

- One row per criterion. Each row:
  - the binary question text,
  - a chip showing `verification_method`,
  - the Verifier's last verdict (✓ binary or ⚠️ subjective + suggestion),
  - a delete handle.
- "+ Add criterion" CTA opens a focused sheet with:
  - free text input,
  - method selector,
  - live verdict from the Verifier as the user types (debounced 600 ms).

### 3.4 Resources (S3)

- Upload zone: drag-drop on desktop; tap to pick on mobile (file or camera).
- Resource list with `ingestion_status` chip (queued / processing / ready / failed).
- Public-commons suggestions cards: "We found 3 free resources for this goal — keep them?"
- KG mini-map (Cytoscape) renders below — clickable nodes reveal chunks.

### 3.5 Today (S4)

The headline screen. Stack of swipe cards à la Tinder, ≤ 5 cards.

```
┌─ Today, Tuesday May 6 — 4 of 5 ─┐
│                                  │
│  Action #2                       │
│  Read JWT refresh deep-dive      │
│  ⏱ 18 min · 🌶 Difficulty 3/5    │
│  📚 Citation: Flask Mega-Tutorial│
│                                  │
│  [ Start ]  [ Hint ]             │
│                                  │
│  ↩ defer  ←  →  ✓ accept         │
└──────────────────────────────────┘
```

- Swipe right → accept and start.
- Swipe left → struggle (triggers Empathy).
- Swipe up → defer to tomorrow.
- On completion: 3-emoji sentiment + optional notes + optional photo proof.
- Token reward animation: tasteful starburst, no sound by default.

### 3.6 Empathy state

- Recovery banner enters from bottom with calm cyan palette.
- Card replaces with a smaller, softer card showing the recovery action.
- Copy is empathic (see [11](11_EMPATHY_LOOP.md)).
- 15 ⭐ pill prominently displayed next to "When you finish".

### 3.7 Flashcard review

- Single card view: front shown, tap to flip.
- Bottom row: `Again | Hard | Good | Easy` with FSRS-predicted intervals shown as small text under each.
- Progress bar shows `n / queue_size`.
- Pomodoro auto-pause at 25 min with a gentle nudge.

### 3.8 Leaderboard

- Bars with avatars, balances. No rank numbers. No "below-the-cut" highlight.
- Pull-to-reveal "your weekly summary".

### 3.9 Visio (WebRTC)

- 4-up grid (or 1+3 speaker).
- Bottom dock: mute, camera, share, end, captions toggle, summary toggle.
- Real-time captions via Web Speech API (multilingual).
- Post-call: "Generate summary?" → shoves into Library.

### 3.10 Settings & profile

- Locale, timezone, daily anchor, sentiment-prompt cadence.
- Notification preferences (granular: nudges, recovery offers, visio invites).
- Privacy: "Hide my balance from tribe", export data, delete account.
- Badges (Verifiable Credentials) with download / share.

---

## 4. Interaction patterns

### 4.1 Real-time presence
Avatars carry a status dot: green (active < 30 s), yellow (active < 5 min), grey (offline). Powered by Phoenix Presence.

### 4.2 Optimistic UI
Every mutation flips the UI optimistically; on failure, a snackbar offers retry with the cached payload.

### 4.3 Streaming text
Agent messages animate token-by-token with `prefers-reduced-motion` honoured (instant render if reduced-motion is on).

### 4.4 Voice-first
- 🎤 button on every text input.
- Push-to-talk on mobile, toggle on desktop.
- Live caption display while recording.
- Transcript editable before send.

### 4.5 Empty / error / offline states
Every screen has an explicit empty state, a friendly error illustration, and an offline banner with last-sync time.

---

## 5. Accessibility (highlights — full spec in [14](14_ACCESSIBILITY_AND_I18N.md))

- All interactive elements reachable by keyboard.
- Focus rings visible (custom 3 px + offset).
- Skip-to-content link.
- ARIA live regions for streaming chat (`aria-live="polite"`).
- Sentiment emoji buttons announce as "Struggle / Neutral / Flow", not as the raw emoji.
- Honour `prefers-reduced-motion`, `prefers-color-scheme`, `prefers-contrast`.

---

## 6. Motion & sound

- Motion is purposeful, never decorative. Total motion budget per screen ≤ 800 ms aggregate.
- Sound is **off by default**. When on (settings), sounds are gentle, low-volume, non-startling.
- Haptic feedback on swipe success only (mobile).

---

## 7. Notifications

| Trigger | Surface | Style |
|---------|---------|-------|
| Daily action available | Push + in-app | Friendly nudge ("Ready when you are") |
| Tribemate joined | In-app toast | "Yusuf joined the tribe 👋" |
| Empathy recovery offered | In-app banner | Calm cyan, never push |
| Goal achieved | Push + in-app celebratory modal | Earned, never patronising |
| Visio scheduled | Push 10 min before | Plain |

Push permission is **never** requested on first load — only after the first successful action completion.

---

## 8. Copy guidelines

- 8th-grade reading level.
- Plain language; jargon allowed only inside a tribe's chosen subject domain.
- Empathic, never cheerleading.
- Never use exclamation points more than 1 per screen.
- Localised; every string in `i18n` files.

---

## 9. Component inventory (selected)

| Component | Notes |
|-----------|-------|
| `<AlignmentWidget />` | Real-time consensus dashboard |
| `<ActionCard />` | Swipe deck card |
| `<EmpathyBanner />` | Calm slide-in |
| `<TokenBalancePill />` | Sticky in top bar |
| `<KGMiniMap />` | Cytoscape; lazy-loaded |
| `<FSRSGradeButtons />` | 4-button row with predicted intervals |
| `<VoiceInput />` | Web Speech wrapper |
| `<StateBadge />` | S1/S2/S3/S4 marker for the tribe |
| `<RecoveryActionCard />` | Visual cyan variant |
| `<CompanionTag />` | "AI" badge on every Companion message |

Full design system in [13_DESIGN_SYSTEM.md](13_DESIGN_SYSTEM.md).

---

## 10. Performance budget

| Metric | Target |
|--------|-------:|
| FCP (3G) | < 1.5 s |
| LCP (3G) | < 2.5 s |
| INP | < 200 ms |
| Bundle gz (entry) | < 120 kB |
| Lazy chunks | < 40 kB each |

Achieved via code-splitting per route, tree-shakable design system, dynamic import of Cytoscape/WebRTC.

---

See [13_DESIGN_SYSTEM.md](13_DESIGN_SYSTEM.md) for tokens and components, [14_ACCESSIBILITY_AND_I18N.md](14_ACCESSIBILITY_AND_I18N.md) for the a11y/i18n contract.
