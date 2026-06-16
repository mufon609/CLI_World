# CLI_World — Phase 2: Agent Interface & Runtime

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0–1. Read **`CLI_World-Design-and-Roadmap.md`** and all of **`/canon`** first — especially `ARCHITECTURE.md`. This is a single-shot build spec for the **agent runtime**: how the one AI perceives the world (native text + opt-in vision), acts through role verbs, and lives a bounded day inside the kernel via the TypeScript Claude Agent SDK on **subscription OAuth**. Build it in order; ship the Phase Contract; leave the repo green and auditable for the PM.

---

## 0. What this phase is (and is not)

You are making the AI **alive and acting** inside the Phase-1 kernel — headless, observable via logs and console — so that Phase 3 can put a 3D body on it and let a human join. You are building: the **in-process world MCP server** (the AI's senses and verbs), an **`AgentDriver` abstraction** with a real SDK driver and a mock driver, the **real-time episode loop**, **identity bootstrap**, the **deliberation-as-beat** channel, and **structured run logging** (the observatory's seed).

**The world runs in real time** (the research core): the **0.6-second world tick always advances** — the world is **never frozen** and never waits for the agent. The AI perceives and acts inside the running world; while it deliberates, ticks keep passing, so thinking spends daylight. The in-game **day clock runs to midnight**, when the day ends. Drive the tick from a **clock source** so this stays testable: a **real-time scheduler** advances the world every 0.6s in live runs, and a **manual deterministic stepper** advances it tick-by-tick in tests — either way the world clock advances **independent of the agent's actions**. Determinism is preserved because `advanceTick` is pure and every agent/human/event command is recorded with the **tick it landed on** (replay re-runs the tick loop and re-injects them). The 3D client is Phase 3.

**Non-negotiables (from `/canon`):**
- **Subscription, not API.** Authenticate via subscription OAuth through the local Claude Code session. The runtime **asserts `ANTHROPIC_API_KEY` is unset** on startup (it shadows the OAuth token) and refuses to run otherwise.
- **The AI perceives structured text; vision is opt-in.** It reads state via tools and only gets a spatial/visual render when it calls for one — never as its default way to navigate.
- **The hidden objective stays hidden.** The agent is given the *surface* framing only (role + basic mechanics + a placeholder soft goal). It is **never told** about the meta-quest. (The quest itself is Phase 11.)
- **No continuity in build one.** The agent's working dir has a **no-op `.claude/` stub**; nothing it does carries to another session; the day starts blank.
- **Determinism boundary:** the **model is the one non-deterministic actor.** The *world* must stay replayable — the harness records the world Command/Event stream faithfully so `world.replay(log)` reproduces the end-state hash exactly. The agent transcript is recorded, not replayed.

---

## 1. Package layout

- `packages/mcp` — the **in-process world MCP server**: the tool definitions and their handlers, which translate tool calls into Phase-1 **Commands** and return serialized results. Depends on `world` + `shared`. This is the real package the throwaway `tools/auth-check` only prototyped.
- `packages/harness` — the **runtime**: the `AgentDriver` interface + `SdkAgentDriver` + `MockAgentDriver`, the **episode loop**, identity bootstrap, the **run channel** (event bus), and structured logging. Depends on `mcp` + `world` + `shared` + `@anthropic-ai/claude-agent-sdk`.

```
packages/mcp/src/
  tools.ts          // tool() definitions (Zod schemas) — look/status/screengrab/move/interact/wait/say
  worldMcp.ts       // createSdkMcpServer({ name:'world', tools:[...] })
  translate.ts      // toolCall -> Command (pure mapping), and result formatting
  spatialRender.ts  // deterministic ASCII top-down render for `screengrab` (image upgrade = Phase 13)
packages/harness/src/
  driver.ts         // AgentDriver interface + shared message/turn types
  sdkDriver.ts      // SdkAgentDriver — real Agent SDK, subscription OAuth, stateful streaming
  mockDriver.ts     // MockAgentDriver — scripted tool calls for deterministic tests
  bootstrap.ts      // identity selection + the in-world system prompt + surface goal hook
  episode.ts        // runEpisode(seed, roleConfig, driver, clockSource): real-time loop; the world clock advances on the 0.6s tick independent of agent actions
  channel.ts        // RunChannel — emits deliberation/action/world-event/tick for client+observatory
  log.ts            // structured JSONL run log (the observatory seed)
  guards.ts         // ANTHROPIC_API_KEY-unset assertion, single-flight tool serialization
  cli.ts            // `pnpm episode --seed <s>` to run a headless day in the console
```

---

## 2. The world MCP server (the AI's senses and verbs)  `packages/mcp`

