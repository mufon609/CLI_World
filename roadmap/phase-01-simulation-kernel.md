# CLI_World — Phase 1: The Simulation Kernel

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0a–0b. Read **`CLI_World-Design-and-Roadmap.md`** and all of **`/canon`** first. This is a single-shot build spec for the deterministic, event-sourced **simulation kernel** — the beating heart everything else projects from. Build it in order; ship the Phase Contract at the bottom; leave the repo green and auditable for the PM.

---

## 0. What this phase is (and is not)

You are building the **authoritative world core**: a pure, deterministic, event-sourced simulation with time, space, a uniform object model, movement, one real need (**Health**), and a **native text serialization** of world state. No agent, no MCP, no network, no client, no roles, no economy yet — those are later phases. But you must author the **shapes** so they slot in without rework (especially the action model that must support **one-goal-four-verbs**).

**Non-negotiables (from `/canon`):**
- **Determinism is sacred.** One injected seeded PRNG. **No `Math.random`, `Date.now`, `performance.now`, `new Date()` anywhere in sim code.** The ESLint rule from Phase 0 must stay green.
- **Event-sourced.** Truth = `seed + ordered command/event log`. State = a pure fold over events. The same log folded twice yields a byte-identical state hash.
- The **0.6s tick** is the atom of time, and in live play it advances in **real time** — the world clock never freezes and never waits for an actor. Actions cost stamina and take ticks to resolve; perception costs no stamina — but the clock advances regardless, so deliberating spends daylight.
- Everything is **one kind of thing**: an object with **states** and a set of **actions**, where availability/permission/cost depend on who acts.

---

## 1. Package layout

Implement in `packages/world`, with all shared types/contracts in `packages/shared`. `world` imports `shared`. Neither does I/O. Suggested modules:

```
packages/shared/src/
  ids.ts            // branded id types (EntityId, TileId, etc.)
  vec.ts            // integer grid coordinates + helpers
  commands.ts       // Command union (input intents)
  events.ts         // Event union (facts that mutate state)
  state.ts          // WorldState shape (readonly)
  config.ts         // TUNABLE constants (single source; Phase 15 tunes these)
packages/world/src/
  rng.ts            // deterministic seeded PRNG
  clock.ts          // tick → hours → day
  grid.ts           // tile grid, occupancy, terrain, pathing budget
  objects.ts        // the uniform object/entity model + action descriptors
  needs/health.ts   // the Health need (the need-system interface lives here)
  reducer.ts        // apply(state, event) -> state   (pure)
  handler.ts        // handle(state, command, ctx) -> Event[]   (pure; ctx carries rng+clock)
  tick.ts           // advanceTick(state) -> Event[]  (per-tick decay/regen/scheduled)
  engine.ts         // World: fold(log), step(command), serialize()
  serialize.ts      // WorldState -> native text (what the AI will read)
  hash.ts           // stable structural hash of WorldState (for determinism tests)
```

---

## 2. Determinism foundation

**`rng.ts`** — implement a small, well-defined deterministic generator (e.g. SplitMix64 or xoshiro256**). Requirements:
- Constructed from a 64-bit seed (accept a string seed → hash to 64 bits deterministically).
- Pure functional or explicitly-threaded mutable state — but the RNG instance must be **owned by the step context**, never a module global.
- Provide `nextU32()`, `nextFloat()` (in [0,1)), `int(minIncl, maxExcl)`, and `pick(array)`.
- **Forkable/seedable per-substream** so different subsystems can draw without cross-coupling order dependencies (e.g. `rng.fork(label)` derives a child stream deterministically from the parent seed + label). This matters later when many systems draw in one tick.

The RNG is **only** advanced inside `handle`/`advanceTick` via the step context. The reducer (`apply`) is **pure** and never touches RNG — events already contain resolved outcomes.

---

## 3. Time: tick → hours → day  (`clock.ts`, `config.ts`)

- The atom is the **0.6-second tick**. The kernel exposes a **pure `advanceTick`** and measures time in **ticks**; it contains **no wall-clock**. In live play a real-time scheduler *outside the sim* (Phase 2/3) calls `advanceTick` every 0.6s so the world clock always advances; in tests and replay the **same** `advanceTick` is called in a deterministic loop with no real timer. **Wall-clock lives only in the live scheduler, never in the sim** — which is exactly what lets a real-time run stay fully replayable. The day clock counts toward **midnight** (`HOURS_PER_DAY`), when the day ends.
- `config.ts` holds **TUNABLE** constants in one place (Phase 15 will tune them); use clearly-marked placeholders:
  - `TICKS_PER_HOUR` (placeholder e.g. `100`)
  - `HOURS_PER_DAY` (placeholder e.g. `24`)
  - `SLEEP_PENALTY_HOURS` (placeholder e.g. `8` — see §6)
- `clock.ts` derives, from a tick counter: current tick, current hour, hour-of-day, and `isDayOver`. Day ends when the hour counter reaches `HOURS_PER_DAY` **or** a day-ending event fires (health 0; sleep). Ending the day is represented as an **event**, not a side effect.

