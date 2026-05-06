# ADR 0004 — Multi-agent orchestration with LangGraph (TS)

- **Status:** Accepted
- **Date:** Phase 1

## Context

Five agents (Coach, Verifier, Curator, Pedagogy, Empathy) plus a Companion need to cooperate, share state, retry, and emit traces. We want explicit state machines per agent and an orchestrator that's deterministic enough to test.

## Options

1. **LangGraph (TypeScript)** — explicit graphs, persistent state, retries, integrates with Vercel AI SDK.
2. **Direct provider SDK + custom orchestration** — minimal but reinvents wheels.
3. **CrewAI / AutoGen** — opinionated, Python-first, less aligned with our Edge (Deno/TS) target.
4. **Hand-rolled XState orchestrator** — appealing for small graphs; we already use XState client-side.

## Decision

Adopt **LangGraph (TS)** for server-side agent orchestration; XState remains for client-side UI state. Provider abstraction via Vercel AI SDK so we can swap models per agent.

## Consequences

- ✅ Explicit graph; debuggable; trace-friendly.
- ✅ Provider-agnostic per agent — Coach can be a tier-1 model while Companion uses a small local one.
- ✅ Persistence hooks for resumable conversations.
- ⚠️ Adds a dependency; we pin and audit.
- ⚠️ Deno compatibility is acceptable but tested per release.

## Revisit when

- A simpler alternative emerges that natively supports Deno + tracing.
- Our agent count grows past ~10 and we need a different abstraction.
