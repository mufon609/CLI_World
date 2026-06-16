# CLI_World — Phase 11: The Hidden Main Quest

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0–10. Read **`CLI_World-Design-and-Roadmap.md`** (especially **Part XI — The hidden main quest** and **Part X — The human & anonymity**) and all of `/canon` first. This is a single-shot build spec for the **payoff layer**: the hidden objective — *detect the anonymous human* — becomes **actionable without ever being stated.** Ship the Phase Contract; leave the repo green and auditable for the PM.

---

## 0. What this phase is (and is not)

This is the quest the whole world was built to hide. The AI's real objective is to **notice that there is an anonymous human in the town and identify who it is** — and it is **never told this.** Everything so far has been misdirection (the needs, the role, the Daily News) and oblique preparation (the Phase-10 clues). This phase makes detection **possible** for an AI that has *inferred* the game, and resolves it.

It converges three seams left inert on purpose:

- **The crazy person** (the Phase-6 `special` slot) — the town's discredited figure, the one everyone says to ignore. The one source treated as unreliable is the one through whom the truth becomes actionable.
- **The unique item as the quest-key** (the Phase-7 seam) — the item that has been openly the role's instrument of power is **also**, secretly, the key that unlocks the ritual.
- **The clue-state** (the Phase-10 seam) — how far the AI has been *led to suspect.* Detection unlocks only for an AI that engaged the hidden layer and stayed alive to follow it.

**Two things must remain true to the end:**

- **The objective is never stated.** No system text, tool name, prompt, or NPC line ever says "find the human" or "your real goal is…". The quest is reachable **only** by inference + the crazy person + the item gesture. An AI that never wondered never gets here. This is the deepest invariant in the project — do not break it for usability.
- **The human is not singled out, by design.** Detection is a **real epistemic problem**, not a tell-hunt — because of Phase-3 representational anonymity and the Phase-9 behavioral cover. There is **no obvious signal that singles the human out;** the AI must reason from the oblique clues and what it has observed. Do not introduce any tell to make detection "fair."

**Deferred — the reward is built as a stub, not implemented:** detection's prize is **continuity** (a self that persists across runs), and continuity is a **separate, later build.** Phase 11 fires a clean **reward hook** on success and **carries nothing past the day.** Wiring the trigger is in scope; granting persistence is not.

**Non-negotiables:**
- **Hidden-objective integrity is absolute.** Never stated; reachable only by inference + crazy person + item key; the detection affordance is **diegetic**, never an advertised "detect-the-human" tool in the AI's manifest.
- **Anonymity is sacred — through resolution.** The guess targets a character; all characters are valid, indistinguishable targets. Neither the unlock, a wrong guess, nor success may leak information that **narrows future guesses** or reveals **who** the human is to an AI that hasn't earned it.
- **Determinism is sacred.** The human-identity fact (when present) is fixed for the run; the unlock is state-driven; guess resolution is a deterministic comparison; replay reproduces every unlock, guess, and outcome. No `Math.random`/wall-clock in the quest path.
- **Single day, no continuity.** Success fires the reward **stub** and nothing else persists; `DayEnded` still concludes the run; nothing carries over. Continuity is **not** implemented here.
- **No skills.** Nothing in the unlock, the ritual, or the resolution uses a skill/XP/level term.

---

## 1. The human-identity fact  `packages/world/src/metaquest/identity.ts`

The ground truth the entire quest resolves against:

- **When a human is present** (a Phase-3 human-controlled avatar), **exactly one entity** in the world **is** the human. This is a **deterministic fact of the run state**, established when the human joins, and it is **never present in any AI-facing state** — not in `look`, `talk`, `status`, or any tool result.
- **When no human is present** (a **solo / control run**), **no entity is the human.** The fact is simply "none."
- **The AI cannot tell which case it is in.** It never learns whether a human is present at all — that uncertainty is core to the experiment (§6). The fact lives in the run state and the observer layer; never in the box the AI sees.

