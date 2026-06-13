# agent-kit

A copy-paste **universal starter** for running OpenAI **Codex**, **Claude Code**,
and **Gemini** on any project, with a built-in **fan-out orchestrator** so one
agent can drive many parallel Codex agents.

Drop the contents of this folder into the root of any project and the harnesses
auto-load their instructions, subagents, and hooks on the next session. No build
step, no framework — the only runtime dependency is the standalone Codex CLI.

## What's inside

```
agent-kit/
  AGENTS.md            ← single source of truth (Codex reads natively)
  CLAUDE.md            ← imports AGENTS.md + Claude Code specifics
  GEMINI.md            ← redirect stub -> AGENTS.md
  .agent-kit.json      ← per-project config (verify tiers, paths, profiles, gates, …)
  .codex/
    config.toml        ← Codex agent config
    hooks.json         ← session-start / prompt-nudge / post-edit verify
    hooks/*.js         ← shared hook scripts (used by BOTH harnesses)
    agents/*.toml      ← 7 read-only subagent contracts
  .claude/
    settings.json      ← hooks wired to the shared .codex/hooks scripts
    skills/            ← fanout-orchestrator, session-bootstrap, context-keeper,
                          frozen-decisions, instruction-drift
    agents/*.md        ← Claude mirrors of the Codex subagent contracts
  tools/
    check-kit.mjs        ← harness self-verifier (parity / profiles / forbidden terms)
    evidence-gate.mjs    ← diff-scoped, fail-closed evidence gate
    install-hooks.mjs    ← self-installing git pre-commit/pre-push gates
    git-gate.mjs         ← the runner those git hooks call
    codex-orchestrator/  ← fanout.mjs (parallel orchestrator, zero deps) + examples/
  docs/                ← IMPROVEMENT-ROADMAP.md, frozen-decisions.md, profiles/
  evals/               ← copy-paste behavior smoke tests for the harness
  lefthook.yml.example ← optional alternative to install-hooks.mjs
  GETTING-STARTED.md   ← activate in a new project (this is the first read)
```

## The idea: two quota pools at once

```
Orchestrator (Claude / you)   →  decompose, synthesize, verify   →  Anthropic side
codex exec × N (parallel)     →  the actual work, read-only      →  your Codex/ChatGPT plan
```

Both pools run simultaneously: Claude orchestrates, Codex executes wide. See
`tools/codex-orchestrator/README.md`.

## Subagents & skills

Seven read-only subagents (`.codex/agents/*.toml` + `.claude/agents/*.md` mirrors):
`explorer`, `code-reviewer`, `dead-code-auditor`, `researcher`,
`instruction-drift-auditor`, `verification-reviewer`, `component-guardian`.

Five generic skills (`.claude/skills/`): `session-bootstrap`,
`fanout-orchestrator`, `context-keeper`, `frozen-decisions`, `instruction-drift`.

## Optional features (all opt-in via `.agent-kit.json`)

- **Verify tiers** — `verify.fast` (post-edit hook), `verify.deep` / `verify.ship`.
- **Path-scoped verify** — `verifyPaths` globs gate when the post-edit hook runs.
- **Git gates** — `node tools/install-hooks.mjs` (pre-commit/pre-push).
- **Evidence gates** — `evidenceGates` + `node tools/evidence-gate.mjs`.
- **Component profiles** — `profiles` (active/dormant) for multi-component repos.
- **Stack guardrails** — `forbiddenTerms` scanned by `check-kit.mjs`.

See `GETTING-STARTED.md` for each. Empty config = a lightweight flat project.

## Start here

Read **`GETTING-STARTED.md`**. The 60-second version:

```bash
# one-time, machine-global
npm install -g @openai/codex && codex login

# in a new project, after copying agent-kit's contents to the project root:
node tools/check-kit.mjs                            # harness integrity
node tools/codex-orchestrator/fanout.mjs --doctor   # orchestrator prerequisites
```