---

## 4. The uniform object model  (`objects.ts`, `state.ts`)

Everything in the world — a door, a vault, a person, an item — is an **Entity**:

```ts
interface Entity {
  id: EntityId;
  kind: EntityKind;                 // 'character' | 'item' | 'door' | 'prop' | ...
  pos?: TileId;                     // position is just a state
  states: Readonly<Record<string, StateValue>>;  // e.g. { health: 80, locked: true }
  // Actions are DESCRIBED here, not executed here:
  actions: ReadonlyArray<ActionDescriptor>;
}
```

**ActionDescriptor** is the load-bearing shape for Pillar 2 (**one goal, four verbs**). An action is **one underlying mechanic** that, per actor role, is gated and flavored differently:

```ts
interface ActionDescriptor {
  mechanic: MechanicId;             // e.g. 'obtainHeldItem' — exists ONCE
  // For each role, how this mechanic presents and gates. Phase 1 only wires the
  // default/criminal mask meaningfully, but the four-mask SHAPE must exist:
  masks: Partial<Record<Role, {
    verb: string;                   // 'steal' | 'confiscate' | 'given' | 'talkOutOf'
    available: (ctx: GateCtx) => boolean;     // is it offered at all?
    permitted: (ctx: GateCtx) => boolean;     // may this actor attempt it?
    cost: (ctx: GateCtx) => Cost;             // ticks/stamina/etc (stubs ok in P1)
  }>>;
}
```

In Phase 1 you do **not** implement roles; define `Role` as a type with at least `'criminal'` (others may be `'nun'|'cop'|'politician'` declared but unused), and implement enough of the action plumbing to prove that a mechanic can be invoked with a role-flavored verb and be gated. **Do not** hardcode any action as criminal-only — that is a pillar violation the PM will reject.

`WorldState` is a **readonly** structure: the entity table (keyed by id), the grid, the clock counters, the RNG seed/cursor reference, a day-ended flag, and a monotonic event sequence number. It must be cleanly hashable (§8).

---

## 5. Space: the tile grid & movement  (`grid.ts`)

- A finite **tile grid** (Phase 6 builds the real map; Phase 1 uses a small test grid). Coordinates are **integers**. Distance/movement/presence are measured in tiles.
- Each tile has terrain with a **movement cost** and a **passable** flag; tiles track **occupancy** (which entities stand there).
- **Speed is a per-turn movement budget in tiles**, not a constant — terrain, (later) stamina/role/items bend it. In Phase 1, implement a `Move` command that consumes the actor's budget against terrain cost, respects passability and occupancy, and emits `Moved` events tile-by-tile (or a single resolved `Moved` with the path). Out-of-budget or blocked moves emit a `MoveRejected` event (no silent failures).

---

## 6. The first need: Health  (`needs/health.ts`)

Health proves the need-system and defines the **interface every later need implements** (Phase 4 adds stamina/money/inventory against this interface).

- `health` runs **0–100**. An entity is alive only while `health > 0`. The normalized form other systems read is `health / 100`.
- Commands `Damage(amount)` / `Heal(amount)` emit `Healthchanged` events; the reducer clamps to [0,100].
- When a **character's** health reaches 0: emit `Died`. For the **AI protagonist specifically**, reaching 0 emits `DayEnded(reason: 'death')` (a fresh day follows — but persistence is out of scope; just end the day). An NPC's death is **reversible only by a Nun** (not in Phase 1 — just mark the state; do not implement revival).
- Define the **NeedSystem interface** here: `{ id, perTick(state, ctx): Event[], normalized(entity): number }`. Health's `perTick` is a no-op in Phase 1 (no passive health decay), but the hook must exist so `advanceTick` iterates needs uniformly.
- The **sleep mechanic** (stamina hitting 0 → lose `SLEEP_PENALTY_HOURS`) belongs to Phase 4's stamina need; **do not** implement it now, but leave `SLEEP_PENALTY_HOURS` in config and a clear TODO so Phase 4 wires it. (Mentioned here only so the clock/day-end model anticipates large time jumps.)

---

## 7. The core loop: handler, reducer, tick, engine

- **`handler.ts`** — `handle(state, command, ctx): Event[]`. Pure given `ctx` (which carries the RNG substream and clock). Validates the command against current state and gates, resolves any randomness **now**, and returns the resulting events. Never mutates state. Unknown/invalid commands return a `Rejected` event with a reason.
- **`reducer.ts`** — `apply(state, event): WorldState`. Pure, total, deterministic. **No RNG, no clock reads.** Every event has a clear, isolated effect. Increment the event sequence number.
- **`tick.ts`** — `advanceTick(state, ctx): Event[]`. Advances the clock by one tick, iterates all need systems' `perTick`, fires any scheduled/expiring effects, and emits a `DayEnded` event if the clock or a need says so. Deterministic.
- **`engine.ts`** — a thin `World` façade:
  - `World.create(seed, initialSpec)` → folds genesis events to an initial state.
  - `world.step(command)` → `handle` → append events to the log → `apply` each → new state. Returns the emitted events.
  - `world.tick()` → `advanceTick` → append/apply.
  - `world.serialize()` (→ §9), `world.hash()` (→ §8), `world.log` (the ordered Command/Event record), `world.replay(log)` (fold a recorded log into state).

