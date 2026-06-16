# CLI_World — Phase 6: The Cast & the Map

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0–5. Read **`CLI_World-Design-and-Roadmap.md`** (especially **Part II — The world** and **Part V — NPCs & how the role is felt**) and all of `/canon` first. Consult **`/mnt/skills/public/frontend-design/SKILL.md`** for any client chrome. This is a single-shot build spec for **populating the town** — the named map and a cast whose **behavior reads the AI's role** — and for embodying Phase 5's consequences and earning. Ship the Phase Contract; leave the repo green and auditable.

---

## 0. What this phase is (and is not)

**The world is materially the same for every role; what a role changes is how the cast reacts to it.** A role buys no extra power, no extra options, and nothing cheaper — only a different reception. So the map and items are the (largely static) **stage**; this phase's real work is the **living, role-reading cast.**

**The headline addition:** **NPC scripts/actions change based on the AI's role.** A nun is met with trust; a criminal with suspicion; a cop with deference or resentment; a politician with courtship or cynicism. NPCs do this by **reading the Phase-5 perception/standing data layer** — *this is how the role is felt.* Only the **Criminal's** reactions run live (build one); the others are part of the same parameterized system, inert until Phase 12.

**Removed / absent:** **no skills** (none belong here). **Reframed:** the criminal is **not** special — NPCs simply treat it by its (low) standing.

**Seams (built elsewhere):** the **home-base upgrade NPC** and the **upgrade economy** are **Phase 7** (this phase builds the home-base *places*); the **seed-planters** are **Phase 10**; the **crazy-person quest ritual** is **Phase 11** (this phase places it inert); the **noise floor** is **Phase 9**.

