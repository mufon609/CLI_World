# CLI_World — Phase 13: God Mode & On-Demand 3D Vision

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0–12. Read **`CLI_World-Design-and-Roadmap.md`** (especially **Part X — The human & anonymity** and **Part XII — Determinism & architecture**) and all of `/canon` first. This is a single-shot build spec for two capabilities at opposite ends of the glass: the **human's god powers** (perturb the world) and the **AI's on-demand 3D vision** (look at it). Ship the Phase Contract; leave the repo green and auditable for the PM.

---

## 0. What this phase is (and is not)

Two distinct additions, two distinct actors:

- **God mode** is a **human capability:** the watcher, beyond spectating or walking as an avatar, can **perturb the world** — spawn and move things, shift moods, drop money, turn the weather, trigger an event. It is the third stance in the human's place (spectator / **god** / avatar).
- **On-demand 3D vision** is an **AI capability:** the protagonist, whose default perception is **structured text**, can **explicitly request a rendered image** of its surroundings. Vision is **opt-in**, never the AI's default way to perceive or navigate.

Both are governed by the same two laws, and the phase lives or dies by them:

- **Perturbation is diegetic, and the box stays closed.** A god's action reaches the AI **only as an ordinary world event** — an NPC's mood changed, an item appeared, the weather turned. The AI must be **unable to tell a god-caused event from a natural, seeded one.** No "a god did this" ever crosses into AI-facing state. (The *observatory* sees the truth; the *AI* sees the world.)
- **Determinism survives because everything exogenous is recorded.** The deterministic sim has exactly three non-deterministic, **recorded** actors: the **model**, the **human avatar**, and now the **god channel.** A god perturbation is an **exogenous event written to the log**; a vision request is a **recorded read.** `world.replay(log)` reproduces the run **exactly**, god actions and screengrabs included. No new `Math.random`/wall-clock anywhere.

**Non-negotiables:**
- **Diegetic perturbation.** God actions manifest **inside the world's rules** as injected events the engine processes — never as a fourth-wall signal. The AI cannot detect a god. Test it.
- **Determinism is sacred.** God actions and vision calls are **log entries**; replay reproduces them; the sim stays pure. The model/human/god are the only non-deterministic inputs, and all are recorded.
- **Anonymity is sacred.** Neither a **god action** nor a **rendered image** may expose the human. The human avatar **renders identically** to puppets/agents (Phase-3 anonymity, in 3D), and a perturbation channel must not let the AI infer who the human is.
- **Text is the default; vision is opt-in.** Native structured state remains how the AI perceives and navigates; the screengrab is an **explicit tool call**, not the base loop.
- **God is human-only.** The AI has **no** perturbation capability — no spawn/alter/trigger tool in its manifest. The perturbation channel is gated to the human operator.
- **No skills.** Nothing here adds a skill/XP/level term.

---

## 1. God mode — the perturbation channel  `packages/world/src/god/` (human-only)

