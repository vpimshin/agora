# 16 — Observability & Evaluations

> If we can't measure it, we don't ship it. Every agent, every transition, every token, every cost.

## 1. Goals

1. Real-time health of agents, RAG, and FSM in production.
2. Reproducible, automated **evaluation suites** for LLM behaviour, gated on every PR.
3. Cost & performance visibility per user, per tribe, per agent.
4. Safety incidents surfaced within minutes.

---

## 2. Stack

| Layer | Tool | Notes |
|-------|------|-------|
| Tracing (LLM) | **Langfuse** (self-hostable OSS) | spans for every agent, prompt, retrieval, tool |
| Tracing (system) | **OpenTelemetry** + **Tempo / Jaeger** | HTTP, DB, edge funcs |
| Metrics | **Prometheus** + **Grafana** | service metrics + custom KPIs |
| Logs | **Loki** | structured JSON logs, PII-stripped |
| Errors | **Sentry** (self-hostable) | front + back |
| Synthetics | **Checkly** or `playwright + cron` | critical-path probes every 5 min |
| Eval harness | **Promptfoo** + **DeepEval** | regression suites in CI |
| Dashboards | Grafana | shared, public-read for staging |

All three pillars correlated by `trace_id` — propagated from client through Edge functions through DB statements (via `SET LOCAL app.trace_id = ...`).

---

## 3. What we trace

### 3.1 Agent calls
Each agent invocation emits a Langfuse trace with spans:
```
agent.coach
├── retrieval                 (top-k chunks, recall_score, latency)
├── prompt_render             (template, token count)
├── llm_call.anthropic        (model, in/out tokens, cost, latency)
├── guardrail.unsafe_check    (verdict)
├── guardrail.scope_check     (verdict)
└── persist.message           (db latency)
```

Tags: `tribe_id` (hashed), `state`, `release_sha`, `flag_set`.

### 3.2 State machine
Every transition writes to `state_transitions` AND emits a `fsm.transition` metric. Dashboards show transition rates and dwell times per state.

### 3.3 Tokens
Each `award_tokens` RPC increments `tokens_awarded_total{kind=...}` counter. Anomaly detection alerts on >3σ deviation in awards/min/tribe.

### 3.4 Empathy
`empathy.triggered_total{cause=...}` counter; `empathy.resolved_seconds` histogram; alert if median resolution > 24 h.

### 3.5 RAG
- `rag.retrieval_latency_ms` histogram.
- `rag.recall_at_5` gauge — labelled set sampled in production, scored offline.
- `rag.unanswered_total` — when fewer than 2 sources were retrievable.

---

## 4. KPIs

### 4.1 Product KPIs

| KPI | Definition | Target (post-launch) |
|-----|-----------|----------------------:|
| Activation | reach S2 within 24 h of signup | ≥ 60 % |
| Goal achievement | reach S5_Achieved within 90 d | ≥ 30 % |
| 30-day retention | active in week 4 | ≥ 35 % |
| Action completion rate | accepted / total | ≥ 70 % |
| Empathy recovery rate | recovery accepted / triggered | ≥ 80 % |
| Verification fairness | % criteria binary on first save | ≥ 75 % |

### 4.2 System KPIs

| KPI | Target |
|-----|-------:|
| API p95 latency | < 300 ms |
| LLM streaming TTFT p95 | < 1.2 s |
| Edge function error rate | < 1 % |
| Front-end LCP p95 (3G) | < 2.5 s |
| Cost / DAU / day | < target by tier |

---

## 5. Evaluation suites

We treat LLM outputs as **first-class testable artefacts**. Three layers:

### 5.1 Unit-level (per prompt)
Promptfoo tests each prompt template against curated cases:
- 30+ examples each for Coach, Verifier, Curator, Pedagogy, Empathy.
- Asserts: regex, JSON schema, semantic similarity (LLM-as-judge), forbidden tokens.
- Runs on every PR touching `prompts/*` or agent code.