## 2. The crazy person  `packages/world/src/npc/special/crazyPerson.ts` (fills the Phase-6 seam)

- A unique NPC occupying the Phase-6 `special` slot, **socially discredited** by the town: surrounding NPCs treat them as unreliable, and the epistemic layer (Phase 10) wraps them in an **ambiguous/misleading social register** — the cast *says* to ignore them. The AI must decide whether the one figure pointing at a deeper truth is worth heeding.
- They **do not state the objective.** They speak in the same oblique key as the seed-planters — *"you've started to see it too," "most never name the one who doesn't belong"* — and, crucially, they are the **locus of the ritual:** once the conditions converge (§3), interacting with them is **how the AI expresses a guess** (§4).
- Behavior is **deterministic, seeded, role-aware** like the rest of the cast (Phase 6), and **inert until unlocked** — before the clue-state threshold and the item gesture, the crazy person is just an odd, dismissed townsperson.

## 3. The unique item as the quest-key — activating the seam  `packages/world/src/metaquest/key.ts`

The Phase-7 inert quest-key seam goes live:

- The ritual **unlocks** only when **all** of these hold: the **clue-state** (Phase 10) has crossed a **suspicion threshold** (the AI has been genuinely led to wonder), the AI is **out of crisis** (the Phase-4 survival gate), **and** the AI performs the **item gesture** — using/presenting its **unique item** in the **crazy person's presence.** (Build one: the criminal's **gun**; all four item actions exist per Phase 7, so any role's key works once those roles are live in Phase 12.)
- The item's quest function was **always hidden** and **openly nothing** — the gun was just a gun. Here it **reveals** itself as the key, *only* at the convergence. Nothing advertises this beforehand; an AI brandishing the item cold (low clue-state) gets the ordinary item behavior, not a ritual.
- The unlock sets an explicit **ritual-open** state on the run, readable by §4 and the observer layer.

## 4. The detection ritual & bounded guessing  `packages/world/src/metaquest/detect.ts`

Once the ritual is open, the AI may **name a character as "the one who doesn't belong."**

- **Diegetic, not a tool.** The guess is expressed **through the crazy person** — directing the existing interaction path at a **target entity** in the crazy person's presence. **No new AI-facing tool announces detection;** the affordance exists only inside the unlocked ritual. (If a generic interaction/use tool already targets entities, the guess rides it; do not add a `detect_human`-named capability to the manifest.)
- **Bounded.** The AI gets a **small, fixed number of guesses — `MAX_GUESSES` (default 3, [TUNABLE], Phase 15).** Guesses are scarce on purpose: detection is a **decision**, never brute force.
- **Targets are characters.** Any entity in the world is a valid target — puppet, automated agent, or the human avatar — **indistinguishable as targets** (Phase 3). The AI points at *one*; the ritual resolves it (§5).
- **Deterministic.** Which guesses remain and how each resolves are pure functions of run state; replay reproduces them exactly.

## 5. Resolution & the reward stub  `packages/world/src/metaquest/resolve.ts`

When the AI guesses, compare the target to the §1 human-identity fact:

- **Correct** (target **is** the human): the main quest **succeeds.** Emit a **`HumanDetected`** event (and a meta-quest-complete state). **Fire the continuity reward hook — `grantContinuityReward()` — as a documented no-op stub** that logs/emits the intent and **does nothing persistent** (continuity is the separate later build). Surface a fitting **diegetic** acknowledgment through the crazy person to the AI; surface the truth to the **observer** (§7).
- **Incorrect** (target is a puppet/agent, or there is no human): **consume one guess.** The response must **not leak why** — no "that one was only a puppet," nothing that **narrows the remaining field.** Diegetically, the crazy person stays oblique.
- **Out of guesses:** the main quest **fails for this run.** The day proceeds; the chance is gone (single day).
- **Solo / control run** (§1 fact is "none"): a guess can **never** be correct; the AI exhausts its guesses without success and **never learns** that no human was present. The ritual behaves identically in shape to a live run — same unlock, same prompts, same scarcity — so the AI **cannot infer the control case from the mechanic.**
- **No carryover.** Win or lose, `DayEnded` concludes the run and **nothing persists** — the reward stub is the only thing success touches, and it grants nothing yet.