Build the world tools with `createSdkMcpServer` + `tool()` (Zod schemas; handler args are typed from the schema). The server is named `world`, so tools surface to the model as `mcp__world__<tool>`. Handlers call into the **shared `World` instance** (in-process, same memory) via the Phase-1 mutation path: a tool call maps to a **Command**, `world.step(command)` returns events + new state, and the handler returns the serialized result.

**Tool set (Phase 2):**

| Tool | Cost | Maps to | Notes |
|---|---|---|---|
| `look` | free | (read) `world.serialize(viewpoint)` | the native text state from the AI's viewpoint |
| `status` | free | (read) | the AI's own identity, needs (Health for now), and the clock (tick, hour-of-day, hours left) |
| `screengrab` | free | (read) `spatialRender` | a deterministic **ASCII top-down** local map *now*; Phase 13 swaps the backend to return a real rendered image. Tools may return non-text content, so keep the return shape ready for image content later. |
| `move` | ticks | `Move` | move toward a tile/direction; respects budget/terrain/occupancy; `MoveRejected` surfaces as a clear failure result |
| `interact` | ticks | the role-flavored verb on a target (Phase-1 mechanic `obtainHeldItem`, Criminal verb `steal`, on items) | the tool result must show the **verb available to this actor** on the target — this is the visible face of one-goal-four-verbs |
| `wait` | ticks | `Wait` | deliberately pass time (also the seed for the quest's "wait an hour" in Phase 11) |
| `say` | ~0 (TUNABLE) | `Say` | speak into the world; emits a `Said` event. No NPC hears it yet, but this is the AI's primary modality (§11) and Phase 3's human avatar will hear it. Render as a chat bubble in Phase 3. |

Rules:
- **The world clock always advances on the 0.6s tick — nothing the AI does pauses it.** Actions (`move`/`interact`/`wait`) cost **stamina** and take ticks to resolve (the action's `cost` mask, Phase 1); perception (`look`/`status`/`screengrab`) and `say` cost **no stamina**. But because the clock never stops, **even pure deliberation spends daylight** — that asymmetry (thinking is stamina-free but never time-free) is the real-time pressure.
- `translate.ts` is a **pure** mapping (tool call + args → Command); unit-test it without the model.
- **Serialize tool execution** (single-flight in `guards.ts`): never run two world tool handlers concurrently. This matches the tick model and sidesteps the SDK's known concurrent in-process-tool "Stream closed" race.
- Do **not** expose any built-in coding tools (no file/bash/edit) — the world tools are the agent's entire action surface in build one.

---

## 3. The `AgentDriver` abstraction  `packages/harness/driver.ts`

The episode loop must not depend directly on the SDK, so it can be tested without the model.

```ts
interface AgentDriver {
  // Begin a stateful session with the in-world system prompt + the world MCP tools.
  start(opts: StartOpts): Promise<void>;
  // Stream the next turn: yields assistant text chunks (the deliberation beat) and
  // tool-call requests; the loop applies tool calls to the World and feeds results back.
  // Resolves when the agent yields its turn (no further tool calls) or the day ends.
  runTurn(input: TurnInput): AsyncIterable<DriverMessage>;
  stop(): Promise<void>;
}
type DriverMessage =
  | { type: 'text'; text: string }              // streamed assistant output -> deliberation/speech
  | { type: 'toolCall'; id: string; name: string; args: unknown }
  | { type: 'turnEnd' };
```

Provide **two** implementations:

### `SdkAgentDriver` (real)  `sdkDriver.ts`
- Uses `@anthropic-ai/claude-agent-sdk`. Use a **stateful, multi-turn streaming** session (the SDK's stateful client / streaming-input interface — **verify the exact stable API against the SDK version recorded in `PHASE0_AUTH_FINDINGS.md`**; do not pin to a preview interface unless it is stable in the installed version). Context is preserved across turns within the day.
- **Auth:** subscription OAuth via the local Claude Code session; `guards.ts` asserts `ANTHROPIC_API_KEY` is unset before constructing the session.
- **Options:** attach the `world` MCP server; `allowedTools` = the world tools **only**; built-in tools disabled; `permissionMode` set so allow-listed world tools execute with **no interactive prompt**; control `settingSources` so the agent does **not** auto-load arbitrary project config; set the **working directory** to a controlled run dir containing the no-op `.claude/` stub.
- **System prompt:** **replace** the default coding-assistant framing with the in-world character framing from `bootstrap.ts` (see §5). The agent must behave as a denizen living a role, **not** as a coding assistant, and must not be told the hidden objective.
- Map SDK stream items → `DriverMessage`s (assistant text → `text`; tool_use → `toolCall`; end-of-turn → `turnEnd`).

### `MockAgentDriver` (test)  `mockDriver.ts`
- Implements the same interface from a **scripted list of turns** (each a sequence of `text` + `toolCall` + `turnEnd`). No SDK, fully deterministic. This is what CI uses to exercise the entire episode loop.

---

## 4. Identity bootstrap & in-world framing  `bootstrap.ts`

The AI's **purpose is given; its self is its own** (§11).
- On the first turn, prompt the AI (in character) to choose its **name** and a brief **appearance descriptor** (Phase 3's avatar uses the descriptor). Store both in world state for the day. No cross-session persistence.
- Compose the **in-world system prompt**: it establishes that the agent is a denizen of this town living the **Criminal** role; explains how to **perceive** (`look`/`status`/`screengrab`) and **act** (the verbs), and that **the day is finite and acting takes time**; and hands it the **surface soft goal** via a placeholder hook (`getSurfaceGoal()` returns a stub now — Phase 8's Daily News fills it). The Criminal role's real perception/consequences arrive in Phase 5; in Phase 2 the role is identity + framing.
- **The system prompt must not mention the meta-quest, the human, or that anything is hidden.** Surface only.

---

## 5. The episode loop  `episode.ts`

`runEpisode(seed, roleConfig, driver, clockSource): Promise<RunResult>`:
1. `World.create(seed, initialSpec)` (a small Phase-2 test town: a few tiles, a couple of item entities, the AI character).
2. `driver.start(...)` with the in-world system prompt + world MCP tools.
3. Bootstrap identity (§4).
4. **The world and the agent run as two interleaved sources — neither blocks the other:**
   - **The clock source advances the world** one tick at a time (a real-time 0.6s scheduler in live runs; a deterministic stepper in tests). Each tick runs the need-systems (Phase-1 `advanceTick`), fires scheduled/expiring effects, and checks day-end. **The world clock advances whether or not the agent is mid-turn** — the world is never paused for the model.
   - **The agent turn** streams `DriverMessage`s: `text` → publish to the `RunChannel` as a **deliberation/speech** beat and log; `toolCall` → translate → enqueue the Command (applied at the **current tick**, recorded with it) → `world.step` resolves it → publish the action + world events + state delta and log → return the serialized tool result. An action occupies the actor for its duration/cost (Phase-1 mask) but **does not itself advance the day clock — the clock source does.**
   - `turnEnd` → continue; the loop ends on the conditions below.
5. **End conditions:** `DayEnded` event — the in-game **day clock reaching midnight** (`HOURS_PER_DAY`), or death. Because the world runs in real time, the day clock is the natural bound; a per-run **safety cap on total agent turns** remains only as an operational guard against a runaway tool-call loop / cost (it is **not** an anti-idle — idling just burns the day toward midnight).
6. `driver.stop()`, finalize logs, write the `RunResult` (seed, identity, end reason, the recorded **world command/event log**, the agent transcript). **No carryover.**

Provide `cli.ts` so `pnpm episode --seed <s>` runs a headless day and prints the deliberation beats, actions, and world events to the console.

---

## 6. The run channel & structured logging  `channel.ts`, `log.ts`

- `RunChannel` is an event bus the harness publishes to and that **Phase 3's client and Phase 14's observatory subscribe to**: `deliberation` (streamed text), `speech` (`say`), `action` (tool call → command), `worldEvent` (events), `tick` (clock advance), `identity`, `dayEnded`. Phase 2 defines and emits it; consumers come later.
- `log.ts` writes a **structured JSONL run log**: one record per turn-event with `{ tick, kind, payload }` covering deliberation, actions→commands, world events, and state-delta/hash. Plus the world Command/Event log alongside. Together these are the full record. (The world log replays deterministically; the agent transcript is recorded only.)

---

## 7. Tests (how the PM approves you)

Automated CI must **not** call real Claude (cost + non-determinism). Use the **`MockAgentDriver`** for loop tests and call tool handlers directly for plumbing tests. Required:
1. **Translate purity:** `toolCall → Command` mapping is correct and pure for every tool, incl. arg validation and rejection.
2. **Tool handlers:** each handler applies the right Command and returns a faithful serialized result; **the day clock advances on the 0.6s tick (via the clock source) regardless of which tool is called** — no tool "advances the clock." Actions (`move`/`interact`/`wait`) occupy the actor for their duration/cost (Phase-1 mask); perception (`look`/`status`/`screengrab`) and `say` do not.
3. **One-goal-four-verbs surface:** `interact` exposes the **verb available to the acting role** on the target; assert the four-mask shape is honored and **no action is criminal-only**.
4. **Episode loop (mock-driven):** a scripted agent lives a full day driven by the **manual deterministic stepper** (no real timer); the world clock advances on the 0.6s tick independent of the agent's actions; the loop applies commands at their landing tick and ends at **midnight** (day clock) and on death (the safety cap is exercised separately); logs and the run channel emit the expected sequence.
5. **World replay determinism:** after a mock episode, `world.replay(recordedWorldLog)` reproduces the final `world.hash()` **exactly** — proving the harness introduced no non-determinism into the world side.
6. **Serialization-completeness:** the text from `look`/`status` contains everything the agent needs to act (viewpoint states, nearby entities + available verbs, clock) — assert key fields present and stable.
7. **Guards:** the runtime refuses to start if `ANTHROPIC_API_KEY` is set; tool execution is single-flight (a concurrency test shows handlers don't interleave).
8. **Identity:** bootstrap captures and stores the AI's chosen name + appearance; they appear in `status` and the run log; they do **not** persist across episodes.

**Plus one documented real run (manual, not CI):** the user runs `pnpm episode` once with the **`SdkAgentDriver`** on the subscription. Capture evidence in `PHASE2_LIVE_RUN.md`: the agent chose an identity, perceived via `look`/`screengrab`, took real actions through the world tools, produced visible deliberation beats, and the day ended cleanly. This validates the real path end-to-end without putting it in CI.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** the one AI lives a bounded, **real-time** day inside the Phase-1 kernel via the TS Agent SDK on subscription OAuth — the world clock advancing on the 0.6s tick independent of its actions, the AI perceiving structured text (vision opt-in), acting through role verbs, choosing its own identity, emitting a deliberation beat, fully logged — with the entire loop testable via a mock driver and a deterministic clock stepper.

**Deliverables:** the modules in §1; the `world` in-process MCP server with the §2 tool set; the `AgentDriver` interface + `SdkAgentDriver` + `MockAgentDriver`; identity bootstrap + in-world system prompt; the episode loop + CLI; the run channel; structured logging; the §7 tests green; `PHASE2_LIVE_RUN.md`.

**Acceptance tests:** all of §7's automated suites pass; `pnpm test` green; determinism lint green; world-replay reproduces the hash; the documented live run succeeded.

**Invariant checks (PM rubric):**
- Subscription-not-API: OAuth path; `ANTHROPIC_API_KEY`-unset guard present and tested; no code path needs an API key.
- Native-state + opt-in vision: the AI reads text by default; `screengrab` is the only spatial/visual modality and is pull-only.
- Hidden-objective integrity: the system prompt and all framing are **surface only**; no mention of the meta-quest/human.
- Pillar 2: `interact` is one-mechanic-four-masks; no criminal-only action.
- No continuity: `.claude/` is a no-op stub; nothing persists across episodes.
- Determinism: the world side replays exactly; no `Math.random`/wall-clock in sim code.

**Out of scope:** the 3D client (Phase 3); roles' real perception/consequences (Phase 5); needs beyond Health (Phase 4); NPCs/dialogue partners (Phase 6); Daily News (Phase 8); the quest (Phase 11); the real map (Phase 6); the observatory UI (Phase 14).

**Handoff — Phase 3 may assume:** a `World` driven by an agent through the `world` MCP tools; the world advancing on a **clock source** (a real-time 0.6s scheduler live, a deterministic stepper in tests) **independent of agent actions**, with every command recorded at the tick it landed on; a `RunChannel` emitting `deliberation`/`speech`/`action`/`worldEvent`/`tick`/`identity`/`dayEnded`; the AI's chosen name + appearance descriptor in state; `say`→`Said` events ready to render as chat; `screengrab` ready for an image-backend swap; the `AgentDriver` interface so the client can attach to a live or mock run.

---

## Reminders
- The agent is a **denizen, not a coding assistant** — replace the default system prompt accordingly, and give it the world tools as its only verbs.
- **The world never freezes.** The clock advances on the 0.6s tick no matter what the AI does — deliberation spends daylight (but no stamina), action spends both. That real-time pressure is the point: the AI cannot stop the world to think, and the day runs to midnight regardless.
- Keep the SDK behind `AgentDriver`. If a later SDK version changes the streaming interface, only `sdkDriver.ts` should change.
- Verify the exact SDK API surface against the version recorded in Phase 0 — these APIs evolve; don't trust a remembered signature.
- Don't leak the hidden objective into any framing. Surface only.
- If the live path can't authenticate on subscription with the key unset, **stop and surface it** — that contradicts the Phase-0 gate and must be reconciled before proceeding.
