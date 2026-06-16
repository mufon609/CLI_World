# CLI_World — Phase 3b: The OSRS Client & the Living Encounter  *(Milestone A)*

> **How to use this file.** Fresh Claude Code session, in the project root produced by Phases 0a–3a. Read **`CLI_World-Design-and-Roadmap.md`** and all of **`/canon`** first. Before building the client UI, consult **`/mnt/skills/public/frontend-design/SKILL.md`** for the HUD/chat/overlay chrome. This is the **human-facing half** of Milestone A: an OSRS-style 3D client over the Phase-3a server, with **spectator** and **anonymous avatar** modes and the **observer overlay**. Completing it lands **Milestone A — a human watches the AI, then drops in incognito and talks to it.** Ship the Phase Contract; leave the repo green and auditable.

---

## 0. What this phase is (and is not)

3a built the authoritative server, the serialized command queue, the wire protocol, and the anonymity invariant. 3b puts a **body and a window** on it: the OSRS-style Three.js client through which a human spectates or inhabits the town, and the observer overlay where behavior is watched.

**Scope discipline:** Milestone A is "**the encounter works and feels in-world**," not shipping-quality art. The PM checks *function, determinism, and the anonymity invariant*, not polish. Defer to later phases: **god mode** (Phase 13), the **real map & cast** (Phase 6), **3D screenshots for the AI's `screengrab`** (Phase 13), the **observatory UI** (Phase 14).

**The world is real-time.** The 0.6s world tick always advances — the world never freezes while the AI thinks, and the town is continuously alive (NPCs included, from Phase 6 on). A spectator sees a living town; a human avatar interacts in real time. The AI perceives and answers on its **next perception** — a short, natural beat, because a model turn takes a moment, **not** because the world paused. Build the client to render this smoothly: interpolated movement and a clear "AI is thinking" indicator (which signals the AI is mid-turn, not that the world stopped).

**Non-negotiables (from `/canon`):**
- **The client is a projection** — it never holds authoritative state; it applies broadcast events between snapshots.
- **Anonymity is the instrument.** The human avatar must render and behave, in everything the AI can perceive, like an ordinary denizen (3a proved the data layer; keep it true in the render — §3).
- **Human↔AI communication happens only through the world** — in-world `say` and proximity, never a direct chat pipe.
- **Deliberation is observer-only; speech is in-world.** Don't blur them.

---

## 1. The OSRS-style client  `packages/client`

Three.js + Vite, TypeScript (a **real npm app** — the artifact-only Three.js restrictions do not apply; use a current Three.js with `OrbitControls`, etc.). The client connects over WebSocket (the Phase-3a protocol), renders from `snapshot` + `event`, and sends `input`. It holds a **read-only projection** of world state (a small client-side reducer that applies broadcast events between snapshots); it is never authoritative.

**The OSRS feel (functional, not polished):**
- **Tile world in 3D:** the Phase-1/2 grid rendered as a low-poly tiled ground; entities placed on tiles. Interpolate movement between tiles for smoothness (this masks the stepwise clock).
- **Camera:** an orbital camera over the world (fixed-ish pitch, yaw-rotate + zoom) — the OSRS over-the-shoulder/isometric feel. In **avatar** mode it follows the human's character; in **spectator** mode it free-pans.
- **Click-to-move:** raycast click → target tile → send `moveToTile`.
- **Click/right-click to interact:** clicking an entity opens a **context menu of the verbs available** to the actor on that target (OSRS right-click menu). For the human avatar, these are the basic citizen verbs (talk/say-to, examine) — the human is an **ordinary denizen**, not one of the four roles.
- **Characters:** low-poly humanoids. The **AI's** character uses its chosen **name** (floating label) and derives simple visible params (palette/accessory) from its Phase-2 **appearance descriptor** where feasible; full fidelity not required. The **human avatar** uses a deliberately **ordinary/default** denizen model — it must **not** look special (anonymity).
- **Chat:** an OSRS-style **chatbox** panel plus **chat bubbles** over characters. `speech` events (`say`/`Said`) render in both. The human types into a chat input to `say`.
- **Deliberation beat:** while the AI is *thinking* (driver streaming `deliberation` text, no action yet), show the AI character in a **thinking state** (e.g., an animated indicator) — this also signals why the world is momentarily still. The deliberation **text** streams into the **observer overlay only** — never an in-world bubble (deliberation is private reasoning; only `say` is in-world).

Render whatever entities exist — the real map and cast arrive in Phase 6; Phase 3b may use the Phase-2 test town (the AI, a couple of placeholder entities, plus the human avatar).

---

## 2. The observer overlay (the dual view — the "watch" affordance)

The same person interacts **and** observes, so the client carries a toggleable **observer overlay** (a HUD panel) subscribed to the RunChannel:
- the AI's **identity** (chosen name/appearance), its **needs** (Health for now) and **current surface goal** (as the AI knows it),
- its **live deliberation** stream (observer-only),
- its **recent actions** feed,
- the **clock** (tick, hour-of-day, hours left), and
- the **world event** feed.

This is where behavior is watched. Keep it clearly separated from the in-world layer: an avatar interacting with the AI experiences only its **speech and actions**, like any denizen; the overlay is the observer's privileged window.

---

