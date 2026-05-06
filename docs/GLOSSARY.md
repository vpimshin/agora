# Glossary

| Term | Meaning |
|------|---------|
| **AGORA** | Agentic Goal-Oriented Reflective Adaptive academy. The platform's codename and philosophy — a marketplace for ideas. |
| **Tribe** | A 1–4 person learning unit that shares a goal, criteria, resources, and a token economy. The privacy boundary. |
| **Companion** | The synthetic AI tribemate used in solo mode. Always rendered with an "AI" tag. |
| **Coach** | The Cognitive Coach agent. Paraphrases, pauses, asks one Socratic question, never reveals an answer. |
| **Verifier** | The Criteria Verifier agent. Enforces binary, observable, measurable criteria. |
| **Curator** | The GraphRAG retrieval + sourcing agent. Returns chunks with citations. |
| **Pedagogy generator** | The agent that produces daily action cards, scaffolded by FSRS + BKT + bandit. |
| **Empathy agent** | The agent that detects struggle and offers a smaller recovery action; rewards more, not less. |
| **FSM** | Finite State Machine. The S0–S5 progression governing every tribe. |
| **S0–S5** | Onboarding → Goal → Criteria → Resources → Actions (with S4_Empathy) → Achieved/Failed. |
| **Alignment widget** | Real-time multi-user convergence dashboard required to leave S1. |
| **Token** | A non-monetary recognition unit (⭐). Append-only ledger. 10 standard, 15 recovery. |
| **Recovery token** | Awarded for completing a recovery (smaller) action after struggle. Higher value than standard, deliberately. |
| **State transition** | An audited move between FSM states; row in `state_transitions`. |
| **Goal** | A learner's outcome, owned by a tribe. |
| **Criterion** | A binary, observable, measurable test of goal achievement. |
| **Resource** | An ingested learning material (file, link). Chunks are embedded and graphed. |
| **Action card** | A daily learning task surfaced via the swipe deck. |
| **Sentiment** | Self-reported (Struggle / Neutral / Flow). Drives empathy + bandit. |
| **GraphRAG** | Retrieval combining vector similarity, lexical search, and graph expansion. |
| **KG / Knowledge graph** | Per-tribe entity-relationship graph stored in Apache AGE. |
| **FSRS / FSRS-6** | Free Spaced Repetition Scheduler v6. |
| **BKT** | Bayesian Knowledge Tracing. Per-skill mastery estimate. |
| **Bandit** | Multi-armed bandit (Thompson sampling) selecting next action variant. |
| **VC / Open Badges 3.0** | Verifiable Credential issued on goal completion. |
| **Mock-mode** | Toggle that replaces external calls (LLM, STT/TTS) with deterministic mocks. Required for the demo. |
| **Empathy state** | A sub-state of S4 entered on struggle; UI palette shifts to calm cyan. |
| **Halo** | Status ring around an avatar (grey/green/pulsing) in alignment widget. |
| **Mentor** | Read-only role with aggregate access; never sees raw chat. |
| **DPA** | Data Processing Agreement, signed with each vendor. |
| **DPIA** | Data Protection Impact Assessment, completed for high-risk processes. |
| **ROPA** | Records of Processing Activities, GDPR Art. 30. |
| **PWA** | Progressive Web App. Our mobile delivery vehicle. |
| **PII** | Personally Identifiable Information; redacted before any LLM call. |
| **RLS** | Postgres Row-Level Security. The bedrock of tribe isolation. |
| **RPC (SECURITY DEFINER)** | Privileged Postgres function; the only path to write to append-only ledgers. |
| **Eval / DeepEval / Promptfoo** | Tools for automated LLM behaviour testing; gate every PR. |
| **Langfuse** | LLM observability platform; self-hostable. |
| **Realtime** | Supabase's pub/sub + presence service. |
| **AGE** | Apache AGE — graph extension on Postgres used for the KG. |
| **pgvector** | Postgres extension for vector similarity. |
| **EAA** | EU European Accessibility Act, in force June 2025. |
| **WCAG 2.2 AA** | Web Content Accessibility Guidelines, our conformance target. |
| **Tier 0 / 1 / 2** | Roadmap priority bands; cut from the top under pressure. |

---

See [01_VISION_AND_PRD.md](01_VISION_AND_PRD.md) for tiered requirements, [04_SYSTEM_ARCHITECTURE.md](04_SYSTEM_ARCHITECTURE.md) for the system context.
