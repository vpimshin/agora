# SDD at a glance

**21 documents · 5 ADRs · 6 agent prompts · 1 glossary** — read top-to-bottom; each section answers one question.

| # | Doc | Answers |
|---|---|---|
| 00 | `OVERVIEW` | What is AGORA in one page? |
| 01 | `VISION_AND_PRD` | Why does it exist, what does it ship, for whom? |
| 02 | `PERSONAS_AND_JOURNEYS` | Who is Maya — and what does her week look like? |
| 03 | `STATE_MACHINE` | The S0→S5 lifecycle every learner moves through. |
| 04 | `SYSTEM_ARCHITECTURE` | Services, boundaries, dataflow — the wiring diagram. |
| 05 | `DATA_MODEL` | Entities, relationships, invariants. |
| 06 | `API_CONTRACTS` | Public surface — typed request/response per endpoint. |
| 07 | `AGENT_ARCHITECTURE` | The five agents: roles, vetoes, arbitration. |
| 08 | `RAG_AND_KNOWLEDGE_GRAPH` | How facts are stored, retrieved, cited. |
| 09 | `PEDAGOGICAL_ENGINE` | Spaced repetition, ladders, mastery thresholds. |
| 10 | `GAMIFICATION_AND_TOKENS` | Token economy — anti-Goodhart guardrails. |
| 11 | `EMPATHY_LOOP` | Detect struggle → shrink task → protect streak. |
| 12 | `UI_UX_SPEC` | Screen-by-screen behavior + interaction states. |
| 13 | `DESIGN_SYSTEM` | Tokens, type, palette (incl. empathy palette shift). |
| 14 | `ACCESSIBILITY_AND_I18N` | WCAG 2.2 AA, RTL, locale-aware. |
| 15 | `SECURITY_AND_PRIVACY` | Private-by-default, threat model, key mgmt. |
| 16 | `OBSERVABILITY_AND_EVALS` | Logs, traces, agent-output evals. |
| 17 | `TESTING_STRATEGY` | Unit / contract / red-team / learner evals. |
| 18 | `DEPLOYMENT` | Environments, rollouts, kill-switches. |
| 19 | `DEMO_SCRIPT` | The 7-min stage walkthrough. |
| 20 | `BUILD_ROADMAP` | Order of work, milestones, what we ship first. |

**`adr/`** — five irreversible decisions (multi-agent vs. single, OB 3.0 over proprietary creds, palette-as-affect, arbitration policy, hard scope cuts). **`prompts/`** — the system prompts for Pedagogy, Verifier, Curator, Empathy, Companion + a shared safety preamble. **`GLOSSARY.md`** — one source of truth for every term.

> Read in order. After 30 minutes you can ship.
