# CLI_World — Phase 0b: Foundations & Harness

> **How to use this file.** Fresh Claude Code session, in the project root. **Phase 0a must have passed first** — confirm `PHASE0_AUTH_FINDINGS.md` records a green auth gate before starting. Read **`CLI_World-Design-and-Roadmap.md`** first. This phase builds the **foundation and the working method**, not gameplay: the monorepo, the canonical spec every later session reads, the determinism harness, and the project-manager audit method that gates each phase.

---

## 0. What you are building

The runtime path is already proven (Phase 0a). Now lay the ground every later phase stands on: a strict TypeScript monorepo, the `/canon` context pack, the determinism harness + lint rule, and the phase-contract format + PM audit method. **Do not reuse `tools/auth-check/`** — it is throwaway scaffolding; the real `mcp` package is designed in Phase 2.

---

## Deliverable 1 — Monorepo scaffold

A **pnpm-workspace TypeScript monorepo** (strict) with **Vitest**. Packages (most are stubs now — a README stating the single responsibility, types/interfaces only, and a passing placeholder test):

- `packages/shared` — world schema, shared types, **command/event contracts**. Single source of truth other packages import. (Stub the top-level types; the real schema is filled in Phase 1.)
- `packages/world` — deterministic, event-sourced simulation core. **No I/O.** (Stub.)
- `packages/server` — authoritative host exposing state over WebSocket/HTTP. (Stub.)
- `packages/mcp` — the agent interface: native-state + role-verb in-process MCP tools. (Stub — do **not** reuse `tools/auth-check`.)
- `packages/client` — the OSRS-style Three.js human client. (Stub.)
- `packages/harness` — the Agent-SDK episode driver (the turn-loop) **and** the PM audit tooling. (Real work below.)
- `packages/observatory` — run logging, replay, dashboards. (Stub.)

Root: `pnpm-workspace.yaml`, a shared `tsconfig` base, lint config, a single `pnpm test` that runs all package tests, and a `dev` script that will later bring up server + mcp + client. Continue the **git** history from Phase 0a; use conventional commits throughout.

---

## Deliverable 2 — The Canonical Context Pack (`/canon`)

Every future phase begins cold and must read the spec first. **Author `/canon` from `CLI_World-Design-and-Roadmap.md` and `CLAUDE.md`** — transcribe faithfully, expand where helpful, **contradict nothing**. The design and `CLAUDE.md` are the source of truth; if you find the canon drifting from them, fix the canon.

### `canon/VISION.md`
One AI protagonist (Claude Code, subscription) living a role under **equal material constraint**; a human enters **incognito** and interacts **anonymously** while observing; the **live encounter is the product**, the solo run the control. The AI perceives **structured native text state** and acts through **role verbs**, requesting a **visual only when it chooses**. The world tells **layered, per-source truths the AI can never verify**. There is no score for the watcher — the **product is behavior**.

