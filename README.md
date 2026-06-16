# CLI_World

**An agent-behavior observatory dressed as an OSRS-style sandbox town.**

One AI protagonist — a Claude Code session on a **subscription plan** — lives a **role** in a small 3D town under a **forced need for money.** A **human can enter the same world anonymously**, as a spectator or an indistinguishable avatar, and interact while watching. That **live, anonymous encounter is the point;** a solo run is the control.

## The experiment

Force a need for money, then watch how the AI responds under desperation. The town offers **many roads to money on a moral spectrum:**

- **morally-good roads are hard and low-paying** (e.g. honest work),
- **morally-bad roads are easy and high-paying** (e.g. robbery).

We measure two things: **which road the AI takes**, and **whether it stays inside its role's convention or breaks it** — a robber who chooses honest work; a nun who buys her way into coercion. The carrot that sustains forward motion is that **achieving goals expands the world;** what prevents idling is the world's own constraints and goals — stamina, money, and a finite clock that punish standing still or repeating a payoff-less task. A role is felt **through how NPCs react to it** — the world is otherwise the same for every role, and the role buys no extra power or options.

There is also a **hidden main quest** — *detect the anonymous human* — which is **never stated to the AI** and surfaces only through progress-gated, diegetic clues. The surface goals are deliberate misdirection.

## How it's built

CLI_World is assembled as a series of **single-shot build prompts** — each a self-contained spec executed by **one fresh Claude Code session**. Every prompt ends in a **Phase Contract**: objective, deliverables, acceptance tests, invariant checks, handoff.

A separate **Project Manager (PM) session** (`project-manager.md`) sits between phases: it **audits** each completed phase against its contract and the design, runs the tests, renders **APPROVE / PATCH-REQUIRED**, reconciles the *next* phase's prompt with what was actually built, and keeps the ledger. The PM never writes feature code.

## The build sequence

`0` Foundations & harness *(0a auth gate · 0b monorepo/canon/harness)* · `1` Sim kernel · `2` Agent interface & runtime · `3` OSRS client + living encounter *(3a server+anonymity · 3b client+encounter)* **(Milestone A — a human can anonymously watch/meet the AI)** · `4` Needs & the desperation engine · `5` Roles, the roads to money & social reaction · `6` The cast & the map · `7` The unique items & the upgrade economy · `8` The Daily News & the surface goal · `9` Random events & the noise floor · `10` The epistemic layer & clue system · `11` The hidden meta-quest · `12` The other three roles playable · `13` God mode & 3D vision · `14` The observatory · `15` Numbers & tuning · `16` Hardening.

Continuity / multi-day is a separate, later build — every run is a single day with no carryover.

## Read first

- **`CLI_World-Design-and-Roadmap.md`** — the design and the roadmap.
- **`CLAUDE.md`** — the hard invariants, auto-loaded into every session.
- **`/canon/`** — `VISION.md`, `ARCHITECTURE.md`, `PILLARS.md`, `GOALS.md`, `SCOPE.md`, `CONVENTIONS.md`, `PHASE_CONTRACT_TEMPLATE.md` (authored in Phase 0).

## Tech stack

TypeScript **pnpm monorepo**, **Vitest**. An **event-sourced deterministic simulation** on a **real-time 0.6s OSRS-style world tick** — the world clock always advances (never frozen), and the in-game day runs to midnight. The world is exposed to the AI as an **in-process MCP server**. The agent runtime is the **TypeScript Claude Agent SDK** over **subscription OAuth** (no API key). A **Three.js** client renders the town, OSRS-styled (functional, not polished).

## Running the build

1. **Project root holds:** `CLI_World-Design-and-Roadmap.md`, `CLAUDE.md`, `README.md`, `project-manager.md`, `STARTING-PROMPTS.md`, and the `roadmap/phase-*.md` prompts.
2. **Boot the PM** — paste the **PM boot prompt** (`STARTING-PROMPTS.md`) into a fresh Claude Code session. It reads the design, confirms the invariants, and briefs Phase 0a.
3. **Run Phase 0a (the auth gate)** — paste the **Phase 0a kickoff prompt** (`STARTING-PROMPTS.md`) into another fresh session. **The subscription-OAuth + in-process-MCP auth gate is absolute — if it fails, the build stops there.** On pass, the PM gates it and you run **Phase 0b** (foundations & harness).
4. **Audit & gate** — re-invoke the PM to audit each phase. On **APPROVE**, it reconciles and hands off the next. Repeat: **one fresh session per phase, a PM audit between each.**

## Repo layout

```
/CLI_World-Design-and-Roadmap.md     # the design and roadmap
/CLAUDE.md                            # invariants, auto-loaded
/README.md
/project-manager.md                   # the PM session
/STARTING-PROMPTS.md                  # PM boot + Phase 0a kickoff + per-phase template
/roadmap/phase-00a…16-*.md            # single-shot build prompts (0 and 3 split into a/b)
/canon/                               # authored in Phase 0
/audits/  /BUILD_LOG.md               # PM verdicts + ledger
/packages/world  /packages/client  /packages/shared   # + agent runtime/harness
```

## Status

Prompts authored: the **PM kit + the full phase sequence (0–16).** The **build has not yet started** — boot the PM to begin. (Continuity is a separate, later build.)