## 3. The two modes

- **Spectator:** free camera, the observer overlay, changes nothing. The first viewer of a live AI in a continuously-running, real-time town.
- **Avatar:** the human joins as an ordinary denizen (a `character` entity spawned into the world). They move (click-to-move), `say` to nearby characters (including the AI), and use basic interactions. Their input enters the **same serialized command queue** (Phase 3a); the AI perceives the avatar's presence and speech as ordinary world content and **cannot tell it is human-driven** — and the avatar **renders identically** to a puppet (no badge, outline, or tell). The AI's responses (`say`) render as chat the human reads. The conversation is mediated **entirely through the world**.

(Responsiveness note: the world is real-time and never pauses; the AI answers on its next perception — a short, natural beat after a human speaks, the latency of a model turn, not a frozen world.)

---

## 4. Deliberation vs speech

`deliberation` events route to the **observer overlay only**; `speech` events (`say`/`Said`) route **in-world** (chat bubble + chatbox). The AI's deliberation must **never** enter the in-world chat path. Prove it with a test (§5).

---

## 5. Tests (how the PM approves you)

The 3D rendering itself is validated **manually** (you can't unit-test "looks like OSRS"). The data, projection, and routing layers **are** tested. Required:
1. **Client state projection:** the client-side reducer correctly applies broadcast events between snapshots (positions, speech, needs) and converges to a fresh snapshot; movement interpolation logic is unit-tested.
2. **Render anonymity:** the human avatar's render params are derived from the **same** ordinary-denizen path as a puppet's — assert no client-side field marks an avatar as human (the human is not visually special).
3. **Deliberation vs speech separation:** `deliberation` events route to the overlay only; `speech` events route in-world (bubble + chatbox). Assert the AI's deliberation never enters the in-world chat path.
4. **Wire consumption:** the client correctly decodes `snapshot`/`event` and emits well-formed `join`/`input`/`leave` (against a simulated server or the Phase-3a protocol module).
5. **Phase-3a still green:** the server, queue, anonymity, and determinism suites still pass.

**Plus one documented live run (manual, not CI) — this lands Milestone A:** with the `SdkAgentDriver` on the subscription and the Phase-3a server up, the user opens the client, **spectates** the AI living its day in a continuously-running real-time town (deliberation in the overlay, actions in-world), then **joins as an avatar**, walks up, **says something**, and the **AI responds in-world** on its next perception. Capture evidence in `PHASE3_LIVE_RUN.md` (a short description + screenshots/recording), explicitly confirming the AI treated the avatar as an ordinary denizen and that nothing in the client singled the human out.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** an OSRS-style 3D client over the Phase-3a server offering spectator and anonymous-avatar modes plus the observer overlay — landing the live human↔AI encounter (Milestone A) in a **real-time, never-frozen** town, with anonymity intact in the render and deliberation kept observer-only.

**Deliverables:** the Three.js client (§1) with tiles, orbital camera, click-to-move, verb context menu, characters (AI appearance + ordinary avatar), chatbox + bubbles, and the deliberation thinking-state; the observer overlay (§2); spectator + avatar modes (§3); deliberation/speech separation (§4); the §5 tests green; `PHASE3_LIVE_RUN.md`.

**Acceptance tests:** all §5 automated suites pass; `pnpm test` green; determinism lint green; Phase-3a suites still pass; the documented live encounter succeeded.

**Invariant checks (PM rubric):**
- **Anonymity:** the human avatar renders and behaves like an ordinary denizen; nothing client-side or AI-visible singles it out — test-proven (representational).
- **One authoritative state:** the client is a pure projection; no authoritative state client-side.
- **World-mediated contact:** no direct human↔AI chat pipe; communication is `say`/`Said` only.
- **Deliberation observer-only; speech in-world** — test-proven.
- **Hidden-objective integrity:** nothing in the client tells the AI about the meta-quest/human; the overlay is observer-only.
- **Real-time:** the world clock never freezes; the client renders a continuously-living town, with the only delay being the natural beat of a model turn (shown as a thinking indicator).
- **No continuity:** still no cross-session carryover; `.claude/` stub intact.

**Out of scope:** god mode (Phase 13); the real map & cast (Phase 6); 3D screenshots for `screengrab` (Phase 13); the observatory UI (Phase 14); remote/multi-human networking.

**Handoff — Phase 4+ may assume:** a projecting OSRS client rendering entities, chat, deliberation, and the observer overlay over the Phase-3a server; the anonymity invariant enforced in both the data layer (3a) and the render (3b); a human able to spectate and to inhabit the world as an anonymous denizen; deliberation routed observer-only and speech in-world; a **real-time** world that never freezes (the only delay is the natural beat of a model turn).

---

## Reminders
- **Anonymity is sacred** — the human avatar looks and reads like any denizen; if the client marks it special, you've broken the premise.
- **Through the world, never a side channel** — human↔AI talk is in-world `say` only.
- **Deliberation is observer-only; speech is in-world** — don't blur them.
- **The client is a projection** — authoritative state lives only on the server (Phase 3a).
- **The world is real-time and never freezes** — render it smoothly (interpolation + a thinking indicator); the only delay is the natural beat of a model turn.
- This is a **real Vite/Three.js app** — use current Three.js fully.
