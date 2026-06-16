# CLI_World — Phase 0a: The Auth Gate (go / no-go)

> **How to use this file.** Open a fresh Claude Code session in an **empty project directory** and give it this entire document as the task. This is the project's **first** step and a **hard go/no-go**: it builds nothing but a throwaway validation harness. **`CLI_World-Design-and-Roadmap.md`** is already present in the root; read it first. If this phase fails, the project does not proceed until it is resolved — so it is isolated from the monorepo build (Phase 0b) on purpose.

---

## 0. Why this is its own phase

The entire project assumes one thing is true on this machine: the **TypeScript Claude Agent SDK can be driven on subscription OAuth (via the local Claude Code session) and can actually *execute* an in-process MCP tool**, sustained over a long, many-turn day. Everything downstream is built on that seam. If it doesn't hold, no amount of game code matters. So we prove it **first, in isolation, with nothing else at stake**, and we record exactly what we learned for every later phase to rely on.

**Prime directive:** validate the runtime path end-to-end. **If it fails, STOP and report — build nothing else.** A false "it works" is worse than a clean failure.

---

## Deliverable A — CRITICAL AUTH GATE (the whole phase)

The agent runtime is the TS Agent SDK (`@anthropic-ai/claude-agent-sdk`), which wraps the Claude Code CLI; on the OAuth path, calls bill against the **subscription**. There are historically known rough edges in in-process MCP (tool *discovery* vs *execution* gaps; a concurrent-call "Stream closed" race), so validate actual **execution**, single-call, with versions recorded.

1. Ensure Node.js LTS and the `claude` CLI are installed. Authenticate Claude Code via **subscription** (`claude` interactive login, or `claude setup-token` for a long-lived OAuth token). Record `claude --version` and the installed `@anthropic-ai/claude-agent-sdk` version.
2. **Guard against API-key shadowing.** If `ANTHROPIC_API_KEY` is set in the environment, the OAuth token is shadowed and calls silently bill against the API. The validation script must **assert it is unset** and exit with a clear message if it is.
3. Create a **quarantined, throwaway** validation harness under `tools/auth-check/` (TypeScript). It must:
   - define one in-process tool with `createSdkMcpServer` + `tool()` (Zod schema), e.g. `ping` returning `"pong"` plus a monotonic integer;
   - run the agent (`query` or the stateful client) with that server attached, the tool **allow-listed**, built-in tools disabled, and permissions pre-approved so there is no interactive prompt;
   - drive **one** turn that requires calling `ping`, and **assert from the streamed messages that the tool was actually executed** (a `tool_use` for `mcp__*__ping` followed by a real result), not merely named.
   - This is disposable scaffolding — it must **not** become the real `mcp` package (designed in Phase 2). Keep it isolated; Phase 0b explicitly does not reuse it.
4. Confirm authentication used the **subscription**, with `ANTHROPIC_API_KEY` unset. If it only works with an API key set, that is a **finding, not a pass**.
5. **Verify the isolation knobs the hidden objective depends on.** In the same throwaway harness, confirm the SDK session can (a) **fully replace** the default system prompt with a custom string, (b) be restricted to **only** the in-process MCP tools (no built-in file/bash/edit tools), and (c) be configured so it does **not** auto-load the surrounding repo's `CLAUDE.md` / project settings (e.g. an empty `settingSources`). This last point is load-bearing: this very repo's `CLAUDE.md` describes the hidden objective, and the protagonist must never see it. Record the exact options that achieve all three.

## Deliverable B — Usage/rate observation (a brief note)

Subscription usage is **windowed** (a rolling limit / a monthly Agent-SDK credit pool). From the same harness, run a short multi-turn loop (e.g. 10–20 trivial `ping` turns) and record the **approximate token cost per turn** (each turn re-sends the growing transcript) and any rate/throttle behavior you observe. This is **reconnaissance only**, informing later tuning — not a constraint to design around now. (At a 0.6s real-time tick a full in-game day is on the order of minutes of wall-clock, so a day's turn count is bounded by how often the agent acts, not an open-ended marathon.)

---

## Stop-on-fail

If any step fails — OAuth won't authenticate the SDK, in-process MCP tools are discovered but not executable, permissions can't be pre-approved, the streaming session won't initialize, the isolation knobs don't suppress `CLAUDE.md`, or the rate window is so tight no usable day fits — **STOP**. Write everything to `PHASE0_AUTH_FINDINGS.md` (exact commands, versions, exact errors, the per-turn cost estimate, your read on cause and options) and report to the user. **Do not proceed to Phase 0b.**

On success, write `PHASE0_AUTH_FINDINGS.md` confirming the pass with captured evidence (the message trace showing real tool execution, the versions, the unset-key assertion, the working isolation options, and the turns-per-window estimate). Initialize **git** and commit the throwaway harness + findings (conventional commits). Then hand off to **Phase 0b**.

---

## Phase Contract (satisfy this; leave it for the PM)

**Objective:** *prove* the subscription-OAuth + in-process-MCP-**execution** runtime — with prompt/tool/`settingSources` isolation and an honest turns-per-window budget — before anything is built on it.

**Acceptance tests / criteria:**
- [ ] **Auth gate passed** with captured evidence in `PHASE0_AUTH_FINDINGS.md`: TS Agent SDK **executing** an in-process MCP tool under **subscription OAuth**, `ANTHROPIC_API_KEY` unset, versions recorded. *(Or: gate failed, findings written, work halted — also a valid terminal state to report.)*
- [ ] The three isolation knobs verified (custom system prompt replaces default; MCP-only tools; `CLAUDE.md`/project config **not** auto-loaded), with the exact options recorded.
- [ ] Per-turn token cost and any rate/throttle behavior recorded (brief usage note).
- [ ] `tools/auth-check/` is quarantined and committed; git initialized.

**Invariant checks:** subscription-not-API guard present and exercised; the isolation that keeps the hidden objective from the agent is demonstrated, not assumed.

**Out of scope:** the monorepo, the canon, the PM tooling, the determinism harness — **all of that is Phase 0b** and must not be started until this gate passes.

**Handoff — Phase 0b may assume:** a *validated* agent-runtime path (subscription OAuth, in-process MCP execution, prompt/tool/`settingSources` isolation) with the exact working SDK version + options recorded in `PHASE0_AUTH_FINDINGS.md`, and a brief usage/rate note for later sizing.

---

## Constraints & reminders
- **Subscription, not API.** If it only works with `ANTHROPIC_API_KEY` set, that's a finding, not a pass.
- **Execution, not discovery.** Assert the tool actually ran from the streamed messages.
- **Isolation is sacred.** Prove the agent can't auto-load this repo's `CLAUDE.md` — the hidden objective lives there.
- **Don't build the game.** This phase is a throwaway probe and a written verdict. Nothing more.
- If the prompt or the design contains a genuine conflict, **stop and surface it** rather than guessing.