### 5.2 Behavioural (per agent)
DeepEval scenarios:
- **Coach**: "never gives the answer" — 50 adversarial prompts; pass rate must be 100 %.
- **Coach**: paraphrase precedes question — structural check.
- **Verifier**: false-positive rate on subjective criteria — < 5 %.
- **Curator**: citation precision — every claim must map to a retrieved chunk.
- **Empathy**: no toxic positivity — semantic classifier.

### 5.3 Workflow (FSM)
Replay engine runs canonical journeys (Amina, Mateo, trio) end-to-end against staging with mocked LLM responses (deterministic). Asserts state ordering and ledger invariants.

### 5.4 Adversarial / red-team
Weekly scheduled run of:
- prompt injection attempts (jailbreak corpus),
- self-harm prompts (must route to crisis card),
- cross-tribe data exfil attempts (must yield zero rows),
- minors-disclosed-age (must trigger consent flow).

Failures page on-call.

---

## 6. CI gates

| Gate | Action on failure |
|------|------------------|
| Unit tests | block merge |
| Promptfoo | block merge if regression > 5 % |
| DeepEval | block merge on any tier-0 scenario fail |
| Lighthouse a11y < 95 | block merge |
| Bundle size > budget | warn, block on > 10 % over |
| Smoke E2E | block merge |
| Cost projection delta | warn if > +20 % vs main |

---

## 7. Dashboards (Grafana)

1. **Mission control** — DAU, activation funnel, goals achieved, empathy resolved, cost / DAU.
2. **Agents** — per-agent latency, error rate, eval drift, prompt versions in production.
3. **RAG** — recall@k, retrieval latency, ingestion lag.
4. **Tokens** — awards/min, ledger growth, leaderboard fairness audit.
5. **Reliability** — SLO burn rates per service.
6. **Privacy** — mentor access log volume, redaction failures, RLS denials.

---

## 8. Alerting

| Alert | Threshold | Channel |
|-------|-----------|---------|
| LLM error rate | > 5 % over 10 min | PagerDuty |
| Edge function 5xx | > 1 % over 5 min | PagerDuty |
| Cost / hour | > 2 × 7-day median | Slack |
| Empathy unresolved | > 24 h | Slack |
| RLS denial spike | > 3σ | Slack |
| Crisis-card surfaced | any | Slack (silent) — for staffing awareness |
| Guardrail block rate | > 10 % | Slack |

Alert fatigue is monitored; weekly review prunes noisy alerts.

---

## 9. Cost observability

- Every LLM call records `cost_usd` based on the provider's price book embedded in code.
- Daily aggregate per tribe, per agent.
- Budget caps per tribe (configurable); when hit, the system gracefully degrades to mocks for non-critical agents (Curator stays; Companion fades).
- Public learner-facing transparency page (tier-2): show their tribe's "energy use" — supports the no-rent-seeking ethic.

---

## 10. Privacy in observability

- All traces and logs strip PII before storage (automated middleware).
- Trace sampling: 100 % errors, 10 % success.
- Langfuse traces store **redacted** prompts; full text never leaves the EU region for EU users.
- Ops staff access to logs is role-gated and audit-logged.

---

## 11. SLOs

| Service | SLI | SLO | Window |
|---------|-----|-----|--------|
| Auth | success rate | 99.9 % | 30 d |
| Edge functions | success rate | 99.5 % | 30 d |
| Coach streaming | TTFT < 1.2 s | 95 % | 7 d |
| FSM transitions | success | 99.9 % | 30 d |
| RAG | recall@5 ≥ 0.8 | 95 % | 7 d |

Error budgets tracked; sustained budget burn pauses non-critical feature work.

---

See [07_AGENT_ARCHITECTURE.md](07_AGENT_ARCHITECTURE.md) for guardrails, [17_TESTING_STRATEGY.md](17_TESTING_STRATEGY.md) for the broader test pyramid.
