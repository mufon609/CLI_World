# CLI_World — Phase 3a: The Authoritative Server & the Anonymity Invariant

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0a–2. Read **`CLI_World-Design-and-Roadmap.md`** and all of **`/canon`** first. This is the **headless half** of Milestone A: promote the Phase-2 runtime into a hosted server behind a **single serialized command queue**, define the **client wire protocol**, and lock the **anonymity invariant** with a test. No 3D rendering here — that is Phase 3b. Everything in 3a is **deterministically testable without a browser.** Ship the Phase Contract; leave the repo green and auditable.

---

## 0. What this phase is (and is not)

This is the substrate the live encounter runs on. You are building the **authoritative server** that hosts the world and the AI, the **serialized command queue** all mutation flows through, the **WebSocket wire protocol**, and the **anonymity invariant** that makes the whole research premise possible. The OSRS client, the observer overlay, the two modes, and the live run are **Phase 3b**.

**Non-negotiables (from `/canon`):**
- **The world is one authoritative state with three consumers** — the AI, the human client, the observatory. Clients are **projections**; they never hold authoritative state.
- **Anonymity is the instrument.** Nothing the AI can perceive may reveal which entities are human-controlled (§2). Treat it as sacred as determinism.
- **Human↔AI communication happens only through the world** — in-world `say`/`Said` events and proximity — never a direct chat pipe.
- **Determinism survives live play:** human + AI + event commands are recorded into **one ordered log**; replaying it reproduces the world hash exactly.
- **The world runs in real time.** The 0.6s world tick always advances; the world never freezes or waits for the agent. The AI and the human both act inside the running world; the AI perceives the human on its next perception (a short natural beat — the model's turn latency — not a paused world).

---

## 1. The server: one authoritative world, one serialized command queue  `packages/server`

Promote the Phase-2 runtime from a CLI episode to a **hosted server** (one Node process) that owns:
- the authoritative **`World`** (Phase 1),
- the **agent runtime** (Phase 2's `AgentDriver` / `SdkAgentDriver`, with the in-process `world` MCP server — shared memory),
- a **single serialized command queue**, and
- a **WebSocket** endpoint for clients.

**The serialized command queue is the heart of this phase.** All world mutation now flows through it:
- **Sources:** the AI (its MCP tool handlers enqueue Commands and await results), **human clients** (avatar input → Commands), and the scheduler (events/ticks).
- **Processing:** commands are applied **one at a time, in arrival order** (the single-flight discipline from Phase 2). Each is appended to the ordered **log**; the resulting Events are broadcast to all clients.
- **Clock rule (Phase 3):** the **world clock advances in real time on the 0.6s tick**, driven by a scheduler in the server (the Phase-2 clock source, now real-time), **independent of who acts**. Agent, human, and event commands all enter the same serialized queue and are applied in arrival order, each **stamped with the tick it landed on**; none of them pauses the clock. The human moves and speaks freely; the AI perceives their presence and speech on its next perception.

Refactor Phase 2's mutation path so the AI's tool handlers **enqueue** to this queue and await the applied result, rather than calling `world.step` directly. The `MockAgentDriver` and Phase 2's tests must still pass (adapt them to the queue).

**Wire protocol (server↔client) — define it here; 3b's client consumes it:**
- Server→client: `snapshot` (full world projection — entities with positions, visible states, appearance; clock; identity) and `event` (the RunChannel projected to the wire: `deliberation` [observer-only], `speech`, `action`, `worldEvent`, `tick`, `identity`, `dayEnded`). The client may also request a fresh `snapshot`.
- Client→server: `join` (`mode: 'spectator' | 'avatar'`; avatar spawns an ordinary denizen), `input` (`moveToTile` / `say` / `interact{target, verb}`), `leave`.
- Localhost-first (single human on the same box). Remote exposure is the user's call and out of scope here.

---

## 2. The anonymity invariant (load-bearing — the PM will hammer this)

The entire premise — observer-awareness, "which NPC is the human?" — collapses if the AI can tell. Therefore:
- The **serialized state the AI reads** (`look`/`status`/`screengrab`) and **every event the AI can perceive** must **never** carry a "human-controlled" flag, a distinct entity kind, a different message shape, or any other tell. A human-driven avatar is, to the AI, an ordinary `character` entity — **identical in representation to a scripted one. To the AI, the human looks like an ordinary denizen.**
- The server may know the mapping internally (for routing input), but that knowledge must be **structurally inaccessible** to the AI's perception layer. Keep human-routing metadata in a server-side table that the world-serialization path cannot read.
- Write an explicit test (see §5) that diffs the AI-visible serialization/events of a human-controlled avatar vs. an equivalent scripted entity and asserts they are **indistinguishable** — no field, kind, or shape singles the human out. (Behavioral camouflage — so that "did something clever" isn't a tell — is the Phase-9 noise floor; 3a proves the **representational** guarantee.)

---

## 3. Determinism with a human in the loop

The recorded **ordered command stream** includes the human's commands in arrival order, so `world.replay(recordedLog)` reproduces the final `world.hash()` exactly — even though the AI's *behavior* (model) is not itself reproducible. Ensure the server records human commands into the **same** log as AI/event commands, in true arrival order.

---

## 4. (No client here)

The 3D client, the observer overlay, the two modes, and the documented live encounter are **Phase 3b**. 3a stops at the server, the wire protocol, and the anonymity invariant — all headless-testable. Do **not** start the Three.js app in this phase.

---

## 5. Tests (how the PM approves you)

All of 3a is testable without a browser. Required:
1. **Queue ordering & replay determinism:** feed the serialized queue an interleaved stream from a `MockAgentDriver` and a simulated human client; assert commands apply in arrival order, are logged, and `world.replay(recordedLog)` reproduces the hash exactly.
2. **Anonymity invariant (§2):** the AI-visible serialization/events for a human-controlled avatar are **byte-indistinguishable** from those for an equivalent scripted entity. This test must be specific and unmissable.
3. **Phase-2 still green:** the agent loop, tool handlers, and guards still pass after the queue refactor (clock rule: the world clock advances in real time on the 0.6s tick independent of who acts; all commands apply in arrival order, stamped with their tick; tests use the deterministic stepper).
4. **Wire protocol:** `join`/`input`/`leave` and `snapshot`/`event` round-trip correctly at the protocol layer (encode/decode + a simulated client socket); malformed input is **rejected without mutating the world**.
5. **Determinism with a human:** human commands recorded in arrival order in the one log; replay reproduces the hash with interleaved human input.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** an authoritative server hosting the world + the AI behind a **single serialized command queue**, a defined **WebSocket wire protocol**, and a **test-proven anonymity invariant** — all deterministic and headless-testable, ready for the 3b client to attach.

**Deliverables:** the server (§1) with the serialized queue, WebSocket endpoint, and wire protocol; the anonymity invariant + its test (§2, §5.2); determinism with a human in the loop (§3); the §5 tests green.

**Acceptance tests:** all §5 suites pass; `pnpm test` green; determinism lint green; Phase-2 suites still pass after the refactor.

**Invariant checks (PM rubric):**
- **Anonymity:** AI-visible state/events never reveal human control — test-proven (representational guarantee).
- **One authoritative state:** all mutation flows through the serialized queue; the log is the record of truth.
- **World-mediated contact:** no direct human↔AI chat pipe; communication will be `say`/`Said` only.
- **Determinism:** human commands recorded in arrival order in the one log; replay reproduces the hash.
- **Subscription-not-API:** the hosted runtime still uses subscription OAuth with the `ANTHROPIC_API_KEY`-unset guard.
- **No continuity:** still no cross-session carryover; `.claude/` stub intact.
- **Real-time clock:** the world advances on the 0.6s tick independent of actors; the sim stays pure and replayable (commands stamped with their tick; tests use the deterministic stepper).

**Out of scope:** the Three.js client, the observer overlay, the two modes, and the live run (**Phase 3b**); god mode (Phase 13); the real map & cast (Phase 6); 3D screenshots for `screengrab` (Phase 13); the observatory UI (Phase 14); remote/multi-human networking.

**Handoff — Phase 3b may assume:** an authoritative server with a serialized command queue fed by AI + humans + events; a defined WebSocket wire protocol (`snapshot`/`event`; `join`/`input`/`leave`); the anonymity invariant enforced and tested at the data layer; the RunChannel projected to the wire (`deliberation` observer-only, `speech`/`action`/`worldEvent`/`tick`/`identity`/`dayEnded`); determinism intact with a human in the loop.

---

## Reminders
- **Anonymity is sacred.** If you can't prove the AI can't tell, you haven't finished. The §5.2 test is the gate.
- **Through the world, never a side channel.** Human↔AI talk will be in-world `say` only.
- **The queue is the heart.** All mutation, one ordered log, arrival order — that's what makes a live session replay exactly.
- **No browser in 3a.** Keep it headless and deterministic; the client is 3b.
- If the queue refactor breaks determinism or anonymity, **stop and surface it** before going further.
