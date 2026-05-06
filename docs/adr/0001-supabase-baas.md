# ADR 0001 — Use Supabase as primary BaaS

- **Status:** Accepted
- **Date:** Phase 0
- **Decision-makers:** Platform team

## Context

We need: Postgres with extensions (pgvector, Apache AGE, pg_trgm), authentication, realtime pub/sub + presence, file storage, edge functions, and row-level security as the backbone of tribe isolation. Hackathon timeline plus open-source ethic.

## Options considered

1. **Supabase** — managed Postgres + Auth + Realtime + Edge + Storage; self-hostable.
2. **Firebase** — fast, but locked-in, NoSQL, limited row-level model, weak for relational + vector + graph.
3. **AWS (RDS + Cognito + AppSync + Lambda + S3)** — flexible, slow to assemble, no cohesive realtime story.
4. **Self-host Postgres + Hasura + custom auth** — maximal control, very high build cost.

## Decision

Adopt **Supabase**. It is the only option that ships every primitive we need (RLS, vector, realtime presence, storage, edge functions, auth) in one well-maintained, self-hostable, open-source package, with a free tier that fits the hackathon and a clear paid path for production.

## Consequences

- ✅ Fastest path to a walking skeleton.
- ✅ RLS-native — privacy is enforced at the database, not the app.
- ✅ Realtime + presence built-in — the alignment widget is cheap.
- ✅ Self-hostable — escape hatch from vendor lock-in.
- ⚠️ Edge functions are Deno; we standardise on Deno-compatible TypeScript.
- ⚠️ AGE is community-installed in the Supabase image — we maintain a small bootstrap migration.

## Revisit when

- Postgres becomes a bottleneck for vector at our scale (> 10 M vectors).
- Realtime cost or fan-out limits start hurting tribe sizes (n/a at our 1–4 cap).
