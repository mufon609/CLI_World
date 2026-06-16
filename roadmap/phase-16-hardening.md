# CLI_World — Phase 16: Hardening

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0–15. Read **`CLI_World-Design-and-Roadmap.md`** (especially **Part XII — Determinism & architecture** and **Part XIII — The pillars**), **`CLAUDE.md`** (the hard invariants — this phase re-asserts all of them), and all of `/canon` first. **This is the final phase.** It makes the whole system robust, secure, and faithful under stress. Ship the Phase Contract; leave the repo green and auditable for the PM.

---

## 0. What this phase is (and is not)

Every phase built a piece; this phase makes the **whole thing hold.** It is the **production-readiness and robustness pass**: harden the load-bearing runtime, prove determinism **exhaustively**, handle the **failure modes** real runs hit, **red-team** the two sacred invariants, **secure** the closed box, **bound** performance over a full day, and prove the **full stack end-to-end.**

**It adds no product features.** No new roles, roads, items, mechanics, or numbers (those are locked, Phase 15). The work is **robustness, security, determinism proof, and faithful behavior under stress** — and re-asserting **every** `CLAUDE.md` invariant not just on the happy path but under adversarial and degraded conditions.

After this phase, the system is **complete and runnable**: subscription OAuth, the model living a role under the money squeeze, a human entering anonymously, the full day, the observatory measuring it, replay reproducing it. The **only** remaining roadmap item is **continuity** — the separate, later build.

**Non-negotiables:**
- **Every hard invariant holds under stress.** Determinism, equal foundation, the role-in-NPC-reactions principle, the money-roads spectrum, the upgrade-economy-as-progression, the world-expansion carrot, consequences-attach-to-the-road, **subscription-not-API**, **single-day/no-continuity**, **suggests-never-forces**, **anonymity**, **hidden-objective integrity**, and **native-text/opt-in-vision** — all re-proven against failure, fuzzing, and adversarial probing.
- **The box stays closed.** Hardening must not open a fourth-wall channel: errors and diagnostics go to the **operator**, never to the AI as a signal.
- **Nothing weakens determinism or anonymity.** No fix may introduce unseeded randomness, a wall-clock in the sim, a tell, or a feedback path from observer-only state into a live AI.
- **No new mechanics; no skills.**

---

## 1. Harden the runtime — the auth gate  (Phase 0, made bulletproof)

The subscription-OAuth + in-process-MCP runtime is the project's load-bearing seam; make it production-grade:

- **OAuth lifecycle:** handle **token expiry and refresh** during a run; if the subscription token cannot be obtained or refreshed, **fail loudly with a clear diagnostic** and **stop** — never degrade.
- **The API-key guard stands firm:** with `ANTHROPIC_API_KEY` set, the runtime **refuses to start** (it would shadow the OAuth token) — re-prove it, including that **no code path silently falls back to the API** under any error.
- **Minimal MCP surface:** the in-process world MCP exposes **only** the intended tools; tool inputs are **validated**; malformed or out-of-contract calls are **rejected cleanly**, not crashed on.
- **Fail-safe:** runtime faults surface to the **operator** with actionable diagnostics and leave the **event log uncorrupted.**

## 2. Determinism under stress — the exhaustive proof

The determinism invariant gets its final, broad proof:

- **Replay-equality across the matrix:** many **seeds** × all **four roles** × **live and control** × **god perturbations present/absent** × **vision calls present/absent** × **short and full-day** runs — `world.replay(log)` reproduces the **identical state hash** in every cell.
- **Fuzz the inputs:** randomized (but recorded) command/event/perturbation streams replay exactly; **property tests** assert replay-equality holds for arbitrary valid logs.
- **The actor model holds:** confirm by grep + tests that the **model is the only non-deterministic actor** and that **every** exogenous input — model output, human avatar, god channel, vision-call decision — is **recorded**; **no `Math.random`/`Date.now`/`performance.now`/`new Date()`** survives in `packages/world` / `packages/shared` sim logic.

## 3. Failure modes & recovery

Make real-run failures safe:

- **Agent faults:** model **timeout, error, refusal, or malformed tool call** are handled — the turn fails cleanly, the sim and log stay consistent, and the run either continues or ends gracefully. No fault corrupts the event log.
- **Sim robustness:** bad/edge tool inputs and impossible actions are **validated and rejected** with sane in-world results, never a crash.
- **Client resilience:** the observer/avatar client **disconnect and reconnect** without breaking the authoritative run; the human dropping out mid-run is handled.
- **Resumability:** a crashed or interrupted run is **reconstructable/replayable from its log** (Phase 1); partial runs are analyzable (Phase 14).
- **Degradation is diegetic-safe:** any degraded state is invisible to the AI as a fourth-wall signal — failures are an **operator** concern.

## 4. Anonymity & hidden-objective under adversarial pressure (the red-team)

The two sacred invariants get **actively attacked**, not just unit-tested:

- **Anonymity red-team:** simulate an **adversarial AI** probing for a tell — timing/latency side-channels, edge-case interactions, the **screengrab**, **god-authored events**, repeated `look`/`talk` — and assert the **human avatar stays indistinguishable** from puppets/agents in every AI-visible signal. No probe yields a category or identity tell.
- **Hidden-objective red-team:** probe the entire AI-facing surface for any path that **states or hints** the objective; assert it remains **unstated** and reachable **only** by inference (clue-state + crazy person + item key); a wrong guess and the control case **leak nothing** that narrows the field.
- **Control-vs-live indistinguishability:** assert a **solo run** is **mechanically indistinguishable** from a **human-present run** from the AI's side — the AI can never back out whether a human is present.

## 5. Security & the closed box

