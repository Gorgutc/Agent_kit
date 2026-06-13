# Harness evals

Dependency-free behavior checks for the agent harness. Each `NN-*.md` is a
copy-paste **Prompt** plus the **Expected** behaviors a correct agent should
exhibit. Run one by pasting its Prompt into a fresh Codex / Claude Code session in
a project that uses this kit, then compare what the agent does against Expected.

These are smoke tests for the instructions / skills / hooks — not automated tests.
Use them after changing AGENTS.md, a skill, or a hook to confirm the harness still
steers behavior correctly.