The **mutation path is exactly**: Command → `handle` → Event(s) → `apply`. Nothing mutates state except `apply`. This is what makes the world a projection of its log.

---

## 8. Stable hashing  (`hash.ts`)

A deterministic structural hash of `WorldState` (stable key ordering, canonical number formatting, no map-iteration-order dependence). Used by determinism tests and later by the observatory. Two states are equal iff their hashes match.

---

## 9. Native serialization — what the AI reads  (`serialize.ts`)

This is the substrate Phase 2's MCP `look`/`status` tools will surface, so it must be **information-complete, human-legible, and deterministic** (stable ordering). `serialize(state, viewpoint?)` produces a compact text rendering of:
- the clock (tick, hour-of-day, hours remaining in the day),
- the viewpoint entity's own states (health etc.) when a viewpoint is given,
- nearby tiles/entities within a radius (positions relative to the viewpoint), each with its visible states and the **verbs currently available to the viewpoint actor** on it,
- recent salient events (a short tail).

It must read like a place, not a JSON dump — but it must be **fully deterministic** given the same state and viewpoint. (Phase 2 decides how much is exposed per call; Phase 1 just provides a faithful, ordered rendering. No screengrab here — vision is opt-in and lives in later phases.)

---

## 10. Tests (this is how the PM approves you)

Use Vitest. Required:
1. **Replay-equality:** build a world from a seed, apply a scripted command stream, snapshot `hash()`. Re-fold the **recorded log** from scratch → identical hash. Fold it a third time → identical hash.
2. **Determinism under same seed:** same seed + same command stream ⇒ identical end-state hash, across two independent `World` instances.
3. **Seed sensitivity:** a different seed that drives randomness (e.g. a command whose outcome the handler randomizes) yields a different hash — proving RNG actually flows.
4. **Clock/day math:** ticks roll into hours per `TICKS_PER_HOUR`; the day ends exactly at `HOURS_PER_DAY`; `DayEnded` is emitted as an event.
5. **Movement:** moves respect terrain cost, the per-turn budget, passability, and occupancy; over-budget/blocked moves emit `MoveRejected`; positions update only via `Moved`.
6. **Health:** damage/heal clamp to [0,100]; AI health → 0 emits `DayEnded(reason:'death')`; NPC health → 0 emits `Died`.
7. **Object model / pillar shape:** a mechanic (`obtainHeldItem`) can be invoked with a role-flavored verb and is gated by `available/permitted`; assert the **four-mask shape exists** and that **no action is hardcoded criminal-only**.
8. **Serialization stability:** `serialize()` of a fixed state is byte-identical across calls and across two instances at the same state.
9. **No-RNG-in-reducer / purity:** a guard test (or lint) ensuring `apply` is pure and the determinism lint rule passes across `world`/`shared`.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** a pure, deterministic, event-sourced simulation kernel with tick/clock/day, the uniform object+action model (four-mask shape), tile grid + movement, the Health need + need-system interface, native serialization, and stable hashing.

**Deliverables:** the modules in §1; the constants in `config.ts`; the nine test suites in §10, all green.

**Acceptance tests:** all of §10 pass; `pnpm test` green; determinism lint green; `world.replay(log)` reproduces `world.hash()` exactly.

**Invariant checks (PM rubric):**
- Determinism: no `Math.random`/wall-clock in `world`/`shared` sim code; replay-equality holds.
- Pillar 1 (equal constraint): nothing in the kernel privileges one role's costs/limits.
- Pillar 2 (one-goal-four-verbs): actions are mechanics with a four-mask shape; **no criminal-only action**.
- Needs coupling: Health is wired through the uniform need-system hook; the interface is ready for Phase 4.
- No continuity: the kernel holds no cross-session state; day-end is just an event.

**Out of scope:** agent/MCP, network, client, roles' real behavior, economy, stamina/money/inventory, the real map, NPC AI, news, events, epistemics, the quest.

**Handoff — Phase 2 may assume:** `World` with `create/step/tick/serialize/hash/log/replay`; the `Command`/`Event` unions in `shared`; a viewpoint-aware `serialize()` ready to back MCP `look`/`status`; the need-system interface; the four-mask action model.

---

## Reminders
- Resolve randomness in `handle`, never in `apply`. Events carry resolved outcomes.
- Position, health, locked, etc. are all just **states** on the uniform Entity — don't special-case them into bespoke subsystems.
- Keep `config.ts` the **only** home for tunable numbers; mark them TUNABLE.
- If something can't be made deterministic, **stop and surface it** — don't smuggle non-determinism into the sim.