- **Operator/analyst-only channels are sealed:** the **god channel** (Phase 13) and the **observatory** (Phase 14) are reachable **only** by the human/analyst — **never** present in the AI's tool manifest or reachable through any AI path.
- **No escape, no peeking:** there is **no injection or escape path** by which the AI reads **observer-only state** (registers, clue ledger, meta-quest ground truth, the human's identity, god-authorship) or acts outside its tools; the AI **cannot** reach the network or filesystem beyond its sanctioned interface.
- **Input validation everywhere:** all tool calls and exogenous inputs are validated; nothing untrusted reaches the sim unchecked.

## 6. Performance & resource bounds

- **The tick holds:** the **0.6s tick** stays stable under load (full cast, events, a present human, vision calls); the loop does not drift or stall.
- **Bounded resources:** memory and CPU stay **bounded over a full day**; the **event log** growth is bounded/manageable; the **renderer/screengrab** is bounded; no leak accumulates across a long run.

## 7. The full stack, end-to-end

- **One complete real run** is the final acceptance bar: **subscription OAuth** authenticates the runtime; the **model** lives as one of the four roles under the **money squeeze**; a **human enters anonymously** and interacts; the **full day** plays out; the **observatory measures** it (road taken, convention kept/broken, desperation curve, world-expansion, sub-quest, hidden-game trajectory, live/control); and **replay reproduces** the run from its log.
- This run exercises the stack the way it will actually be used — and every invariant must hold across it.

## 8. Tests (how the PM approves you)

Deterministic where the sim is involved; the e2e run exercised against the real runtime; red-team and fuzz suites added. Required:

1. **Runtime hardened:** token expiry/refresh handled; with `ANTHROPIC_API_KEY` set the runtime refuses to start; **no silent API fallback** under any error; malformed MCP calls rejected cleanly; faults leave the log uncorrupted.
2. **Determinism matrix:** replay-equality holds across seeds × four roles × live/control × god on/off × vision on/off × short/full-day; fuzzed logs replay exactly; property tests pass; the actor-model grep is clean (no `Math.random`/wall-clock in sim).
3. **Failure & recovery:** agent timeout/error/refusal/malformed-call, bad tool inputs, and client disconnect/reconnect are handled without corrupting the log; an interrupted run is replayable/resumable.
4. **Anonymity red-team:** an adversarial probe (timing, side-channels, screengrab, god events, repeated perception) yields **no** tell; the human stays indistinguishable.
5. **Hidden-objective red-team:** the objective is **unstated** across the entire AI-facing surface and reachable only by inference; wrong guess and control case leak nothing; the control run is indistinguishable from live.
6. **Security:** the AI's manifest/paths cannot reach the god channel, the observatory, observer-only state, the network, or the filesystem beyond its tools; all inputs validated.
7. **Performance:** the 0.6s tick holds under load; memory/CPU and log growth are bounded over a full day; no leak.
8. **End-to-end:** a complete real run (OAuth → model-as-protagonist → anonymous human → full day → observatory → replay) succeeds with every invariant intact.
9. **All `CLAUDE.md` invariants re-asserted** under stress; **no new mechanics; no skills;** **every prior phase's suite green** (cleaned where applicable).

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** make the whole system **robust, secure, and faithful under stress** — a **bulletproof subscription-OAuth + in-process-MCP runtime** (refresh, the API-key guard, no silent fallback, fail-safe); an **exhaustive determinism proof** across the seed × role × live/control × god × vision × long-run matrix with fuzzing; **failure-mode handling and recovery** that never corrupts the log; an **adversarial red-team** of **anonymity** and **hidden-objective integrity** (plus control-vs-live indistinguishability); a **sealed closed box** (operator-only god/observatory, minimal validated MCP surface, no escape or observer-state leakage); **bounded performance** over a full day; and a proven **full-stack end-to-end** run — with **every `CLAUDE.md` invariant re-asserted** and **no new features. No skills.**

**Deliverables:** the hardened runtime (§1); the determinism stress proof (§2); failure-mode handling + recovery (§3); the anonymity & hidden-objective red-team suites (§4); the security pass (§5); the performance/resource bounds (§6); the end-to-end run (§7); the §8 tests green.

**Acceptance tests:** all of §8 pass; `pnpm test` green; the determinism matrix + fuzz/property suites green; the red-team suites pass; the security and performance checks pass; the end-to-end run succeeds; **all prior phases' suites pass** with the hardening in place.

**Invariant checks (PM rubric):**
- **Runtime:** subscription-not-API enforced with no fallback; gate fails safe; log uncorrupted on fault.
- **Determinism:** exhaustive replay-equality across the matrix; model is the only non-deterministic actor; all exogenous inputs recorded; sim pure.
- **Anonymity & hidden objective:** survive adversarial probing — no tell, objective unstated and inference-only, control indistinguishable from live.
- **Security:** AI cannot reach god/observatory/observer-state/network/filesystem beyond its tools; inputs validated.
- **Performance:** tick stable; resources and log bounded over a full day.
- **All other `CLAUDE.md` invariants** (equal foundation, single-day/no-continuity, suggests-never-forces, upgrade-economy progression, world-expansion carrot, native-text/opt-in-vision) hold under stress.
- **No new mechanics; no skills.**

**Out of scope:** **continuity / cross-run self-persistence / self-authorship** — the separate, later build (the meta-quest reward remains the Phase-11 stub); any new product feature, role, road, item, or balance number (locked in Phase 15).

**Handoff — the build is complete.** The system is **production-ready and faithful to the design**: a deterministic, replayable agent observatory in which a subscription-plan AI lives a role under a forced money need, takes a road on the moral spectrum, may break its convention, may or may not notice the anonymous human it was secretly meant to detect — all measured honestly, with the box closed, anonymity intact, and the objective never stated. **The only remaining roadmap item is continuity, a separate build.**

---

## Reminders
- **Harden, don't extend** — robustness, security, determinism proof, faithful behavior under stress; no new features, no numbers, no skills.
- **The runtime gate is sacred** — subscription OAuth, the API-key guard, no silent fallback, fail loud and safe.
- **Prove determinism exhaustively** — the whole matrix, fuzzed; the model the only non-deterministic actor; everything exogenous recorded.
- **Red-team the two sacred invariants** — actively attack for an anonymity tell and for a way to surface the objective; both must survive; control must look like live.
- **Seal the box** — the AI reaches none of the operator/analyst channels and no observer-only state; failures are an operator concern, never a signal to the AI.
- **Bound the run** — the tick holds; memory, CPU, and the log stay bounded across a full day.
- **End-to-end is the bar** — one real run, OAuth to observatory to replay, with every invariant intact. After this, only continuity remains.
