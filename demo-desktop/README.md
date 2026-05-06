# AGORA — Desktop Stage Demo

Full-screen, hackathon-stage click-through. **Built for a 1080p+ projector**, not for phones.

## Run

```bash
cd /home/victor/Desktop/Learning/New_platform
python3 -m http.server 8080
# open http://localhost:8080/demo-desktop/
# press F for fullscreen
```

## Controls

| Key | Action |
|---|---|
| `Space` / `→` | Next scene |
| `←` | Previous scene |
| `F` | Toggle fullscreen |
| `R` | Restart from scene 1 |

## The story (≈2 min)

**Persona — Maya Chen**, junior SWE, blocked on system design, 45 min/day, hates fluff.

| # | Scene | What to point at on stage |
|---|---|---|
| 1 | **Cold open** | The five agents orbiting the learner — that's the whole product in one image |
| 2 | **Goal capture** | Maya types fuzzy → Pedagogy paraphrases concrete; right-rail shows live agent reasoning |
| 3 | **The Council** | Verifier rejects "feel confident" → criteria converge to objective; progress bar hits 100% |
| 4 | **Knowledge Graph** | Watch nodes & edges *animate in*; Curator explains why 14 results were dropped |
| 5 | **Today** | Three cards, energy-aware. Click **Stuck** to trigger empathy → |
| 6 | **Empathy loop** | Whole UI palette shifts to cyan, plan auto-shrinks 25→10 min, guardrails listed |
| 7 | **Verifiable Credential** | OB 3.0 / W3C VC card with signature + QR. "Maya owns it." Final beat. |

## What this demo proves

1. **Multi-agent orchestration is visible** — agent bar in header lights up scene-by-scene.
2. **The Empathy Loop is real UI**, not a slide — the whole color system and pacing changes.
3. **Verifiable outcomes** — every scene closes with an artefact (criteria, graph node, badge).
4. **Spec-first** — every panel cites the SDD doc that drives it.

## Tech

Single HTML file. Tailwind CDN + Inter web font. Vanilla JS. No build, no backend, ~46 KB.
Honors `prefers-reduced-motion`.
