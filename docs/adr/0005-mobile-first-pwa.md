# ADR 0005 — Mobile-first PWA (not native)

- **Status:** Accepted
- **Date:** Phase 0

## Context

Target learners are mobile-first, often on intermittent connectivity, across many regions and app stores. We need offline support, push notifications, microphone access, and installability — without the cost of two native apps.

## Options

1. **Progressive Web App (React 19 + Vite)** with service worker, Web Push, Web Speech, Web Share.
2. **React Native / Expo** — native install, app-store distribution, more device APIs.
3. **Capacitor wrapper over the PWA** — best of both, deferrable.
4. **Native Swift + Kotlin** — out of scope for hackathon and ethic.

## Decision

Ship a **PWA** as the day-one product. Plan a **Capacitor** wrapper for App Store / Play Store distribution as a tier-2 follow-up if user demand justifies it.

## Consequences

- ✅ One codebase, instant updates, no review delays, low cost.
- ✅ Works in regions where app stores are restricted.
- ✅ Service worker enables the offline "shell" + cached library reads.
- ⚠️ Push on iOS is supported (16.4+) but with caveats; we keep notifications informational and not load-bearing.
- ⚠️ Microphone & background audio quirks per platform; tested in CI mobile matrix.

## Revisit when

- A specific market requires App Store presence.
- A hardware capability we need is web-unavailable (e.g., advanced biometrics).