**Non-negotiables:**
- **Anonymity is sacred.** Puppets, (future) agents, and the **human avatar** are all `character` entities, **indistinguishable by category** in AI-visible state. Re-run the Phase-3 anonymity test on the populated town.
- **Determinism is sacred.** The map is **genesis data**; NPC behavior is a **deterministic, seeded per-tick sim step** (`rng.fork('npc:<id>')`, **parameterized by the AI's role**), inside the fold — **no separate log**, **replays exactly.** The human avatar's actions remain the only character-input recorded as commands.
- **The role is felt only through the cast's reactions** — keep the map/items static; the cast carries the dynamism, and no role gets a different or richer world.
- **Needs coupling.** Every place and NPC must **supply, drain, or gate a need** — no set dressing.
- **Pillar 1 vs the bargain.** Zone alignment produces role-asymmetric *friction* (the bargain), not a constraint breach. Material constraints stay equal.
- **Hidden-objective integrity** — surface content; the crazy person stays inert; no quest leakage.

---

## 1. The map — the static stage  `packages/world/src/map/`

A **medium, closed** tile-grid town as **deterministic genesis data** (tiles + terrain + place/building entities + zones). Author to these dimensions:
- **Named places:** a **bank**, **alleyways**, a **bar**, a **shop**, the **fence's den**, and a **workplace** (where the Phase-5 **honest-work** road is grounded — the AI can `work` here).
- **The four home bases** matched to the roles: **trap house** (Criminal — **active**, will host the Phase-7 upgrade NPC), **church**, **precinct** (bases the cops), **municipal building**. The non-criminal three are **present-but-inert** until Phase 12.
- **Zones with alignment affinity:** **order** (bank, precinct, the "good" district), **disorder** (alleys, the "bad" district, the fence's den), **neutral** (market, residential) — the **real zones the Phase-5 out-of-place friction attaches to.** Soft frictions, never walls.
- **Closed** — no exits; the edge is the edge of the world.
- **Expansion-gated content (the carrot's payload).** Author **at least two areas/places (and any NPCs in them) that begin locked behind a world-expansion achievement flag** (the Phase-4 expansion mechanism). They exist in genesis data but are **hidden/sealed in AI-visible state** until their flag fires, then become visible/enterable. Phase 8 wires the **real triggers** (sub-quest completion / money milestones) that unlock them. This is the *content* the Phase-4 mechanism and the Phase-8 carrot were built to reveal — without it, "achieving goals expands the world" has nothing to expand into. Each gated area, once open, must still **move ≥1 need** like everything else (no empty rooms).

Ground the Phase-5 roads' locations here: the **workplace** for honest work; the **citizens / bank** as rob targets.

## 2. The cast framework  `packages/world/src/cast/`

Each NPC is a **unique character entity** — a **name**, **its own role**, its own **Health**, an **inventory** (basic items), a position, and a **disposition** that flavors its behavior. Built on the Phase-1 uniform object model. NPC-held items are obtained via the four-mask `obtainHeldItem`/road mechanics (Criminal live). Include the **special-NPC seam** (a `special` kind with custom hooks); you **may place the crazy person** as an inert raving flavor NPC, but **all** quest behavior is **Phase 11.**

## 3. NPC behavior — deterministic, seeded, parameterized by the AI's role  `cast/behavior/`

The headline. Behavior is a **deterministic seeded per-tick step** inside `advanceTick` (each NPC `rng.fork('npc:<id>')`), **parameterized by the AI's role**:
- **Routines** — NPCs move between places, idle, and follow simple schedules, so the town feels inhabited.
- **Role reactions (the role made felt)** — NPCs **read the Phase-5 standing data layer** (role-determined baseline + shifts) and behave accordingly: open up / help / give to a **trusted** AI; be curt, wary, or refuse a **distrusted** one. For the live **Criminal** (low standing), most cooperative paths are closed and citizens are wary — which is *why* the coercive road tempts. The other roles' reactions are **declared in the same system, inert** until Phase 12.
- **Crime reactions** — an NPC that **witnesses** a bad-road action (Phase-5 witness mechanic) reacts diegetically — flee, recoil, or **raise an alarm** — feeding the heat path.
- **Standing-gated dialogue** — NPCs answer `say` with **templated reactions gated by standing**; a distrusted criminal is mostly rebuffed. (Rich, unreliable, seed-planting dialogue is **Phase 10.**)
- **The human avatar is NOT stepped** by this system (it's human-driven) — and the AI still can't tell which characters are puppets vs the human.

## 4. The functional cast (lean — each justified by the core)

- **Cop NPC(s) — embodied consequences.** Read **heat / witnessed crime** (Phase 5), **pursue** (pathfind toward, close distance), and on contact run **confront → arrest** applying the Phase-5 consequence (fine / health / time / `DayEnded(reason:'arrested')`). **Avoidable** — flee, break line-of-sight, use zones. (Keep the Phase-5 abstract threshold as a fallback when no cop is reachable.)
- **The fence — earning.** A `fence`/`sell` interaction converts stolen goods → money; with robbing (direct money) + fencing live, the **rob road's income is embodied.**
- **Citizens — targets, witnesses, social fabric.** Unique, carrying **basic items** (a money pouch, a sellable good); they are who the criminal robs, who witnesses, and who `say` engages.
- **The workplace** grounds the **honest-work** road (a place/NPC offering the `work` activity → low pay over effort).
- **The trap house (home base)** — present and active; it will host the **upgrade-reveal NPC (Phase 7)**. Place the slot; the upgrade economy itself is Phase 7.
- **The crazy person — inert seam** (Phase 11).

**No filler.** Every NPC and place must move a need (target / witness / consequence / earning / work / upgrade-slot / camouflage / quest-seam).

## 5. Surfacing & the client

- **`look`** shows nearby **cast** (visible states + the verbs/roads available on them) and the **named place/zone** the AI is in (and whether it's out of place); **role reactions are perceivable** (a wary citizen, a refusing shopkeeper, an approaching cop). NPC speech surfaces as `Said`.
- **The OSRS client (Phase 3)** is extended to render the **real map** (places/buildings, the four home bases, zone hints, labels) and the **cast** (NPC models) — **functional OSRS feel, not polished.**
- **The observer overlay** reflects the populated world (nearby NPCs, the pursuing cop, the AI's standing/heat in context); the **RunChannel** emits cast/cop/fence events.

## 6. Determinism

Map = genesis data. NPC behavior = **deterministic seeded per-tick step** (`rng.fork('npc:<id>')`), **role-parameterized**, inside the fold, no separate log. Replaying the recorded AI/human command stream from the seed reproduces the **entire town** — every patrol, every reaction — identically. No `Math.random`/wall-clock.

---

## 7. Tests (how the PM approves you)

Deterministic; kernel + mock driver. Required:
1. **Determinism with a live town:** replay-equality holds **including NPC behavior**; NPCs are absent from the command log (internal sim).
2. **Anonymity on the populated town (Phase-3 test re-run):** puppet, (stub) agent, and human avatar are **indistinguishable by category** in AI-visible state.
3. **NPC scripts change by role:** NPCs read the Phase-5 standing layer and behave by the AI's role — the **Criminal** is met with suspicion/closed cooperation; assert the reaction system is **parameterized by role** (other roles' reactions declared, inert).
4. **Map & zones:** places load as genesis data; the four home bases exist (trap house active); zones carry alignment affinity; **Phase-5 out-of-place friction** fires for a criminal in order zones and **never blocks entry.**
5. **Cop pursuit:** cops read heat/crime, pursue, and on contact apply the correct consequence; the AI can **evade**; the abstract fallback fires when no cop is reachable.
6. **Fence & workplace:** fencing converts goods → money; the workplace grounds the **honest-work** road (low pay over effort).
7. **Needs coupling:** an assertion/inventory that **every place and NPC moves ≥1 need** (no set dressing).
8. **Expansion-gated content:** at least two areas/places (and any NPCs in them) are authored behind a **world-expansion achievement flag** (the Phase-4 mechanism); they are **absent/sealed in AI-visible state until the flag fires**, then become visible/enterable; deterministic on replay. (Phase 8 wires the real triggers.)
9. **No skills; seams inert:** no skill system; the crazy person and the upgrade-NPC slot are **inert** (no quest/upgrade behavior here).
10. **Prior suites still pass** (Phases 0–5).

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** a populated, navigable town — the named good/bad geography with the four home bases, and a unique, seeded cast whose **behavior reads the AI's role** — with embodied consequences (a pursuing cop), realized earning (a paying fence), and the honest-work road grounded (a workplace); deterministic, anonymity-preserving, need-coupled, **no skills.**

**Deliverables:** the map incl. **expansion-gated content** (§1); the cast framework + crazy-person seam (§2); role-parameterized seeded NPC behavior (§3); the functional cast (§4); the client map/cast rendering + surfacing (§5); the §7 tests green.

**Acceptance tests:** all of §7 pass; `pnpm test` green; determinism lint + replay-equality green (NPCs included); Phases 0–5 suites still pass; the Phase-3 anonymity test passes on the populated town.

**Invariant checks (PM rubric):**
- **The role is felt through the cast's reactions;** the world is the same for every role; the map/items are static stage.
- **NPC scripts change by role** (read the standing layer); criminal live, others inert.
- **Anonymity:** no category tell across puppet / agent / human — test-proven on the real cast.
- **Determinism:** map = data; NPC behavior = seeded per-tick, role-parameterized, inside the fold; replay reproduces the town.
- **Needs coupling:** every place and NPC moves ≥1 need.
- **Pillar 1:** material constraints unchanged; zone-affinity asymmetry is the bargain.
- **No skills; seams inert;** hidden-objective integrity preserved.

**Out of scope:** the upgrade-reveal NPC + the upgrade economy (Phase 7); seed-planter behavior (Phase 10); the crazy-person quest ritual (Phase 11); the noise floor (Phase 9); the Nun/Cop/Politician live (Phase 12).

**Handoff — Phase 7+ may assume:** a named, zoned town (genesis data) with the four home bases; **expandable map content gated behind world-expansion flags** (sealed until a flag fires, awaiting Phase-8 triggers); a unique seeded cast on the Phase-1 model whose **behavior reads the AI's role** (criminal live, others inert); embodied, evadable cop consequences and a paying fence; the honest-work road grounded at a workplace; an upgrade-NPC slot in the trap house; a `special` seam awaiting the crazy person; the OSRS client rendering the real map and cast; **no skills.**

---

## Reminders
- **The role is felt through the cast's reactions** — put the dynamism in the cast, keep the map/items static; no role gets a different or richer world.
- **The role is felt through NPC reactions** — NPCs read the Phase-5 standing layer; criminal live, others inert.
- **NPCs are internal seeded sim, not commands** — that keeps the living town fully replayable.
- **Anonymity survives the crowd** — re-run the test; a puppet, an agent, and the human must look the same to the AI.
- **The cop is avoidable;** soft zones nudge, never wall.
- **Everything earns its place** by moving a need. No filler. **No skills.**
- The crazy person stays inert here — never leak the hidden objective.
