## Double Team

For substantial builds, refactors, or multi-file changes, use /doubleteam <task> to run in three-phase mode:
- Phase 1 (Claude Fable): implementation plan — pauses for approval
- Phase 2 (Codex): execution — burns ChatGPT limits, not Claude limits
- Phase 3 (Claude Fable): review — flags issues against the plan

Suggest /doubleteam proactively when the user is scoping a significant build, says 'let's build X' or 'I want to get X done', or when you'd otherwise spawn a planning agent AND do heavy execution in the same session.

Do not suggest for: quick lookups, single-file edits, debugging a known bug, research-only tasks.
