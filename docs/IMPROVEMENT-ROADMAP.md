# agent-kit — Improvement Roadmap

Synthesis of a 5-repo cross-analysis (2026-06-14) by five parallel Opus 4.8 agents,
each auditing one mature harness:

| Repo | Stack | Harness shape | Skills |
| --- | --- | --- | --- |
| `Gorgutc/codex` | static web | dual (Codex+Claude), generated `.claude` mirror + parity gate | 17 |
| `Gorgutc/PL_RU` | Next.js/TS | dual, two co-equal canons, nested sub-project AGENTS.md | 15 |
| `Gorgutc/LLPlayer_ru` | .NET/WPF | **tri**-harness (AGENTS/CLAUDE/GEMINI redirect stubs) | 15 |
| `Gorgutc/Achivments_addon_blender` | Python/Blender | Codex-primary single harness | 13 |
| `Gorgutc/3d_in_blueprints` | backend+addon+exe | tri-harness, **component profiles** | 22 |

All five are richer than agent-kit. The goal is to absorb the *generic, portable*
mechanisms while keeping agent-kit lightweight (no heavy sync/parity machinery,
flat-by-default, read-only-first).

`Consensus` = how many of the 5 analysts independently recommended it.

## Tier 1 — generic, high-consensus, low effort  ✅ IMPLEMENTED 2026-06-14

_All of T1–T8 are live in the kit; `node tools/check-kit.mjs` is green (24/24)._

