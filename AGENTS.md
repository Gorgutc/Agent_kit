# Agent Rules

This file is the **single source of truth** for every agent harness in this
project: OpenAI **Codex** (reads `AGENTS.md` natively), **Claude Code** (imports
it through `CLAUDE.md` via `@AGENTS.md`), and **Gemini** (`GEMINI.md` redirect
stub). Additional harnesses get a **redirect stub only** — canonical skills are
never mirrored per harness.

> This file ships from a reusable **agent-kit** starter. Fill in the
> `PROJECT SPECIFICS` section per project; the rest is harness scaffolding that
> works the same everywhere.

## Working rules

- Read before editing. Prefer ripgrep (`rg`) for search.
- Keep edits scoped to the requested task; preserve unrelated user changes.
- Before deleting or overwriting something you did not create, inspect it and
  surface any contradiction instead of proceeding.
- Report outcomes faithfully: if a check fails, say so with the output.
- State intent when behavior changes; verify after shipped-code edits.

## Orchestration — fan out to Codex agents

This project ships a dependency-free **fan-out orchestrator** at
`tools/codex-orchestrator/fanout.mjs`. It runs many `codex exec` turns in
parallel. Use it (or per-harness subagents) for substantial work: broad audits,
dead-code / duplication search, multi-file reviews, codebase-wide research.

How to use it:

1. Decompose the task into independent **units** (one prompt each).
2. Write a `spec.json` (see `tools/codex-orchestrator/README.md`).
3. Check prerequisites once: `node tools/codex-orchestrator/fanout.mjs --doctor`.
4. Run: `node tools/codex-orchestrator/fanout.mjs --spec spec.json`.
5. Read `runs/<runId>/summary.md`, then **synthesize and verify** — the
   orchestration is not done until findings are checked.

### Subagent prompt contract

Every delegated unit/subagent prompt must stand alone and specify these 7 fields:

1. **Documentation** — what to read first (AGENTS.md + the relevant files).
2. **Selected skills** — which skills apply.
3. **Selected agents** — which subagent role this is.
4. **Write zone** — exactly which files it may touch (read-only by default).
5. **Verification** — how its result will be checked.
6. **Stop rules** — when to stop / escalate instead of guessing.
7. **Expected output** — the precise shape of the return (findings list, verdict, …).

### Decomposition / routing matrix

For multi-stream work, decide each stream explicitly:

| Stream | Goal | Agent | Write zone | Mode (parallel / sequential / local) | Reason |
| --- | --- | --- | --- | --- | --- |
| … | … | … | … | … | … |

- **parallel** only when write zones are disjoint (or all read-only).
- **sequential** when a stream depends on another's output.
- **local** when no spawn tool is available — and say so (see degradation rule).

### Result contract

Each unit returns **PASS / FAIL + evidence + explicit defers**. A *defer*
(something intentionally left for later) must never silently override a
**blocker** (something that makes the result unsafe to ship).

### Degradation rule

If no subagent-spawn tool is available in the current environment, **do the work
locally and document the fallback** ("spawn unavailable; executed locally")
rather than treating the absence as a blocker.

### Harness rules

- **Default to `sandbox: "read-only"`.** Read-only units fan out wide safely — no
  working-tree conflicts.
- **Parallel writes need isolation.** Only use `sandbox: "workspace-write"` when
  each unit has its **own git worktree** (per-unit `cwd`); otherwise parallel
  writers clobber each other. Merge the worktrees afterward.
- **Concurrency is capped by your Codex plan's rate limits**, not the script.
  Start at `concurrency` 4–6 and raise empirically.

Single-shot alternative: the Codex CLI's own `codex exec "<prompt>"`, or (if
installed) the `codex@openai-codex` Claude Code plugin for a single review/rescue.

## Skills

Generic, stack-agnostic skills in `.claude/skills/` (Codex reads their guidance
from this file):

- `session-bootstrap` — start-of-task: read instructions, check git, classify
  narrow vs broad, pick skills/agents.
- `fanout-orchestrator` — when/how to fan work out to parallel Codex agents.
- `context-keeper` — carry the few load-bearing facts across a long task.
- `frozen-decisions` — respect the frozen list; change the doc and its verifier
  together (`docs/frozen-decisions.md`).
- `instruction-drift` — audit the harness itself for staleness/contradictions.

## Subagents

Reusable read-only agent contracts in `.codex/agents/*.toml` with Claude mirrors
in `.claude/agents/*.md`:

- `explorer` — broad read-only code/file search; conclusions, not dumps.
- `code-reviewer` — reviews a diff/branch for correctness bugs.
- `dead-code-auditor` — finds unused/duplicated/unsafe-to-remove code.
- `researcher` — answers a question from the codebase, read-only.
- `instruction-drift-auditor` — audits the harness for drift.
- `verification-reviewer` — runs the verify command, reports PASS/FAIL.
- `component-guardian` — profile-aware guard (see Component profiles).

## Component profiles (optional)

For multi-component repos (e.g. backend + addon + desktop), declare each
component as a **profile** in `.agent-kit.json` instead of one flat instruction
set:

```json
"profiles": [
  { "name": "backend",     "status": "active",  "doc": "docs/profiles/backend.md" },
  { "name": "windows-exe", "status": "dormant", "doc": "docs/profiles/windows-exe.md" }
]
```

- **dormant** = the rules exist, but NO toolchain / dependency / build command for
  that component may be introduced until an explicit request flips status to
  `active`. The `component-guardian` subagent enforces this.
- Empty `profiles` (the default) = a single flat project. Templates live in
  `docs/profiles/_TEMPLATE.md`.

## Commands

Set verification commands in `.agent-kit.json` (`verify.fast` runs in the
post-edit hook; `verify.deep` / `verify.ship` are for the orchestrator and
pre-PR/CI; `verifyCommand` is a back-compat alias). Harness tooling:

```bash
node tools/check-kit.mjs                              # harness integrity / parity
node tools/codex-orchestrator/fanout.mjs --doctor     # orchestrator prerequisites
node tools/evidence-gate.mjs                          # fail-closed evidence gate (if configured)
node tools/install-hooks.mjs                          # install git pre-commit/pre-push gates
# project examples — set these in .agent-kit.json
npm test
npm run build
```

## Done when

- The requested files are changed and unrelated edits are preserved.
- The project's verify command(s) pass after code changes.
- `node tools/check-kit.mjs` passes if harness files changed.
- New instructions stay short and tied to the real code.

---

## PROJECT SPECIFICS

<!-- Fill this in per project. Delete the guidance lines once filled. -->

- **Project:** TODO — one line on what this project is.
- **Stack / runtime:** TODO — language, framework, constraints.
- **Verification:** TODO — the exact command(s) that must pass. Mirror them in
  `.agent-kit.json` (`verify.fast` / `verify.deep`).
- **Do-not-touch:** TODO — frozen files, generated regions, secrets. Record
  durable decisions in `docs/frozen-decisions.md`.
- **Branch / publish flow:** TODO — branch naming, PR policy.
