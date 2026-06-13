# Codex fan-out orchestrator

A small, dependency-free harness that lets an agent (Claude Code, or you) act as
an **orchestrator** and fan a job out to **many parallel `codex exec` turns**.
Orchestration runs on the Claude/Anthropic side; the Codex turns run on your
Codex/ChatGPT plan. Two independent quota pools, used at the same time.

```
Orchestrator (Claude / you)
        │  decompose -> spec.json
        ▼
fanout.mjs  ──spawn──►  codex exec  (unit 1)  ┐
            ──spawn──►  codex exec  (unit 2)  │  your Codex/ChatGPT plan
            ──spawn──►  codex exec  (unit 3)  │  (read-only by default)
            ──spawn──►  codex exec  (unit N)  ┘
        │  collect finals + logs
        ▼
   summary.json / summary.md  ──►  orchestrator synthesizes + verifies
```

`fanout.mjs` has **zero dependencies** (Node built-ins only) and does **not**
need any Claude Code / Codex plugin — only the standalone Codex CLI. Copy the
single file anywhere.

## Quick start

```bash
# 0. One-time, machine-global:
npm install -g @openai/codex
codex login

# 1. Check prerequisites (no quota spent)
node tools/codex-orchestrator/fanout.mjs --doctor

# 2. Run the bundled read-only audit fan-out (4 parallel Codex agents)
node tools/codex-orchestrator/fanout.mjs --spec tools/codex-orchestrator/examples/audit.spec.json
```

Results land in `tools/codex-orchestrator/runs/<runId>/` (or use `--out <dir>`):

| File             | Contents                                  |
| ---------------- | ----------------------------------------- |
| `<id>.final.md`  | the agent's final message for that unit   |
| `<id>.log`       | full stdout+stderr stream for that unit   |
| `summary.json`   | machine-readable results (status, timing) |
| `summary.md`     | human digest with previews                |

Exit code is `0` only if every unit exited `0`.

## CLI

| Flag | Meaning |
| --- | --- |
| `--doctor` | Check Node + Codex CLI + login. No quota. (alias: `--check`) |
| `--spec <file>` | Run the fan-out described by the spec |
| `--out <dir>` | Write run artifacts under `<dir>` instead of next to the script (useful for a shared/global copy) |
| `--run-id <id>` | Name the run directory (default: timestamp) |

## Spec format

```jsonc
{
  "concurrency": 4,           // max parallel codex processes (default 4)
  "sandbox": "read-only",     // read-only | workspace-write | danger-full-access
  "model": "gpt-5.3-codex",   // optional; omit for the Codex default
  "effort": "medium",         // optional reasoning effort
  "cwd": "<abs project path>",// working root for every unit (default: process.cwd())
  "timeoutMs": 900000,        // per-unit hard timeout (default 15 min)
  "skipGitCheck": false,      // true for non-git directories
  "units": [
    { "id": "audit-deps", "prompt": "..." },
    { "id": "review-api", "prompt": "...", "model": "...", "effort": "high", "cwd": "..." }
  ]
}
```

Per-unit `model` / `effort` / `cwd` / `sandbox` / `timeoutMs` override the
top-level defaults for that unit only. `id` must match `[A-Za-z0-9._-]` and be
unique; `prompt` is fed to `codex exec` over **stdin** (length/quoting never a
problem).

### Environment overrides

| Var | Effect |
| --- | --- |
| `CODEX_JS` | Path to `@openai/codex/bin/codex.js`; launched via `node` (most robust) |
| `CODEX_BIN` | Explicit Codex executable to spawn |

By default the launcher auto-resolves `codex.js` on Windows (avoids the `.cmd`
arg-escaping problem) and `codex` on POSIX.

## Honest limits

- **Concurrency is throttled by your Codex plan**, not by this script. Pushing
  `concurrency` very high hits rate limits (429) — realistic effective
  parallelism is single digits to low double digits. Start at 4–6.
- **Read-only is safe to fan out wide.** `sandbox: "read-only"` lets agents read
  but not edit, so many can share one working tree with zero conflict risk.
- **Writing in parallel needs isolation.** With `sandbox: "workspace-write"`,
  parallel agents in the *same* `cwd` clobber each other. Give each unit its own
  `git worktree` via per-unit `cwd`, then merge afterward. The harness warns when
  sandbox is not read-only.
- **No streaming.** Each unit is a separate process; you get its final message +
  full log when it closes, not a live stream.