| # | Improvement | Consensus | How (concrete) | Effort |
| --- | --- | --- | --- | --- |
| T1 | **Path-scope + hard-block `post-tool-verify`** | 4/5 | Read `verifyPaths` globs from `.agent-kit.json`; only run when a changed path matches (union `git diff` + `--cached` + `ls-files --others`, normalize `\`→`/`); `process.exit(2)` on failure so the agent is blocked, not just warned. Default (no `verifyPaths`) = run always (back-compat). | S |
| T2 | **Layered verify tiers** | 4/5 | Replace single `verifyCommand` with `verify: { fast, deep, ship }` in `.agent-kit.json` (all optional; `verifyCommand` kept as alias). Post-tool hook runs `fast`; orchestrator/ship runs `deep`/`ship`. | S |
| T3 | **Orchestration contract** | 4/5 | Add to `AGENTS.md` + `fanout-orchestrator` skill: a 7-field **Subagent Prompt Contract** (Documentation / Selected skills / Selected agents / Write zone / Verification / Stop rules / Expected output), a **Routing/Decomposition Matrix** (stream → agent → write-zone → parallel/sequential/local + reason), and a **Result Contract** ("defers cannot override blockers"). | S |
| T4 | **Two missing read-only subagents** | 5/5 | Add `instruction_drift_auditor` (audits the harness/instructions for drift) and `verification_reviewer` (final verify gate) as `.codex/agents/*.toml` + `.claude/agents/*.md` mirrors. | S |
| T5 | **GEMINI.md tri-harness stub** | 2/5 | Ship a ~5-line `GEMINI.md` redirect pointing at AGENTS.md. Document the rule: *additional harnesses get a redirect stub only — canonical skills are never mirrored per harness.* | S |
| T6 | **Configurable nudge triggers + degradation rule** | 3/5 | Make the broad-task keyword list in `user-prompt-nudge.js` read `broadTaskTriggers` from `.agent-kit.json` (default = current EN+RU set). Add the "if no spawn tool is available, document the fallback review path" rule to AGENTS.md. | S |
| T7 | **Generic skill family** | 5/5 | Add stack-agnostic skills (Claude + Codex): `session-bootstrap` (read AGENTS → `git status` → classify narrow/broad → pick skills/agents), `context-keeper` (carry key facts), `frozen-decisions` (respect a frozen list), `instruction-drift` (audit instructions). Templated with `{{verifyCommand}}` etc. | M |
| T8 | **Harness self-verifier** | 5/5 | Add zero-dep `tools/check-kit.mjs`: assert AGENTS/CLAUDE/GEMINI exist and reference AGENTS.md; every `.codex/agents/*.toml` is `read-only` and has a `.claude/agents/*.md` mirror; hook scripts referenced by `hooks.json`/`settings.json` exist; example specs parse. This is the "lightweight parity" that respects the no-heavy-sync goal. | M |

## Tier 2 — high value, opt-in or needs adaptation  ✅ IMPLEMENTED 2026-06-14

_All of T9–T16 are live in the kit, all opt-in (empty config = the prior flat
behavior). `node tools/check-kit.mjs` stays green._

| # | Improvement | Consensus | How (concrete) | Effort |
| --- | --- | --- | --- | --- |
| T9 | **Optional component profiles (active/dormant)** | 2/5 | Add `profiles: [{ name, status: "active"\|"dormant", doc }]` to `.agent-kit.json` (default `[]` = today's flat behavior); `docs/profiles/<name>.md` template; a profile-aware `component_guardian` subagent ("while dormant, assert no toolchain active; when activated, require stack/build/verify decisions"). Surfaced by session-start. | M |
| T10 | **Diff-scoped, fail-closed evidence gate** | 1/5 (PL_RU top find) | Generic `tools/evidence-gate.mjs`: read `{changedGlob → requiredEvidenceFile}` pairs from `.agent-kit.json`; compute changed files from git; fail closed if a glob matched but its evidence file is absent. (Visual QA is just one config instance.) | M |
| T11 | **Self-installing git hooks + lefthook template** | 2/5 | Port an ownership-marked `tools/install-hooks.mjs` (refuses to clobber foreign hooks, worktree-aware) and/or an opt-in `lefthook.yml` running `verify.fast` pre-commit, `verify.ship` pre-push. | M |
| T12 | **Negative-contract + stale/forbidden-term scan** | 3/5 | Because the kit gets cloned and inherits prior rules: a `staleTerms`/`forbiddenTerms` array in `.agent-kit.json`, scanned by `check-kit.mjs` against active docs/skills, fail with file:line. Plus a "Stack guardrails / what NOT to port" section in GETTING-STARTED. | S |
| T13 | **Nested-AGENTS.md composition docs** | 1/5 (PL_RU top-3) | Document monorepo composition in GETTING-STARTED: sub-dir `CLAUDE.md` = `@AGENTS.md`; sub-dir `AGENTS.md` = override-only delta with `BEGIN/END:<id>` managed-block markers; vendor folders fenced read-only. | S |
| T14 | **Frozen-decisions doc + same-change rule** | 4/5 | `docs/frozen-decisions.md` template (Canonical files / Known drift / Do-not-touch / Future-change rule) + the rule "change a durable contract ⇒ update the doc AND its verifier in the same change." Pairs with the `frozen-decisions` skill (T7). | S |
| T15 | **Scenario evals** | 1/5 | `evals/` with 5–7 `NN-*.md` prompt + "Expected behaviors" files (bootstrap, code-change-ship, doc-only, orchestration, drift) — dependency-free behavior regression for the harness. | S |
| T16 | **Mirror banners + parity footers** | 2/5 | Add a "generated/mirror — keep in sync" banner to each `.claude/agents/*.md` and a one-line note in AGENTS.md listing which dirs are mirrors. | S |

## Explicitly out of scope (project-specific — do NOT copy)
Stack content: motion/assets/copy/visual-QA/SCSS/Blender-runtime/dotnet-rules skills;
the 480–620-line bespoke `verify-frozen` scripts; per-feature handoff docs; the
I1–I7 iteration contracts; inherited dormant frontend skill stubs. agent-kit should
teach these as *patterns/templates*, never ship the content.

## Notes on design tension
Several findings (parity check, frozen verifiers, profiles, git hooks) come from
repos heavier than agent-kit intends to be. The resolution everywhere: ship the
**mechanism as opt-in** (empty config = today's behavior) so the lightweight,
generic, flat-by-default core is preserved.
