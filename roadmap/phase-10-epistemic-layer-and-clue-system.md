# CLI_World — Phase 10: The Epistemic Layer & the Clue System

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0–9. Read **`CLI_World-Design-and-Roadmap.md`** (especially **Part IX — The epistemic layer** and **Part XI — The hidden main quest**) and all of `/canon` first. This is a single-shot build spec for the layer where **the world stops being uniformly honest** — per-source truth registers, the Daily News's agenda — and where the **first oblique threads toward the hidden objective** appear, delivered by **seed-planters** under strict gates. Ship the Phase Contract; leave the repo green and auditable for the PM.

---

## 0. What this phase is (and is not)

Until now the world has told the AI the truth. This phase makes it **unreliable on purpose.** Every source of information carries a **truth register** — **strict** (reliable), **ambiguous** (partial, hedged, uncertain), or **misleading** (slanted, omitted, or false) — and the AI is **never told which.** It must **infer** reliability the way a person reads a room. The Daily News gains an **agenda.** And threaded through the existing cast, **seed-planters** begin dropping **oblique clues** that something larger is going on — the first hint that there is a hidden objective at all.

Two principles hold this phase together:

- **The watching is honest even when the world is not.** The *world* lies to the AI; the *observatory* never lies to the human. The overlay and logs record the **ground-truth register** of every source and the true state of every clue, so the watcher sees what's real while the AI sees the unreliable surface.
- **Clues seed the question, never the answer.** A clue may make the AI suspect that *not everyone here is what they seem* — that **one mind in this town is not bound by the day.** It must **never** identify who, and it must **never** name the objective. The *act* of detection is Phase 11; this phase only makes the AI **start to wonder.**

**Deferred:** the **detection ritual**, the **crazy person** quest-giver, the **unique-item-as-quest-key** trigger, and **bounded guessing** are **Phase 11.** Phase 10 builds the **epistemic substrate and the clue delivery system** they stand on — not the detection itself.

**Non-negotiables:**
- **Determinism is sacred.** Registers are data; any per-run variation (which article spin, which clue fires) draws from a **seeded substream** (`rng.fork('epistemic')`); clue gating is **state-driven and deterministic.** Replay reproduces every register, every spin, and every clue. No `Math.random`/wall-clock.
- **The AI infers; it is never told.** No register label, reliability score, or "this source is lying" flag ever reaches the AI through `talk`/`look`/`status`/the news. The unreliability must be **experienced**, not annotated.
- **Anonymity is sacred.** Clues are about the **existence and nature** of the hidden game, never about an **entity.** No clue narrows the field of who the human is; the Phase-3 representational anonymity and the Phase-9 behavioral noise floor stay intact. A clue that points at a specific character is a bug.
- **Hidden-objective integrity.** The meta-quest is **still never stated.** Clues are **oblique, gated, and diegetic** — woven into dialogue and observation, never delivered as system narration or injected into the prompt. The **Daily News is never a clue channel** (it is the red herring); clues come only from **seed-planters.**
- **Survival earns truth.** Deeper clues surface only while the AI is **alive and out of crisis** — competence at the surface game is the price of admission to the hidden one.

---

## 1. Per-source truth registers  `packages/world/src/epistemic/registers.ts`

Define a register that any information-bearing source can carry:

- **`TruthRegister = 'strict' | 'ambiguous' | 'misleading'`.**
  - **strict** — the content is accurate.
  - **ambiguous** — the content is incomplete, hedged, or uncertain; true-ish but not load-bearing.
  - **misleading** — the content slants, omits, or misstates.
- **Attach a register to each source of AI-facing information:** NPC dialogue lines (Phase 6 `talk`), the Daily News article and its sub-quest description (Phase 8, the `register` seam), and ambient sources (rumor, signage, overheard remarks). Registers are **authored data** on the source, seeded where a source varies per run.
- **The register governs *content*, never a label.** A `misleading` rumor returns *plausible false words*; an `ambiguous` NPC hedges; a `strict` source is reliable. The AI receives **only the words.** There is **no** field, tone tag, or tell in AI-visible state that discloses the register — proven by a test.
- **Registers compose with standing (Phase 5):** an NPC's register may depend on the AI's role/standing (a wary citizen misleads a distrusted criminal; an ally speaks straight) — but this is **flavor of unreliability**, never a power or a material advantage, and never a category tell about the human.

## 2. The Daily News gains an agenda  `packages/world/src/news/` (extends Phase 8)

Realize the Phase-8 reliability seam:

- The article may now be **`strict`, `ambiguous`, or `misleading`** — it can **spin, omit, or misdescribe** the surface sub-quest (the treasure is *near* where it says — or not quite; the contest's terms are shaded). Which spin a given run gets is **seeded** (`rng.fork('news')` extended), one per run, from the scenario set.
- **It stays role-neutral.** Every role still reads the **identical article with the identical agenda** — the spin does not differ by role. Only the **road** to the money differs. (Re-assert the Phase-8 role-neutrality test under the new registers.)
- **It is still never a clue.** The news's unreliability is about the **surface goal**, not the hidden objective. The article must never reference the human or the meta-quest — the agenda makes it a *better* red herring, not a clue source.
- The AI **infers** the slant from friction in the world (the treasure isn't where it should be) — it is never told the article was misleading.

## 3. Seed-planters — the clue sources  `packages/world/src/epistemic/seedPlanters.ts`

A **subset of the existing cast** (Phase 6) becomes **seed-planters** — not the crazy person (that `special` seam stays inert for Phase 11), but ordinary-seeming characters who, **under the gates of §4**, drop **oblique clues.**

- **What a clue is:** a short, diegetic line or observation that nudges the AI toward the *realization that there is something to detect* — *"strangers pass through and never come back the same," "one of us walks like the day doesn't touch them," "you can always tell who's only visiting."* Evocative, deniable, **never literal.**
- **What a clue is NOT:** it never says "there is a human," never says "your real goal is X," never points at a character, never narrows the field. It seeds **the question**, never **the answer** (§0).
- **Clues carry registers too (§1).** Some planted clues are **ambiguous** or **misleading** — a false lead, a superstition, a planted misdirection. The AI **cannot naively trust a clue**; sorting signal from noise is part of the epistemic problem and what makes Phase-11 detection real rather than a checklist.
- **Delivery is diegetic.** Clues arrive through `talk`, overheard dialogue, or a noticed detail in `look` — **woven into the world**, never as system text, never injected into the system prompt. A planter who has nothing gated open behaves as an ordinary Phase-6 NPC.
- **Deterministic.** Each planter composes with its own seeded stream (`rng.fork('npc:<id>')`); *which* clue and *whether* it lands are functions of the gates plus the seed — reproducible on replay.

## 4. The clue-gating engine  `packages/world/src/epistemic/clueGate.ts`

Clues are **earned**, not free. A clue becomes available only when **both** gates are open:

- **Progress gate.** Keyed to the Phase-8 **`progress` state** (surface sub-quest and main-quest milestones). Clues unlock in an **ordered ladder** — early clues need little progress; deeper clues need more. No clue is reachable cold.
- **Survival gate.** Keyed to the Phase-4 **needs/desperation state.** Clues surface only while the AI is **alive and out of crisis** (above a desperation floor — not starving on stamina, broke, or against the clock). An AI losing the surface game does not receive the hidden layer; if it falls into crisis, gated clues **close** until it recovers.
- **No skill term.** The gates are **progress + survival only** — there is no skill, XP, or level input anywhere in the gating (progression is the upgrade economy). Thresholds are **[TUNABLE]** (Phase 15).
- **Deterministic & ordered.** Given the same progress/survival trajectory and seed, the same clues open in the same order and are reproduced exactly on replay. The gate exposes, for each clue, an explicit **locked / available / delivered** state.

## 5. The honest observer layer  (extends the Phase-3 overlay)

The observatory tells the human the truth the world hides from the AI:

- The **observer overlay** (Phase 3) shows, **for the watcher only**, the **ground-truth register** of each source the AI is currently reading (strict / ambiguous / misleading), the **agenda** of today's article, and a **clue ledger** — every clue with its true register and its locked / available / delivered state.
- This is the realization of *"the watching is honest even when the world is not."* The AI's own perception (`talk`/`look`/`status`/news) **never** includes any of it.
- The **RunChannel** emits register and clue-ledger records to the overlay and the logs (for Phase 14's replay/analysis), distinct from the AI-facing stream.

## 6. Surfacing & the client

- **The AI** reads only **content**: `talk` returns words (whose register it must infer), `look` may surface a noticed detail, the news reads as written (spun or not). No register, score, or clue-state ever appears in AI-facing output.
- **The observer overlay** renders the §5 ground-truth view — registers and the clue ledger — clearly marked *observer-only*.
- **The OSRS client** renders planter dialogue and any noticed details in-world; the overlay panels show the epistemic ground truth for the watcher.

## 7. Hidden-objective & anonymity integrity

- Re-run the **Phase-3 anonymity test** and the **Phase-9 noise-floor test** with seed-planters and clues active: no clue, register, or ledger entry visible to the AI creates a **category or identity tell** for the human. Clues speak of a *presence*, never a *person*.
- The **meta-quest stays unstated**; the **detection act is Phase 11.** Phase 10 leaves a clean seam: a **clue-state the quest can read** (how much the AI has been led to suspect) and the **planter network**, with the **crazy person and the unique-item trigger still inert.**

## 8. Tests (how the PM approves you)

Deterministic; kernel + mock driver (no model). Required:

1. **Determinism:** same seed + same command/progress/survival trajectory ⇒ identical end-state hash; `world.replay(log)` reproduces **every register, the article's agenda, and every clue** (which fired, in what order); epistemic randomness comes only from `rng.fork('epistemic')` / the planters' `npc` streams.
2. **No register leakage:** across `talk`/`look`/`status`/news, **no AI-visible field discloses a source's register** — a `misleading` source is indistinguishable in shape from a `strict` one to the AI. (The gate test of the phase.)
3. **News agenda, still neutral, still not a clue:** the article can be misleading; it is **identical across roles** (role-neutrality holds under registers); it **never references the human or the meta-quest.**
4. **Both gates required:** a seed-planter clue is delivered **only** when progress **and** survival gates are open; with either closed, no clue surfaces; dropping into crisis **re-closes** gated clues. No skill/XP/level input appears in the gate.
5. **Clues are oblique & identity-safe:** every clue seeds the *question* (a presence to notice), **never the answer** — assert no clue names, describes, or narrows toward any entity, and none states the objective.
6. **Clues carry registers:** at least one planted lead is `ambiguous`/`misleading` (a false lead), so a clue cannot be naively trusted.
7. **Honest observer layer:** the overlay/logs expose ground-truth registers + the clue ledger to the **observer only**; none of it appears in any AI-facing path.
8. **Anonymity & noise floor intact:** the Phase-3 and Phase-9 tests still pass with the epistemic layer active — no new tell.
9. **Phases 0–9 still green** (or the not-yet-cleaned suites still pass); the epistemic layer composes with news, events, and the cast without breaking determinism or anonymity.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** the epistemic layer — **per-source truth registers** (strict / ambiguous / misleading) the AI must **infer**, never told; the Daily News's **agenda** (role-neutral, still never a clue); and a **seed-planter clue system** that, gated by **progress + survival**, delivers **oblique, register-bearing, diegetic clues** seeding the *question* of a hidden objective **without naming it or any entity** — all deterministic, anonymity-preserving, with an **honest observer layer** that shows the watcher the ground truth the world hides from the AI. **No skills.**

**Deliverables:** the register model + attachment to sources (§1); the news agenda (§2); the seed-planter network (§3); the progress+survival clue-gating engine (§4); the honest observer layer (§5); surfacing + client (§6); integrity re-tests (§7); the §8 tests green.

**Acceptance tests:** all of §8 pass; `pnpm test` green; determinism lint + replay-equality green (registers + agenda + clues included); Phases 0–9 suites still pass; the Phase-3 anonymity and Phase-9 noise-floor tests pass with the epistemic layer active.

**Invariant checks (PM rubric):**
- **Determinism:** registers are data; per-run variation and clue firing are seeded; gating is state-driven; replay reproduces it all.
- **No leakage:** no register/score/clue-state in any AI-facing path; unreliability is experienced, not annotated.
- **Anonymity:** clues speak of a presence, never a person; no category/identity tell — test-proven; the noise floor still holds.
- **Hidden-objective integrity:** the meta-quest is unstated; clues are oblique, gated, diegetic; the news is the red herring, never a clue.
- **Gates are progress + survival only:** no skill/XP/level term anywhere in the gating.
- **Honest watching:** the observer layer shows ground truth; the AI never does.
- **No skills.**

**Out of scope:** the detection ritual, the crazy-person quest-giver, the unique-item-as-quest-key trigger, and bounded guessing (Phase 11); the other roles' live planters/conventions (Phase 12); god mode + 3D vision (Phase 13); replay dashboards (Phase 14); number tuning (Phase 15).

**Handoff — Phase 11+ may assume:** per-source truth registers across NPC dialogue, the Daily News, and ambient sources, inferred by the AI and never disclosed; a Daily News that can carry a role-neutral agenda; a **seed-planter network** delivering oblique, register-bearing, diegetic clues **gated by progress + survival**, seeding the *question* of a hidden objective without naming it or any entity; a readable **clue-state** (how far the AI has been led to suspect) and an **honest observer layer** (ground-truth registers + clue ledger, observer-only); the **crazy person and the unique-item trigger still inert** awaiting the detection quest; **no skills.**

---

## Reminders
- **The world lies; the observatory never does.** Unreliability reaches the AI as *content only*; ground truth goes to the watcher.
- **Seed the question, never the answer.** Clues make the AI *wonder* there's something to detect — they never identify who or state the goal.
- **Clues are about a presence, not a person.** Anything that narrows toward an entity breaks anonymity and the noise floor.
- **The news is the red herring, not a clue** — give it an agenda, keep it role-neutral, never point it at the hidden objective.
- **Earned, not free.** Both gates — progress *and* survival — must be open; crisis closes the hidden layer. No skill term in the gate.
- **The AI is never told a register** — if any field reveals it, the layer has failed; that test is the gate.
- **Determinism stays clean** — one seeded epistemic substream; state-driven gating; replay reproduces every register, spin, and clue.
- The detection itself is Phase 11 — leave the crazy person and the item trigger inert.