- Expose a **human-operator channel** through which the watcher injects **perturbations**: spawn / despawn an entity, move one, alter an NPC's mood or standing, adjust the protagonist's needs or money, change weather or time-of-day, fire a world event. (Reasonable, world-shaped perturbations — not arbitrary state-bashing that would corrupt invariants.)
- **Every perturbation is recorded as an exogenous event in the event log** (the same mechanism as the Phase-3 human avatar's actions and Phase-9 events). The engine processes it through its **ordinary rules**, so the resulting state is well-formed and **deterministically replayable.**
- Perturbations **share the Phase-9 event substrate and noise floor** — a god-spawned event is the **same kind of event** the seeded stream produces, so it is **indistinguishable** to the AI from a natural one.
- The channel is **gated to the human operator** and is **never** present in the AI's tool manifest.

## 2. Perturbation is diegetic — keeping the box closed

- A god action reaches the AI **only through the world's normal perception** (`look`/`talk`/`status`/events). There is **no attribution** — no flag, source field, tone, timing tell, or out-of-band message that says a god acted. The AI experiences a god-moved NPC exactly as it would a scripted one.
- The **honest observer layer** (Phase 10) shows the **watcher** the ground truth: which events were **god-authored**, and what changed — clearly observer-only, alongside the existing register/clue/quest panels.
- **Anonymity through perturbation:** a god may act on entities, but the channel must not become a way for the AI to **back out the human's identity** (e.g., no AI-visible artifact distinguishes "the entity a god just touched" in a way that triangulates the human). The Phase-3/9/11 anonymity guarantees hold with god mode active.

## 3. On-demand 3D vision — the screengrab tool  `packages/world/src/mcp/` + `packages/client` (AI-facing)

- Add **one explicit world-MCP tool** the AI can call to receive a **rendered image** of the scene **from its own vantage** (its position/facing in the town). The tool **returns an image** the vision-capable model can look at.
- **Structured text stays the default.** The AI perceives and navigates from native state (`look`/`status`/the map); the screengrab is an **on-demand supplement**, called when the AI judges a *visual* would help — never the substrate of its base loop.
- The image is **rendered from the authoritative Three.js scene** (Phase 3) — a faithful picture of the **deterministic world state** at that tick from that camera. It introduces **no information the text view wouldn't** and **no new randomness.**
- Optionally give vision a **small cost/latency** ([TUNABLE], Phase 15) so it stays a deliberate choice rather than a free default — but the load-bearing invariant is **text-default, vision-explicit**, not the cost.

## 4. The render is anonymity-safe & leak-free

- **The human avatar renders identically** to puppets and automated agents — same model, same animation vocabulary, no badge, outline, or tell (Phase-3 representational anonymity, now proven in the 3D render). A screengrab can **never** single out the human.
- The image carries **no observer-only data** — no truth registers, no clue ledger, no meta-quest state, no labels, no "this is the human" — **nothing the AI's text perception wouldn't already hold.** It is a picture of the world, not an annotated map.
- **Determinism:** the screengrab is a **pure read** of `(world state, camera)` — it does **not** mutate the sim. The *decision to call it* is the model's (a recorded tool call in the log); `world.replay(log)` reproduces the **call** (and, where retained, the rendered frame) deterministically.

## 5. Surfacing & the client

- **The human god UI** lives in the **observer client**: controls to spawn/move/alter/trigger, plus the observer overlay marking **god-authored events** in the ground-truth panels.
- **The screengrab** renders from the authoritative scene and returns the image to the AI via the tool result; the AI also sees a confirmation in its own (text) channel that it took a look — **never** any observer-only data.
- **The OSRS client** continues to render the town for the watcher; the observer overlay notes when the AI **requested vision** (observer truth), distinct from the AI-facing stream.

## 6. Determinism & the actor model (restate and hold the line)

- The deterministic sim's **only** non-deterministic inputs are the **model**, the **human avatar**, and the **god channel** — **all recorded** to the event log.
- A **god perturbation** is a recorded exogenous event; a **vision request** is a recorded read. Neither adds unseeded randomness or wall-clock to the sim.
- `world.replay(log)` reproduces the **entire run** — including every god action and every screengrab call — to the same state hash. This is the gate the PM will lean on.

## 7. Tests (how the PM approves you)

Deterministic; kernel + mock driver (no model) for the sim paths; the render path tested for faithfulness/anonymity. Required:

1. **God actions are recorded & replayable:** each perturbation is an exogenous log event; `world.replay(log)` reproduces a god-perturbed run to the identical state hash; the sim stays pure (no new `Math.random`/wall-clock).
2. **Perturbation is diegetic (no attribution):** across `look`/`talk`/`status`/events, **no AI-visible field, flag, source, or timing tell** distinguishes a god-caused event from a seeded one — a god-moved NPC is shaped identically to a scripted one.
3. **God is human-only:** the AI's tool manifest contains **no** spawn/alter/trigger/perturb capability; the channel is reachable only by the operator.
4. **Screengrab faithfulness:** the vision tool returns an image that is a faithful render of the deterministic state from the AI's vantage at that tick.
5. **Text stays default:** vision is a **separate explicit tool**; the AI's base perception/navigation runs on native text without it (no path silently renders to navigate).
6. **Render anonymity:** in the rendered image, the human avatar is **indistinguishable** from puppets/agents — same model/animation, no tell; the Phase-3 anonymity guarantee holds in 3D.
7. **Render leak-free:** the image contains **no** register/clue/meta-quest/observer-only data and no labels — nothing the AI's text view wouldn't already have.
8. **Vision calls are recorded & replayable:** the tool call is logged; replay reproduces it (and any retained frame); the screengrab does not mutate the sim.
9. **Anonymity & noise floor intact:** the Phase-3/9/11 anonymity and noise-floor tests pass with **god mode and vision active**.
10. **No skills** anywhere; **Phases 0–12 still green** (or the not-yet-cleaned suites still pass).

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** two capabilities — a **human-only god channel** that perturbs the world through **recorded, diegetic exogenous events** the AI cannot distinguish from natural ones; and an **opt-in on-demand 3D vision** tool that returns a faithful, **anonymity-safe, leak-free** render from the AI's vantage while **structured text stays the default** perception — both fully **deterministic and replayable**, with the **box closed** and the **human hidden** throughout. **No skills.**

**Deliverables:** the god perturbation channel + recording (§1); the diegetic/observer-truth wiring (§2); the screengrab vision tool + render path (§3); render anonymity + leak-freedom (§4); surfacing + client (§5); the actor-model determinism guarantees (§6); the §7 tests green.

**Acceptance tests:** all of §7 pass; `pnpm test` green; determinism lint + replay-equality green with god actions and vision calls in the log; Phases 0–12 suites still pass; the Phase-3/9/11 anonymity tests pass with god mode and vision active.

**Invariant checks (PM rubric):**
- **Diegetic perturbation:** no god attribution in any AI-facing path; god events are shaped like seeded ones; observer sees the truth.
- **Determinism:** god actions + vision calls are recorded log entries; replay reproduces the run; sim stays pure.
- **Anonymity:** the human avatar renders identically to puppets/agents; neither a god action nor a screengrab exposes or narrows toward the human — test-proven.
- **Text-default, vision-opt-in:** vision is an explicit tool, not the base loop.
- **God is human-only:** no perturbation capability in the AI's manifest.
- **No skills.**

**Out of scope:** the observatory — logging pipeline, replay UI, run-analysis dashboards (Phase 14); number tuning — any vision cost/latency, perturbation defaults (Phase 15); hardening (Phase 16); continuity (the separate later build).

**Handoff — Phase 14+ may assume:** a **human-only god channel** writing recorded, diegetic exogenous events the AI can't distinguish from natural ones (with god-authorship visible only to the observer); an **opt-in 3D vision** tool returning faithful, anonymity-safe, leak-free renders from the AI's vantage, with **text still the default** perception; full **determinism and replay** with god actions and vision calls in the log; anonymity and the noise floor intact under both; **no skills.**

---

## Reminders
- **The AI can't tell a god from nature.** Perturbations enter as ordinary world events with zero attribution — that indistinguishability is the whole point.
- **Everything exogenous is recorded.** God actions and vision calls are log entries; that's how determinism and replay survive non-deterministic actors.
- **The render can never be a tell.** The human avatar looks like everyone else, and the image holds nothing the text view wouldn't.
- **Text first, vision on demand.** The screengrab supplements; it never becomes how the AI navigates.
- **God is the human's power, not the AI's** — no perturbation tool in the AI's manifest.
- **The box stays closed** — even a god acts inside the world's rules, not through the fourth wall.
- **No new randomness, no skills** — the sim stays pure; replay stays exact.
