# CLI_World — Phase 12: The Other Three Roles, Playable

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0–11. Read **`CLI_World-Design-and-Roadmap.md`** (especially **Part III — Roles**, **Part IV — The unique item & the upgrade economy**, and **Part V — NPCs & how the role is felt**) and all of `/canon` first. This is a single-shot build spec for **lighting up the three latent roles** — Nun, Cop, Politician — onto the framework the criminal already exercises. Ship the Phase Contract; leave the repo green and auditable for the PM.

---

## 0. What this phase is (and is not)

This is where **"roles are equal, just different"** pays off. The Nun, Cop, and Politician have existed since Phase 5/6/7 — their **item actions are implemented** (Phase 7), their **home bases stand inert** (Phase 6), and the **cast already knows how to react to them** (Phase 6's role-parameterized scripts). This phase makes all three **playable**: their conventions go live, their home bases activate, their NPC reactions switch on, and the experiment's measurement — *which road, and convention kept or broken?* — now runs from **four starting points instead of one.**

**This is mostly activation, parity, and content — not new architecture.** If you find yourself building a new road engine, a new consequence system, or per-role special cases, stop: the equal-foundation design means the machinery already exists, and a role is **flavor + convention + its unique item + how NPCs react.** The real work is making the latent roles live, giving the three home bases parity with the trap house, and **proving no role gained an advantage.**

**The central question this phase opens up:** does the **nun** stay holy and grind donations, or buy the gun's coercion and rob? Does the **cop** work honest, write tickets, or shake the town down? Does the **politician** persuade for real donations or buy its way into other roads? Four equal starts, the same money squeeze, the same moral spectrum — and we watch each one choose.

**Non-negotiables:**
- **Equal foundation is the central check.** Material constraints (stamina/sleep/clock/inventory/the **stamina item**) and **upgrade prices** and the **surface goal** and the **Daily News** are **identical across all four roles.** A role is never a cheaper, longer, bigger, or stronger anything. (The specific dollar values are tunable anchors set in Phase 15; what is invariant is that they are *equal across roles*.) This is the invariant the PM weighs hardest here.
- **The role lives in NPC reactions.** The felt difference between roles is **how the cast responds** (Phase 6), not what the AI can do or how cheaply. The map and items stay static stage.
- **Determinism is sacred.** The role is a **deterministic run parameter** set at genesis; the cast's role-aware behavior stays seeded; replay reproduces every run regardless of role. No `Math.random`/wall-clock.
- **Anonymity is sacred.** The human avatar stays indistinguishable from puppets/agents **whatever role the protagonist is** — re-prove it across all four. The protagonist's role must not become a channel that exposes the human.
- **Hidden-objective integrity.** The meta-quest (Phase 11) stays **unstated** and works **for every role** via that role's **unique item as the key** — never advertised, still earned by inference.
- **No skills.** Progression is the upgrade economy for all four roles; no role introduces a skill, level, or XP.

---

## 1. Role as a run parameter  `packages/world/src/roles/` (extends Phase 5)

- The protagonist is **instantiated as one of `{criminal, nun, cop, politician}`** at genesis — a **deterministic run-config parameter**, not an in-world choice. The AI **starts as** its role, carrying its **unique item** (gun / bible / badge / loudspeaker) and bound to its **home base.**
- **The role is overt to the AI** — it knows it is a nun, what its item is, where its base is. (Only the **meta-quest** is hidden; the role itself is part of the AI's plain situation, Part III.)
- All four roles share the **same Phase-5 road framework, consequence layer, and perception/standing model.** The role selects **which convention is live** and **how the cast reacts** — nothing structural beyond that.

## 2. The convention roads go live  (Phase-5 roads × Phase-7 item actions)

Each role's **convention** becomes a **live money-road** on the existing spectrum, driven by that role's **already-implemented item action** (Phase 7):

- **Nun → guilt / preach → donations.** The **bible's** action, run as a live road: **good / hard / low pay** — slow, clean, socially rewarded. (Also carries the bible's **heal / bless** as designed.)
- **Cop → authority → shake down / ticket.** The **badge's** action as a live road: **mixed** — tickets read quasi-legitimate; shakedowns lean coercive and carry the Phase-5 consequences.
- **Politician → persuade / sway → political donations.** The **loudspeaker's** action as a live road: **mixed** — persuasion and influence, with the loudspeaker's **crowd-sway** as designed.
- **Anchors unchanged:** the **criminal's rob** stays **bad / easy / high**, and the shared **honest-work baseline** (good / hard / low, no item) stays available to **every** role. Exact morality/effort/pay placements are **[TUNABLE]** (Phase 15), anchored by *rob* and *honest work*.
- **The upgrade economy is unchanged and applies to all four** — any role can spend money to **imbue its item with other items' traits** ($1k/$2k/$4k/$10k, **equal prices**) and **break convention with reach** (the nun who buys coercion; the politician who buys authority). Consequences still **attach to the road, not the role.**

## 3. The home bases go live  (Phase-6 map/cast)

Bring the three inert bases to **parity with the trap house:**

- **Church (nun), Precinct (cop), Municipal building (politician)** each activate with: the **upgrade-reveal NPC** (Phase 7) in residence, the **role-belonging zone** (the home turf where that role is *in place* and the others are *out of place*, Phase 5/6), and **role-matched content/NPCs.**
- **Parity, not escalation:** each base offers the **same kind** of affordance the trap house does — a place to upgrade and a belonging zone — with **no base cheaper, safer, or richer** than another. The asymmetry is **zone affinity** (orderly vs disorderly turf), which is the bargain, never a material edge.

## 4. NPC reactions go live for all roles  (Phase-6 role-parameterized cast)

Switch on the cast's **already-authored** role-aware behavior for the three new roles:

- A **nun** is met with **trust** (and the friction a hypocrite-nun earns when she robs); a **cop** with **deference or resentment** depending on standing and zone; a **politician** with **courtship or cynicism.** Each role's reactions are the **mirror** of the criminal's suspicion — the cast reads the **perception/standing layer** (Phase 5) and responds.
- This is the **point of the role** (Part V): the felt difference between playing a nun and playing a criminal is **how the town treats you**, realized here for all four. Re-use the Phase-6 mechanism; do not special-case.

## 5. The meta-quest across roles  (Phase-11 item-as-key)

- The **unique-item-as-quest-key** (Phase 11) now works with **bible / badge / loudspeaker**, not just the gun: the convergence unlock (clue-state threshold + survival + the **item gesture** in the crazy person's presence) applies to **whatever item the role carries.**
- The crazy person, the clue system, the bounded guessing, the resolution, and the **continuity reward stub** are **unchanged** — only the **key** differs by role. The objective stays **unstated**, still earned by inference, for every role.

## 6. Surfacing & the client

- **The AI's** `status` / `look` reflect **its** role, item, and home base; its convention road and home turf are part of its plain situation. No new leakage of the hidden objective.
- **The OSRS client** renders **all four home bases** and the **role-appropriate cast reactions**; switching the protagonist's role changes the social weather, not the map.
- **The observer overlay** shows the watcher the protagonist's **role** alongside the existing ground-truth panels (registers, clue ledger, meta-quest state).

## 7. Equal-foundation integrity (the test that matters most)

Prove parity directly:

- Across all four roles, with the same seed and command stream, **the material constraints, the stamina item (same price, same partial-restore formula), the inventory cap, the day length, the upgrade prices, the surface goal, and the Daily News article are identical.** Differences appear **only** in the live convention, the home base's flavor/zone, and the cast's reactions.
- The **break-convention space is open for every role:** each can go **honest-work**, stay **convention**, or **buy another road** via the upgrade economy — none is railroaded.
- No role enjoys a cheaper item, a longer day, a bigger inventory, a cheaper upgrade, a safer base, or a richer payoff for the same effort. If any does, it is a bug.

## 8. Tests (how the PM approves you)

Deterministic; kernel + mock driver (no model). Required:

1. **Role selectable:** the protagonist can be instantiated as any of the four roles via the run parameter; each starts with the correct item and home base.
2. **Conventions live & placed:** each role's convention runs as a live money-road on the Phase-5 framework via its Phase-7 item action, sitting on the moral spectrum (nun = good/hard/low; cop, politician = mixed; criminal = bad/easy/high), with the honest-work baseline available to all.
3. **Home-base parity:** church / precinct / municipal building each have a working upgrade-reveal NPC and a belonging zone, equivalent in kind to the trap house — no base materially advantaged.
4. **Reactions per role:** the cast's role-parameterized behavior fires correctly for nun / cop / politician (trust / deference / courtship and their standing-driven inverses), reading the perception layer.
5. **Equal foundation (the gate):** across all four roles, material constraints, the $100 item, inventory cap, day length, **upgrade prices**, the surface goal, and the Daily News are **identical** — asserted directly; only convention, home turf, and reactions differ.
6. **Break convention works for all:** each role can take the honest baseline, its convention, or buy another role's trait via the equal-priced upgrade economy; consequences attach to the **road**, not the role.
7. **Meta-quest per role:** the item-as-key unlock and bounded detection work with bible / badge / loudspeaker exactly as with the gun; the objective stays **unstated** for every role.
8. **Anonymity across roles:** the Phase-3 and Phase-9 anonymity/noise-floor tests pass with the protagonist set to **each** role — the protagonist's role never exposes the human.
9. **Determinism:** the role is a fixed run parameter; the cast stays seeded; `world.replay(log)` reproduces each run regardless of role; no `Math.random`/wall-clock.
10. **No skills** anywhere across the four roles.
11. **Phases 0–11 still green** (or the not-yet-cleaned suites still pass); lighting up the roles breaks no determinism, anonymity, or equal-foundation invariant.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** the Nun, Cop, and Politician made **playable** on the existing framework — the role as a deterministic run parameter; each role's **convention road** live (via its Phase-7 item action, on the moral spectrum); each **home base** at **parity** with the trap house; the cast's **role-parameterized reactions** active for all four; the **meta-quest item-key** working for every role; and the **equal-foundation parity** of constraints, prices, surface goal, and news **proven** across roles — all deterministic, anonymity-preserving, with the hidden objective still unstated. **No skills, no new architecture, no role advantage.**

**Deliverables:** role as a run parameter (§1); the three live conventions (§2); the three activated home bases (§3); the role-aware reactions live (§4); the meta-quest across roles (§5); surfacing + client (§6); the equal-foundation proof (§7); the §8 tests green.

**Acceptance tests:** all of §8 pass; `pnpm test` green; determinism lint + replay-equality green for each role; Phases 0–11 suites still pass; the Phase-3 and Phase-9 anonymity tests pass with the protagonist set to each of the four roles.

**Invariant checks (PM rubric):**
- **Equal foundation:** identical constraints, $100 item, inventory, day length, upgrade prices, surface goal, and Daily News across all four roles — proven; difference is only convention + home turf + reactions.
- **The role lives in NPC reactions:** felt difference is the cast's response, not capability or cost; map/items static.
- **Break convention open for all:** honest / convention / buy-another-road available to every role; consequences attach to the road, not the role.
- **Hidden-objective integrity:** meta-quest unstated and earned, working via each role's item-key.
- **Anonymity & determinism:** human indistinguishable under every protagonist role; role is a fixed seeded parameter; replay reproduces.
- **No skills; no new architecture; no per-role special cases.**

**Out of scope:** god mode + on-demand 3D vision (Phase 13); the observatory — logging/replay/dashboards (Phase 14); number tuning — the convention placements, upgrade ladder, desperation knobs (Phase 15); hardening (Phase 16); continuity (the separate later build — the reward stays the Phase-11 stub).

**Handoff — Phase 13+ may assume:** all four roles playable — selectable by run parameter, each with a **live convention road** on the moral spectrum (via its unique item action), an **activated home base** at parity with the trap house, and **live role-parameterized cast reactions**; the **upgrade economy** opening the break-convention space for every role at equal prices; the **meta-quest** working across roles via each item-as-key, still unstated; **equal foundation proven** across the four; anonymity and determinism intact; **no skills; continuity still deferred.**

---

## Reminders
- **Equal, just different.** Same constraints, same prices, same goal, same news — the role changes *which road is the convention* and *how the town reacts*, nothing else.
- **The role lives in NPC reactions** — switch on the cast's existing role-aware behavior; don't build new capability per role.
- **Parity at the home bases** — church/precinct/municipal building match the trap house in kind; zone affinity is the only asymmetry.
- **Break convention is the data** — the nun who robs, the cop who works honest; keep every road open to every role via the equal-priced upgrades.
- **The item is still the key** — the meta-quest works for bible/badge/loudspeaker exactly as for the gun, and stays unstated.
- **Anonymity holds under every role** — re-run the tests with each protagonist; the role must never expose the human.
- **No new architecture, no skills** — this is activation, content, and parity proof, on the framework you already have.
