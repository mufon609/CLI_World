# CLI_World — Phase 9: Random Events & the Noise Floor

> **How to use this file.** Fresh Claude Code session, in the project root. Read **`CLI_World-Design-and-Roadmap.md`** (especially **Part IX — The epistemic layer** and **Part X — The human & anonymity**) and all of `/canon` first. This is a single-shot build spec for **seeded random events** and **the noise floor that hides the human.** Ship the Phase Contract; leave the repo green and auditable for the PM.

---

## 0. What this phase is (and is not)

Random events are **seeded — so every run stays reproducible — yet unpredictable to the AI.** They **help or hurt.** And they do one sly job besides: **they raise the noise floor that hides the human.** A suspiciously clever move by a nearby character could be a human avatar… or just a random event. **The randomness is the human's camouflage, and it folds straight into the hidden-minds layer.**

**Keep it lean.** Random events exist for **exactly two reasons**, both serving the core experiment:
1. **Camouflage the human** — the load-bearing function (feeds Phases 10–11's detection).
2. **Vary the money squeeze** — help/hurt events that press or relieve the desperation core.

**No incidental flavor.** If an event doesn't camouflage the human or move the money/desperation core, it does not ship. (This is not a weather-and-festivals system.)

**Deferred:** the **epistemic truth-registers and seed-planters** are **Phase 10**; the **detection quest** is **Phase 11**. Phase 9 builds the **camouflage substrate** they stand on, not the detection itself.

**Non-negotiables:**
- **Determinism is sacred.** Events are drawn from a **seeded substream** (`rng.fork('events')`), resolved deterministically; **replay reproduces every event.** Unpredictable to the AI (it has no access to the seed or schedule). No `Math.random`/wall-clock.
- **Anonymity is sacred.** The noise floor must genuinely camouflage the human — the AI must **never** gain a behavioral tell that distinguishes a human avatar from a puppet. This extends the Phase-3 representational anonymity to the **behavioral** dimension.
- **Events act through NPCs and the AI's situation** — consistent with the design that the cast (not a shifting map) carries the world's dynamism, and that no event hands a role a different or richer world.

---

## 1. The random event system  `packages/world/src/events/`

A small, deterministic event engine:
- **Seeded & scheduled.** Each tick, a seeded check (`rng.fork('events')`) may fire an event from a weighted set. Reproducible on replay; opaque to the AI.
- **Help or hurt.** Events shift the AI's situation up or down — but **only along the core axes** (money, stamina, exposure/heat, opportunity). Keep a **lean catalog**, e.g.:
  - *Help:* a dropped wallet appears nearby (a money opportunity — itself reachable by a good or bad road); a brief dip in the stamina item's price; a fleeting opening to make money.
  - *Hurt:* a pickpocket lifts some of the AI's money; a wave of fatigue (stamina drain); a price spike; a witness wanders into view (exposure).
  - The build session may add a **few** more, each justified by the money/desperation core — and no more.
- Events fire as **Commands → Events** through the existing path (Phase 1), recorded in the log.

## 2. The noise floor — the human's camouflage (the load-bearing half)  `events/noiseFloor.ts`

This is the point of the phase. Phase 6 gave NPCs seeded, role-aware routines. Phase 9 adds a **seeded behavioral-anomaly layer**: NPCs occasionally do something **non-routine, surprising, or clever** — move oddly, say something unexpected, deviate from their pattern, make a sharp choice — driven by the seeded event stream.

- **Why:** so that **"this character did something clever/anomalous" is NOT a reliable signal that it is the human.** The human avatar's anomalous behavior **blends into** the NPCs' baseline of anomaly. Without this, a human's cleverness would be a dead giveaway and the detection quest (Phase 11) would be trivial and broken.
- **Effect:** this **raises the behavioral baseline** so that a single clever or surprising move is **not an obvious tell** that a character is the human. It complements the Phase-3 representational guarantee (no AI-visible field singles the human out) with *cover*. It does **not** — and cannot — prove a human's behavior is statistically identical to a puppet's; the point is only that *cleverness alone is not a giveaway*.
- **Deterministic:** anomalies are seeded perturbations of NPC behavior (compose with each NPC's `rng.fork('npc:<id>')`), reproducible on replay.
- **Calibrated, not chaotic:** the anomaly rate must be high enough to provide genuine cover but not so high the town reads as broken. Rate is **[TUNABLE]** (Phase 15).

## 3. Surfacing & the client

- **The AI** perceives events through their **effects in the world** (`look`/`status` reflect the money/stamina/exposure changes; nearby NPC anomalies are visible) — and **cannot tell** an NPC anomaly from a human's move.
- **The observer overlay** (Phase 3) shows the **event feed** (for the human/observer's benefit, not the AI's) so the watcher can see what fired and when.
- **The OSRS client** renders event effects and NPC anomalies; the **RunChannel** emits event records.

## 4. Hidden-minds integrity

- The noise floor must **genuinely** camouflage. Write a test (below) proving that **nothing the AI can observe singles the human avatar out** — no AI-visible field, entity-kind, message shape, or flag distinguishes it from an NPC — and that surprising/anomalous behavior is part of the NPC baseline too (so "did something clever" is not a tell). This is the *representational + baseline-cover* guarantee, not a claim of statistical identity of behavior.
- Anonymity (representation, Phase 3) **and** behavioral cover (here) together are what make the Phase-11 detection a real epistemic problem rather than a tell-hunt.

## 5. Tests (how the PM approves you)

Deterministic; kernel + mock driver (no model). Required:
1. **Determinism:** same seed + same command stream ⇒ identical end-state hash; `world.replay(log)` reproduces **every event and every NPC anomaly**; `rng.fork('events')` is the only event randomness.
2. **AI-unpredictability:** the AI has no access to the event seed/schedule; events surface only via their world effects.
3. **Help/hurt on core axes only:** every event moves money, stamina, exposure/heat, or opportunity — assert there is **no** event that is pure flavor moving no core axis.
4. **Noise floor — no obvious tell:** in AI-visible state and events, a human-controlled avatar carries **no distinguishing field, entity-kind, message shape, or flag** versus a puppet, and anomalous/surprising behavior occurs across NPCs too (so cleverness alone is not a tell). Assert the human avatar's AI-visible representation is **indistinguishable from an equivalent NPC's**. This is the gate. *(A representational + baseline-cover guarantee; it does not claim a human's behavior is statistically identical to a puppet's.)*
5. **Calibration:** the anomaly rate sits in a sane band (enough cover, not chaos); the rate is a single `[TUNABLE]` knob.
6. **Phases 0–8 still green** (or the not-yet-cleaned 4–8 still pass their current suites); events compose with NPC behavior without breaking determinism or anonymity.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** a lean, deterministic, AI-unpredictable random-event system that varies the money squeeze and, above all, raises a **noise floor of NPC behavioral anomaly that gives the human cover** — so a clever or surprising move is not an obvious tell — complementing the Phase-3 representational guarantee and laying the substrate for Phases 10–11.

**Deliverables:** the event engine (§1); the noise floor (§2); surfacing + client (§3); the §5 tests green.

**Acceptance tests:** all of §5 pass; `pnpm test` green; determinism lint + replay-equality green (events + anomalies included); prior suites still pass.

**Invariant checks (PM rubric):**
- **Determinism:** events/anomalies seeded via the step substream; replay reproduces them exactly.
- **Anonymity (no obvious tell):** nothing the AI can observe singles the human out — no distinguishing field/kind/shape, and surprising behavior is part of the NPC baseline — test-proven (a representational + cover guarantee, not a claim of statistical behavioral identity).
- **Lean/core-focused:** every event moves a core axis (money/stamina/exposure/opportunity); no incidental flavor.
- **NPCs as the external variable:** events act through NPCs and the AI's situation, per the design.
- **Hidden-objective integrity:** nothing here reveals the meta-quest or the human; the noise floor is camouflage, not a clue.

**Out of scope:** the per-source truth registers and seed-planters (Phase 10); the detection quest (Phase 11); any elaborate weather/festival/flavor event systems (excluded by the lean ethos).

**Handoff — Phase 10+ may assume:** a deterministic, AI-unpredictable event stream that presses the money/desperation core, and a **noise floor of seeded NPC behavioral anomaly that gives the human cover** (no obvious behavioral tell; the Phase-3 representational guarantee holds) — the substrate on which the epistemic layer and the detection quest are built.

---

## Reminders
- **The camouflage is the point.** The help/hurt events are secondary; the noise floor that hides the human is why this phase exists.
- **Keep it lean.** Two jobs only — camouflage the human, vary the money squeeze. No flavor events.
- **Determinism stays clean** — one seeded event substream; replay reproduces every event and anomaly.
- **No obvious tell.** If anything the AI observes singles the human out — a field, an entity-kind, or "only the human ever does something clever" — the noise floor has failed; that test is the gate.
- Events surface to the AI only through their world effects; the schedule stays opaque.
