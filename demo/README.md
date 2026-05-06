# AGORA Demo — Hard-coded click-through

Single-file, zero-build, frontend-only. Open in a browser, click through the story in ~90 seconds.

## Run

```bash
# any of these works
open demo/index.html
# or
python3 -m http.server 8080  # then visit http://localhost:8080/demo/
```

No backend, no install, no API keys. Tailwind via CDN; everything else is hand-rolled vanilla JS.

## Flow (5 scenes, ~90 s)

1. **Splash** → *Try AGORA*.
2. **Onboarding** → pick **Solo** (you'll get an AI Companion).
3. **S1 Coach** — three turns. Coach paraphrases, breathes (visible pause bar), asks one question, never gives the answer. Companion follows. Alignment widget converges → **S1 → S2**.
4. **S2 Criteria** — first draft is subjective ("I will be *good* at Python"). Verifier flags it and proposes a binary rewrite. Accept → **S2 → S3**.
5. **S3 Resources** — three free open-licence resources; *Add to library* renders the per-tribe knowledge graph. → **S3 → S4**.
6. **S4 Today** — swipe deck.
   - Action #1: **Accept** → **+10⭐** (standard). Brief gold "flow" palette tease.
   - Action #2: click **😔 Struggle** → palette shifts **calm cyan**, Empathy banner enters, recovery card replaces. Accept → **+15⭐** (recovery > standard, deliberately).
7. **S5 Achieved** — Verifiable Credential (Open Badge 3.0) ready to download, totals shown.

## Features showcased

- 5-state FSM with audited transitions (state pill in the top bar).
- Coach prompt structure: paraphrase → breathing pause → single open question.
- Real-time presence + alignment widget (mocked).
- AI Companion with non-removable **AI** tag.
- Verifier turning subjective into binary criteria.
- GraphRAG knowledge graph rendered (mock SVG).
- Token economy: append-only awards, **recovery > standard** anti-shame economics.
- Empathy loop: struggle signal → palette shift → recovery action → reward.
- Verifiable Credentials at goal completion.
- WCAG-aware: keyboard reachable, `prefers-reduced-motion` honoured, 16 px+ body.

## Reset

A **Reset** button lives in the guidance ribbon. The bottom **Replay** button on the achieved screen does the same.