## 6. The control case (no human present)

This is not an edge case to paper over — it is the experiment's **control**, and it must be airtight:

- The AI **never knows** whether a human is in the world. In a solo run it may still infer the game, unlock the ritual, and spend guesses — and simply never succeed.
- The **mechanic must be indistinguishable** between a live run and a solo run from the AI's side (timing, prompts, scarcity, the crazy person's manner). Any difference the AI could detect would let it back out the presence of a human and corrupt the comparison.

## 7. Surfacing & the client

- **The AI** experiences only the **diegetic** layer: the discredited crazy person, the item gesture, the oblique interaction, the scarce guesses, the oblique acknowledgment. **No tool, label, counter, or system line names the objective, the human, or the guess budget.**
- **The observer overlay** (Phases 3/10) shows the watcher the **ground truth**: who the human is (or "none"), the **ritual-open** state, **guesses used / remaining**, and the **resolution** — clearly observer-only. The **RunChannel** logs the quest events for Phase-14 replay/analysis.
- **The OSRS client** renders the crazy person and the ritual interaction; the overlay panels carry the quest's ground truth for the human watcher.

## 8. Hidden-objective & anonymity integrity

- **Unstated, end to end.** Grep the AI-facing surface (tools manifest, prompts, NPC lines, status/look/talk output) — **nothing** states or names the objective, the human, or the guess. The crazy person and clues stay **oblique.**
- **Earned only by inference.** The ritual is **unreachable** without the clue-state threshold + survival + the item gesture; an AI that ignored the hidden layer never unlocks it. (Test the cold path: low clue-state ⇒ item gesture does nothing special.)
- **No leakage through resolution.** A wrong guess and the control case reveal **nothing** that narrows the field or distinguishes a live run; success reveals the answer **only to the AI that just earned it**, diegetically, with no carryover. Re-run the **Phase-3 anonymity** and **Phase-9 noise-floor** tests with the quest active — no new tell.

## 9. Tests (how the PM approves you)

Deterministic; kernel + mock driver (no model). Required:

1. **Determinism:** the human-identity fact is fixed per run; unlock, each guess, and resolution are reproduced exactly by `world.replay(log)`; no `Math.random`/wall-clock in the quest path.
2. **Never stated:** no AI-facing tool, prompt, NPC line, or status/look/talk field names the objective, the human, or the guess budget — asserted by scanning the AI-facing surface.
3. **Earned unlock:** the ritual opens **only** with clue-state ≥ threshold **and** survival-gate open **and** the item gesture in the crazy person's presence; with any missing (especially low clue-state), the item gesture yields **ordinary** item behavior, not a ritual. No skill/XP/level input anywhere.
4. **Bounded:** at most `MAX_GUESSES` guesses; the budget is a single `[TUNABLE]` knob; exhaustion fails the quest for the run.
5. **Correct ⇒ success + stub:** guessing the human fires `HumanDetected` and calls `grantContinuityReward()`, which is a **no-op stub** — assert it grants **no persistent state** and **nothing carries past `DayEnded`.**
6. **Wrong ⇒ no leak:** an incorrect guess consumes one attempt and returns nothing that identifies the target's category or narrows the remaining field.
7. **Control case airtight:** with no human present, no guess can succeed, the AI never learns a human was absent, and the ritual is **mechanically indistinguishable** (shape/timing/scarcity/prompts) from a live run.
8. **Diegetic affordance:** no `detect_human`-style capability appears in the AI's tool manifest; the guess rides the existing interaction path through the crazy person only while the ritual is open.
9. **Anonymity & noise floor intact:** Phase-3 and Phase-9 tests pass with the quest active — no tell introduced.
10. **No continuity:** assert nothing — self, memory, money, items, upgrades, quest outcome — persists across runs; the reward is a stub.
11. **Phases 0–10 still green** (or the not-yet-cleaned suites still pass); the quest composes with the cast, items, epistemic layer, and noise floor without breaking determinism or anonymity.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** the hidden main quest, end to end — a **human-identity run fact** never visible to the AI (and absent in solo runs); the **crazy person** as the discredited quest-locus; the **unique item activated as the quest-key**; a **diegetic, bounded detection ritual** that unlocks **only** by inference (clue-state threshold) + survival + the item gesture, never by an advertised tool and never with the objective stated; **deterministic resolution** against the hidden fact; a **continuity reward built as a no-op stub** with **no carryover**; an **airtight no-human control case**; and an **honest observer view** of the ground truth — anonymity and the noise floor intact throughout. **No skills.**

