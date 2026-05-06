# ADR 0003 — FSRS-6 for spaced repetition

- **Status:** Accepted
- **Date:** Phase 3

## Context

We need an evidence-based spaced repetition scheduler that handles imperfect recall, varies per item, and adapts to the learner.

## Options

1. **FSRS-6** (Free Spaced Repetition Scheduler v6) — open, parameter-fitting, peer-reviewed-ish, Anki-adopted.
2. **SM-2** — simple, but less accurate; the Anki classic.
3. **Anki SM-15+** — proprietary lineage; not open.
4. **Leitner boxes** — too crude.

## Decision

Adopt **FSRS-6**. It outperforms SM-2 in published evaluations, is open-source with multi-language ports, and exposes per-user parameter fitting once enough data is collected.

## Consequences

- ✅ Stronger retention with fewer reviews — respects learner time.
- ✅ Per-user parameter optimisation in tier-1 (offline job).
- ⚠️ Slightly more state to persist (`stability`, `difficulty`, `last_review_at`, etc.) — captured in [05](../05_DATA_MODEL.md).
- ⚠️ Requires occasional re-fitting; we automate via scheduled job.

## Revisit when

- A successor (FSRS-7 / DASH / etc.) achieves materially better results in our domain.
