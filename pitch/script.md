# AGORA · 5-minute pitch script

> Companion deck: `pitch/index.html`. Live demo: `demo-desktop/index.html`.
> Run both with `python3 -m http.server 8080` from `New_platform/`.
> Recommended path on stage: open the pitch deck → press **F** for fullscreen → use **Space** / **→** to advance → final slide auto-launches the demo (or press **D** any time).

---

## Timing budget (5:00)

| Slide | Section | Target | Cumulative |
|---|---|---|---|
| 1 | Title — hook | 0:20 | 0:20 |
| 2 | Problem | 0:50 | 1:10 |
| 3 | Solution — five agents | 0:55 | 2:05 |
| 4 | How it works — the loop | 0:55 | 3:00 |
| 5 | Spec-Driven approach | 0:45 | 3:45 |
| 6 | Lessons learned | 0:50 | 4:35 |
| 7 | Hand-off to demo | 0:25 | 5:00 |

Then the **2-minute demo** runs separately. Plan total stage time ≈ 7 min (5 pitch + 2 demo) — adjust if your hackathon slot is tight.

---

## SLIDE 1 — Title (0:20)

> *Walk on, point at the orbit visual.*

> "Hi. We're team AGORA.
> What you see behind me is our entire product in one image: a learner, surrounded by **five AI agents**, each with one job — and one veto.
> In the next five minutes I'll show you why we built it, how it works, and what nearly broke us.
> Then I'll let it speak for itself."

*Press Space.*

---

## SLIDE 2 — Problem (0:50)

> *Let the red palette land for a beat.*

> "Online learning for adults is broken. We all know it. The numbers say it: **3% MOOC completion**, **70% forgotten in a day**, certificates nobody trusts.
>
> But the deeper problem isn't motivation. It's that the product optimizes for the wrong thing — **completion, not capability**.
>
> Meet Maya. *(point at the panel)* Junior engineer, 26, 45 minutes a day, stuck on system design. She's taken three courses. She can recite CAP. She still freezes at the whiteboard.
> Maya is not lazy. The product is wrong."

*Press Space.*

---

## SLIDE 3 — Solution (0:55)

> *Let the five-card row breathe — point at each as you name it.*

> "Our bet: a single chatbot can't fix this. A coordinated **council of agents** might.
>
> – **Pedagogy** designs the path — and refuses to teach without a destination.
> – **Verifier** removes ambiguity — no 'feel confident', only 'observable, falsifiable, fair'.
> – **Curator** cuts noise — drops fourteen sources for every one it keeps, and tells you why.
> – **Empathy** notices struggle — shrinks the task, never the person.
> – **Companion** shows up daily — energy-aware, 45 minutes, no marathons.
>
> Five roles, one shared learning contract. Each role auditable. Each output verifiable."

*Press Space.*

---

## SLIDE 4 — How it works (0:55)

> *Walk through the four boxes left to right.*

> "Under the hood it's a four-step loop, and every step produces a file you can audit, replay, or revoke.
>
> 1. **Goal capture** — fuzzy goal in, signed `rubric.json` out.
> 2. **Living map** — a personalized knowledge graph; each concept tied to a real artefact.
> 3. **Daily practice** — Companion plans, Empathy adapts in real time.
> 4. **Verifiable proof** — Open Badges 3.0 plus W3C Verifiable Credentials. Signed. Portable. Revocable. **The learner owns the credential — not the platform.**
>
> Three things never change: data is private by default, every agent step is logged, and Verifier rejects any metric that could be gamed."

*Press Space.*

---

## SLIDE 5 — Spec-Driven approach (0:45)

> *This is the credibility slide — slow down here.*

> "Most hackathon projects are vibes plus duct tape. We took a different bet.
>
> We wrote the **spec before we wrote a line of code**: 21 design documents, 5 architectural decision records, 6 agent prompts, one glossary.
>
> The thesis: **if AI can implement from a spec, then the spec is the product.** A new contributor — human or AI — onboards in thirty minutes by reading docs in order.
>
> What you're about to see in the demo is hard-coded — but every screen maps to a doc in this folder. Nothing is improvised."

*Press Space.*

---

## SLIDE 6 — Lessons learned (0:50)

> *Use the two-column structure: left = won, right = scars.*

> "Quick honesty pass.
>
> **What flowed:** Once we named the agent roles, the architecture designed itself. Visualizing emotional state as a UI **palette shift** was a happy accident — instantly readable on stage. Open standards saved us weeks: OB 3.0 and W3C VC just *exist*.
>
> **What burned us:** Agents disagree — Pedagogy wanted to push, Empathy wanted to slow down. We had to write an explicit arbitration ADR. Turning 'feel confident' into a falsifiable rubric is **the actual product, not a feature**. And every metric we added invited a way to game it — we rewrote the token economy three times.
>
> The hardest discipline? **Not coding.** The spec is so rich the temptation to ship a backend was constant. We resisted. What you'll see is a click-through — on purpose."

*Press Space.*

---

## SLIDE 7 — Hand-off (0:25)

> *Tone shift: enough talking, time to show.*

> "Enough slides.
>
> The next two minutes are Maya's story — captured live. Watch a fuzzy goal become a verifiable credential, through the eyes of all five agents. Notice the palette when she gets stuck. That's the empathy loop running.
>
> Press play."

*Press Space (or D) — the demo launches.*

---

## DEMO HAND-OFF (0:00 — beginning of the 2-min demo)

When the demo opens, **press F for fullscreen**, then **Space** to advance scenes. Narration is optional during the demo — the screen carries it. If you do narrate, two beats matter:

- **Scene 5 → 6:** *"Watch what happens when she clicks 'I'm stuck'."* (The whole UI shifts cyan.)
- **Scene 7:** *"That's not a course completion. That's a signed, portable claim. She owns it."*

---

## CLOSE (after demo, ~15 sec)

> "AGORA. Five agents. One contract. Verifiable outcomes — built spec-first because the spec **is** the product. Spec, prompts, ADRs and demo are all on GitHub at `dman1313/TeamConnect/v2`. Thank you."

---

## Backup Q&A pocket-cards

- **"How is this different from an LLM tutor?"**
  An LLM tutor is one role pretending to be five. We split the roles so they can disagree — and we log the disagreement. Verifier can veto Pedagogy. Empathy can pause Companion. That's the safety mechanism a single agent fundamentally can't have.

- **"What about hallucinations?"**
  Two answers. (1) Curator ties every claim to a cited artefact in the knowledge graph; nothing reaches the learner without provenance. (2) The credential at the end is evidence-bound — peers grade the artefact, not the AI's opinion. Goodhart-resistant by design.

- **"Why hard-coded for the hackathon?"**
  Because shipping a half-broken backend would have undermined the central bet — that the **spec** is what we're betting on. The demo proves the design works at the screen level. The next sprint is code-gen from the same spec.

- **"What's next?"**
  30-learner pilot. Four weeks. One verifiable cohort. Open the agent prompts as a public reference repo.

---

## Stage tech checklist

- [ ] Demo server running: `python3 -m http.server 8080` from `New_platform/`
- [ ] Open `http://localhost:8080/pitch/` first → press **F** for fullscreen
- [ ] Verify network: Tailwind CDN + Inter font load (else palette stays unstyled). If offline: pre-cache before stage.
- [ ] Test the auto-launch from slide 7 → demo
- [ ] Bring a clicker that maps to **Space** / **→**
- [ ] Have `Cmd/Ctrl+R` ready to reset the demo if you click past the empathy step
