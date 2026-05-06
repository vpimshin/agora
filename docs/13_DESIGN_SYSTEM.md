# 13 — Design System

> Tokens drive everything. No raw colour, no raw spacing in components. Ever.

## 1. Foundations

### 1.1 Typography

- **Sans serif:** [Inter Variable](https://rsms.me/inter/) — primary UI.
- **Mono:** JetBrains Mono — code blocks.
- **Arabic:** IBM Plex Sans Arabic — RTL screens.
- Variable axes used: weight (400, 500, 600, 700), optical size.

| Role | Size (px) | Line-height | Weight |
|------|----------:|------------:|-------:|
| Display | 36 / 48 | 1.1 | 700 |
| H1 | 28 | 1.2 | 700 |
| H2 | 22 | 1.25 | 600 |
| H3 | 18 | 1.3 | 600 |
| Body | 16 | 1.5 | 400 |
| Body small | 14 | 1.5 | 400 |
| Caption | 12 | 1.4 | 500 |
| Code | 14 | 1.5 | 400 mono |

Min body size on mobile **never below 16 px** (prevents iOS auto-zoom on input).

### 1.2 Colour tokens (semantic)

Light theme (HSL form, Tailwind v4 `@theme`):

```css
@theme {
  /* Brand */
  --color-brand-50:  255 252 245;
  --color-brand-500: 256 96% 60%;   /* primary purple */
  --color-brand-700: 256 70% 40%;

  /* Surfaces */
  --color-bg:        220 30% 99%;
  --color-surface:   220 25% 96% / 0.6;   /* glass */
  --color-fg:        220 25% 12%;
  --color-fg-muted:  220 12% 35%;
  --color-border:    220 14% 88%;

  /* Semantics */
  --color-success:   148 60% 42%;
  --color-warning:    35 85% 50%;
  --color-danger:    354 70% 50%;
  --color-info:      210 80% 50%;

  /* Empathy palette (calm cyan) */
  --color-empathy-bg: 187 70% 96%;
  --color-empathy-fg: 195 45% 25%;

  /* Flow palette (warm gold) — for sentiment=flow */
  --color-flow-bg:    44 90% 96%;
  --color-flow-fg:    36 80% 30%;
}
```

Dark theme defines the same tokens with inverted lightness; user can override via system or settings.

### 1.3 Contrast

All text/background pairs are checked at build time by a Vite plugin. Minimum **4.5:1 for body**, **3:1 for large text and UI components**. Empathy and Flow palettes are explicitly tuned to pass at AA.

### 1.4 Spacing

Base 4 px scale: `0, 1, 2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 24` × 4 px. Tailwind defaults retained.

### 1.5 Radii

`xs=4 sm=8 md=12 lg=16 xl=20 2xl=28 full=9999`.

### 1.6 Shadows (glassmorphism)

```css
--shadow-glass-sm: 0 1px 2px hsl(220 25% 12% / .06), 0 4px 12px hsl(220 25% 12% / .08);
--shadow-glass-md: 0 4px 16px hsl(220 25% 12% / .10), 0 16px 32px hsl(220 25% 12% / .08);
--shadow-glow:     0 0 0 4px hsl(var(--color-brand-500) / .25);
```

Glassmorphic surfaces use:
```css
background: hsl(var(--color-surface));
backdrop-filter: blur(20px) saturate(140%);
border: 1px solid hsl(var(--color-border) / 0.6);
```

### 1.7 Motion

- Duration scale: `instant 0ms, fast 120ms, default 220ms, slow 360ms, leisurely 600ms`.
- Easing: cubic-bezier(0.2, 0.8, 0.2, 1) for default; `ease-in` for exits.
- All durations mapped to `0` if `prefers-reduced-motion`.

---

## 2. Components (selected specs)

### 2.1 Button

| Prop | Variants |
|------|----------|
| `variant` | primary, secondary, ghost, danger, empathy |
| `size` | sm, md, lg, icon |
| `loading` | boolean (spinner replaces label, width preserved) |
| `iconLeft / iconRight` | ReactNode |

Primary uses `--color-brand-500`. Empathy uses `--color-empathy-fg/bg`. Always 48 px tall on mobile (lg).

### 2.2 ActionCard

Glassmorphic surface, 16 px radius, shadow-glass-md, drag-to-swipe gesture (Framer Motion). Variants: `standard`, `recovery` (cyan tint), `flow` (gold tint when last completion was flow).

Accessibility: keyboard alternative — focus card, then `Enter` to accept, `Esc` to defer, `?` to struggle.

### 2.3 AlignmentWidget

4-up grid of avatars, each with a halo:
- Grey halo: vote pending.
- Green halo: aligned.
- Pulsing halo: actively typing.

Supports 1–4 members. When unanimous, the widget plays a 600 ms convergence animation before transitioning to S2.

### 2.4 EmpathyBanner

Slide-in from bottom, 64 px tall mobile / 56 px desktop. Calm cyan palette. Exposes a single CTA "Try simpler" + dismiss `×`. `role="status"`, `aria-live="polite"`.

### 2.5 SentimentPicker

3-emoji segmented control with text labels for screen readers ("Struggle", "Neutral", "Flow"). Buttons grow on press. Prevents accidental tap with 200 ms hold-to-confirm on mobile.

### 2.6 StreamingMessage

Renders incoming SSE tokens. Pause is a literal `<span class="breathing-dot">·</span>` between paraphrase and question. Honours reduced-motion (no animation, hard cut).

### 2.7 KGMiniMap

Cytoscape canvas, 100 % width × 220 px tall by default; expands on tap to full-screen sheet. Nodes scaled by salience; edges weighted. Touch-friendly: pinch zoom, two-finger pan, single-tap to select, long-press for details.

### 2.8 TokenBalancePill

Top-bar pill: `⭐ 230`. Animates increment on award (count-up over 600 ms). Tap → opens ledger modal.

---

## 3. Iconography

[Lucide Icons](https://lucide.dev) — single source. No mixed icon libraries. Stroke 1.75. 24 px default, 20 px in dense rows.

---

## 4. Imagery & illustrations

- Brand illustrations sourced from [unDraw](https://undraw.co) recoloured to brand palette.
- Avatars: DiceBear or user-uploaded; never pulled from third-party trackers.
- All images use `loading="lazy"` and explicit `width/height` to prevent CLS.

---

## 5. Tokens to code

Shipped to runtime via Tailwind v4's `@theme` directive **and** as JSON in `/design-tokens/tokens.json` so non-web surfaces (badge images, social cards) can consume them.

Story authoring lives in `Storybook` (lazy-loaded; not shipped to prod).

---

## 6. Localisation hooks

- All component copy is `t('...')`.
- Numbers use `Intl.NumberFormat`, dates use `Intl.DateTimeFormat`.
- RTL support driven by `dir="rtl"` on `<html>`. All component CSS uses logical properties (`margin-inline-start`, etc.).

---

## 7. Theming

| Theme | Source |
|-------|--------|
| Light (default) | Defined above |
| Dark | Inverted lightness with palette tweaks |
| High-contrast | Triggered by `prefers-contrast: more`; thicker borders, raised contrast |
| Empathy | Per-screen palette shift during S4_Empathy |
| Flow | Per-screen palette shift after consecutive `flow` sentiments |

Theme transitions animate over 220 ms.

---

## 8. Lint & guardrails

- A custom Tailwind plugin forbids raw colour classes (`bg-white`, `text-black`) and raw HSL values.
- A11y linting via `eslint-plugin-jsx-a11y`.
- Storybook a11y add-on enforces axe checks on every story.
- Visual regression via Playwright + Chromatic-equivalent OSS (`pixelmatch`).

---

## 9. Brand voice (for copy that lands in components)

Adjectives the brand answers to: warm, deliberate, plain-spoken, respectful, never preachy.
Adjectives the brand rejects: cute, sales-y, gamified-bro, condescending.

---

See [12_UI_UX_SPEC.md](12_UI_UX_SPEC.md) for screen-level usage and [14_ACCESSIBILITY_AND_I18N.md](14_ACCESSIBILITY_AND_I18N.md) for the a11y contract.