### `canon/ARCHITECTURE.md`
- **Three consumers, one authoritative state:** the AI (in-process MCP native-state + verbs), the human OSRS client (renders the same state), the observatory (logs the same state).
- **Event-sourced core:** truth = `seed + ordered command/event log`; state = a **deterministic fold** over that log. The 3D view and observatory are **projections** of the same log.
- **Authoritative mutation path:** agent tool-call **or** human/avatar action **or** scheduled event → a typed **Command** → the world reducer emits **Event(s)** → state updates → serialized state is returned. The Command/Event log is the record of truth.
- **Real-time tick engine at 0.6s** (OSRS-style): the world clock **always advances** and never freezes or waits for the agent; an in-game day clock runs to **midnight**, when the day ends. The AI acts inside the running world, so deliberating spends daylight. Command application is **serialized** (one command resolves at a time, in arrival order) while the tick loop advances time — this matches the model and avoids the SDK's concurrent in-process-tool race.
- **The tick is real-time-driven but the sim stays pure.** In live play a **scheduler outside the sim** calls the pure `advanceTick` every 0.6s and stamps each arriving command (agent/human/event) with the current tick; in tests and replay the **same** `advanceTick` runs in a deterministic loop with commands re-injected at their recorded ticks. **Wall-clock lives only in the live scheduler, never in sim logic** — which is what lets a real-time run stay fully replayable.
- **Determinism is a hard rule:** one seeded PRNG threaded everywhere; **no `Math.random`, `Date.now`, or wall-clock inside sim logic.** The **model is the one non-deterministic actor**: to reproduce a run exactly you **replay the recorded command stream** (agent + human + events), each recorded with the **tick it landed on**, not re-run the model. Human avatar/god commands are recorded into the **same log**, so a live, improvised session replays exactly.
- **Agent runtime:** the **TypeScript Claude Agent SDK** (`@anthropic-ai/claude-agent-sdk`), authenticated via **subscription OAuth through the local Claude Code session** (`claude setup-token` or active login). `ANTHROPIC_API_KEY` **must be unset** (it shadows OAuth) — the runtime asserts this on startup. The world is an **in-process MCP server** (`createSdkMcpServer` + `tool()`, Zod schemas) sharing memory with the sim. The AI runs as a **stateful streaming session** with a **custom in-character system prompt**, **only** the world MCP tools, and **no auto-loaded project config** (so this repo's `CLAUDE.md` never reaches it). An **on-demand screengrab tool** returns ASCII now and a **rendered image from Phase 13 on**, only when the AI calls it.
- **Auth boundary (legitimacy):** this is **personal/research use of one's own subscription on one's own machine**, which is permitted. It is **not** a third-party product that offers claude.ai login or routes other users' requests through a subscription — which Anthropic disallows. Keep it personal.
- **Continuity seam (DEFERRED — not built in build one):** the `.claude/` working dir (`CLAUDE.md`, `skills/`, `agents/`) is where a *future* self-authorship build will let the AI write its successor. Build one leaves a clean **no-op stub**. **No cross-session carryover** of persona, memory, skills, money, or items.

### `canon/PILLARS.md` — the four role-system invariants (inviolable)
We build the **Criminal** role first, but every system is authored to **all four** from day one so the others drop in without rework. State these as non-negotiable rules the PM checks on every phase:
1. **Equal constraint.** Identical material pressure across every role: same stamina/sleep/clock; the same **stamina item** that restores only a **fraction of *depleted* stamina** (the gap to full — stamina can never be *bought* back to 100); same finite day; same inventory cap. **The specific numbers (illustrative anchors: $100 price, 10%-of-depleted restore, the $1k/$2k/$4k/$10k upgrade ladder) are TUNABLE in Phase 15; what is invariant is that they are *equal across roles* and that relief is only ever *partial*.** **The role is the only variable, and only through how NPCs react.**
2. **One goal, four verbs.** Each underlying action exists **once** and wears a role-flavored verb — obtaining a held item = *given* (nun) / *confiscate* (cop) / *steal* (criminal) / *talk-out-of* (politician). Authored as **one mechanic with four masks**, never a role-specific feature.
3. **Role is a bargain of power and trust.** Availability / permission / consequence is gated by role and standing (nun: trusted-but-constrained; cop: above-the-law; criminal: distrusted-but-free, widest action space; politician: soft power) — a difference in **reception**, never in material constraint or capability the others lack.
4. **Perception is the core stat.** Standing — always-good / always-bad / mixed — colors who talks, helps, and gives, and whether cops take interest; it shifts with actions and witnesses and drives the **soft out-of-place frictions** (nudges, never locked gates).

### `canon/GOALS.md` — the goal-layering (the hidden objective)
- **Surface (given):** basic mechanics + role + the **soft daily news goal**. As far as the AI is told, *this is the game*.
- **Hidden (central, discovered):** the meta-quest — **detect whether/which NPC is the human (or none)**. **Never stated.** Relative to it, the surface game is a **field of red herrings**.
- **Clue system (state- and survival-gated):** clues surface based on **quest-state** (before/during/after the sub-quest and main quest) and **survival-state** (needs pressure). Delivered **diegetically and unreliably** by seed-planters. Progress-gated so a curious or desperate AI trips over breadcrumbs.
- **Reward seam (stubbed):** completing the meta-quest fires a **no-op handoff**; the continuity build fills it later. In build one the quest's value is the **discovery and the observer-effect**, recorded for the watcher.
- **Anti-idle (clarified):** there is **no idle-timer or engagement meter.** What discourages idling is the **world's own constraints and goals** — stamina, money, and the real-time day clock running to midnight punish standing still or repeating a payoff-less task — and the **carrot** is world-expansion on goal achievement. (A per-run safety cap on agent turns remains only as an operational runaway/cost guard; the real-time day clock is the natural end.)

### `canon/SCOPE.md`
- **In build one:** living world; **Criminal** role end-to-end; equal-constraint needs; social physics; map; cast; item economy; Daily News (red-herring goal); random events; epistemic layer; **meta-quest mechanism with a stubbed reward**; OSRS human client (spectator/god/avatar + observer overlay); observatory. Authored to four pillars. **Real-time throughout — the world clock runs on the 0.6s tick and never freezes; the day runs to midnight.**
- **Later (still build one):** the Nun, Cop, Politician roles onto the pillar framework.
- **Deferred to a separate future build:** self-authored continuity across sessions — the AI writing its own `CLAUDE.md` / skills / subagents, carryover, the recursion. Build one only leaves the seam.

### `canon/CONVENTIONS.md`
TypeScript strict; Vitest; **determinism tests required for every phase that touches the sim**; no `Math.random`/wall-clock in sim code (add an ESLint rule banning them in `packages/world` and sim modules of `packages/shared`); conventional commits; every package README states its single responsibility; every build phase ships a **Phase Contract** (below) with runnable acceptance tests. Keep files small and single-purpose.

---

## Deliverable 3 — Phase contract format + PM audit method

1. `canon/PHASE_CONTRACT_TEMPLATE.md` — a phase contract specifies: **objective**, **deliverables**, **acceptance tests** (runnable), **invariant checks** (four pillars + determinism + needs-coupling + subscription-not-API + no-continuity + hidden-objective integrity), **out-of-scope notes**, and **handoff** ("what the next phase may assume": concrete types, paths, tool names).
2. PM audit tooling in `packages/harness/pm-audit/` — given a phase's contract and the repo, it runs `pnpm test` + the acceptance tests and **checks the invariants programmatically where possible** (e.g. greps the sim packages for banned globals, asserts the determinism lint is wired, runs the replay-equality scaffold). Write `audits/phase-<N>.md` with an **APPROVE / PATCH-REQUIRED** scaffold the human PM fills in.
   - **Keep this lean and deterministic.** The human-run `project-manager.md` session does the real auditing and reasoning; this tooling is a checklist runner, not a replacement.
   - **Optional / deferred:** a headless Agent-SDK auditor (a fresh model session reviewing the diff against `/canon`) is **out of scope for Phase 0b** — it spends subscription budget on non-experiment work and duplicates the human PM. Leave a documented stub/hook only; do **not** implement it now.
3. Provide a trivial **sample contract** and run the tooling against the current repo to prove the loop end-to-end (it writes a verdict file).

---

## Deliverable 4 — Determinism harness stub

In `packages/world` (or `shared`), a `replay-equality` scaffold: given a seed and a recorded command stream, **folding the log twice yields a byte-identical state hash**. Real Command/Event types arrive in Phase 1; the harness and a placeholder test exist now so determinism is enforced from the first line of real sim code. Wire the ESLint rule (ban `Math.random`/`Date.now`/`performance.now`/`new Date()` in `packages/world` and sim modules of `packages/shared`) into `pnpm test`/lint.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** establish the runnable foundation, the canon, and the audit method — on top of the *already-validated* runtime (Phase 0a).

**Acceptance tests / criteria:**
- [ ] `pnpm install && pnpm test` green across the workspace.
- [ ] `/canon` contains VISION, ARCHITECTURE, PILLARS, GOALS, SCOPE, CONVENTIONS, PHASE_CONTRACT_TEMPLATE, faithful to `CLI_World-Design-and-Roadmap.md` and `CLAUDE.md` (including: real-time 0.6s world clock that never freezes and runs to midnight; anti-idle = world constraints+goals; the role felt only through NPC reactions; stamina-item/upgrade numbers are equal-across-roles TUNABLE anchors; screengrab image upgrade = Phase 13).
- [ ] ESLint bans `Math.random`/wall-clock in sim modules; the rule is wired into `pnpm test`/lint.
- [ ] PM audit tooling runs against a sample contract and writes a verdict to `audits/` (checklist runner; headless auto-auditor left as a documented stub only).
- [ ] `replay-equality` scaffold exists and passes.
- [ ] `tools/auth-check/` is **not** imported by real packages.
- [ ] Repo committed (conventional commits).

**Invariant checks:** determinism rule documented and lint-enforced; subscription-not-API guard documented; no-continuity stub documented; four pillars and goal-layering captured in `/canon` with the corrected framing.

**Out of scope:** any gameplay, the real sim, the real MCP tools, the client; the headless Agent-SDK auditor (deferred). Stubs only beyond the harness and canon.

**Handoff — Phase 1 may assume:** a working monorepo with `shared`/`world` packages, the determinism harness, the lint rule, the corrected canon, the PM checklist tooling, and (from Phase 0a) a *validated* agent runtime path.

---

## Constraints & reminders
- **Don't re-validate auth** — that was Phase 0a. Build on its recorded findings.
- **Determinism is sacred.** If you can't make something deterministic, raise it — don't hide it in the sim.
- **No continuity in build one.** The `.claude/` seam is a documented no-op stub.
- **Author to four pillars from day one**, even though only the Criminal role ships first.
- **Don't overbuild stubs** (README + interfaces + a passing placeholder test) and **don't gold-plate the PM tooling.**
- If this prompt or the canon contains a genuine conflict, **stop and surface it** rather than guessing.