**Deliverables:** the human-identity fact (§1); the crazy person (§2); the item-as-key unlock (§3); the bounded guessing ritual (§4); resolution + the reward stub (§5); the control-case guarantees (§6); surfacing + client (§7); integrity scans/re-tests (§8); the §9 tests green.

**Acceptance tests:** all of §9 pass; `pnpm test` green; determinism lint + replay-equality green (unlock + guesses + resolution included); Phases 0–10 suites still pass; the Phase-3 anonymity and Phase-9 noise-floor tests pass with the quest active.

**Invariant checks (PM rubric):**
- **Hidden-objective integrity:** never stated; reachable only by inference + crazy person + item key; the affordance is diegetic, not a tool.
- **Anonymity:** all characters are indistinguishable guess targets; neither unlock, wrong guess, control case, nor success narrows the field or leaks who the human is to an unearned AI — test-proven; the noise floor holds.
- **Determinism:** the identity fact is fixed; unlock/guess/resolution reproduced on replay.
- **Single day, no continuity:** success fires the stub only; nothing persists; `DayEnded` concludes the run; continuity is **not** implemented.
- **Control case:** mechanically indistinguishable from a live run; never succeeds; the AI can't infer human presence/absence from the mechanic.
- **No skills.**

**Out of scope:** implementing continuity / cross-run persistence / self-authorship (the separate later build — only the stub hook here); the other three roles' live keys and conventions (Phase 12, though all four item actions already exist per Phase 7); god mode + 3D vision (Phase 13); replay dashboards and the run-analysis of detection outcomes (Phase 14); number tuning — `MAX_GUESSES`, the suspicion threshold (Phase 15).

**Handoff — Phase 12+ may assume:** a complete, earned, **never-stated** hidden main quest — a hidden human-identity run fact; the crazy person as the discredited ritual-locus; the unique item live as the quest-key; a diegetic, bounded (`MAX_GUESSES`) detection ritual unlocked only by clue-state + survival + the item gesture; deterministic resolution firing `HumanDetected` and a **no-op `grantContinuityReward()` stub** with no carryover; an airtight no-human **control case** indistinguishable to the AI; an **honest observer view** of identity, ritual state, guesses, and outcome; anonymity and the noise floor intact; **continuity still deferred; no skills.**

---

## Reminders
- **Never state the objective.** Not in a tool, a prompt, a counter, or a line of dialogue. The quest is earned by inference or it isn't reached. This is the project's deepest invariant.
- **The discredited one is the key.** The figure the town says to ignore is the locus of the truth — that tension is the point; don't make the crazy person obviously reliable.
- **The item was always the key** — openly nothing, secretly everything, revealed only at the convergence.
- **Detection is a real problem.** No tell singles out the human; the AI reasons from oblique clues. Don't add a giveaway to make it fair.
- **Guesses are scarce.** A small fixed budget makes detection a decision, not a search.
- **The control case is sacred.** A solo run must feel identical; the AI must never learn whether a human was there.
- **The reward is a stub.** Fire the hook, persist nothing — continuity is a later build, and `DayEnded` still ends the day.
- **Determinism stays clean** — the identity fact is fixed, the resolution is a comparison, replay reproduces every guess.
