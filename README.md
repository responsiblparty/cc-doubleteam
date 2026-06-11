# cc-doubleteam

Three-phase project mode for Claude Code: Claude plans, Codex executes, Claude reviews — execution burns your ChatGPT limits, not your Claude limits.

## Why

You get Fable-quality planning plus Codex execution without burning your Claude usage on the heavy lifting. The models work in sequence, each at their best effort level.

## Prerequisites

- Claude Code installed
- Codex CLI: `npm install -g @openai/codex` then `codex login` (requires paid ChatGPT Plus subscription)
- Codex plugin in Claude Code:
  ```
  /plugin marketplace add openai/codex-plugin-cc
  /plugin install codex@openai-codex
  ```

## Install

```sh
git clone https://github.com/responsiblparty/cc-doubleteam
cp -r cc-doubleteam/skills/doubleteam ~/.claude/skills/doubleteam
```

Optionally copy `CLAUDE.md` contents into your project's `CLAUDE.md` to enable proactive suggestions:

```sh
cat cc-doubleteam/CLAUDE.md >> /path/to/your/project/CLAUDE.md
```

Restart Claude Code after install. Type `/doubleteam` to confirm it's working.

## Usage

- `/doubleteam <task>` — starts immediately with your task
- `/doubleteam` — Claude asks what the task is
- Claude will also proactively suggest it when a session is heading into a substantial build

## How Phase 2 works

Phase 2 runs Codex by invoking the companion script directly via Bash — **not** through the `codex:rescue` subagent. Subagents inherit the session's original permission mode and will hit a Bash permission wall before they can invoke Codex. The skill uses dynamic path discovery so it works across any Codex plugin version:

```bash
COMPANION=$(ls ~/.claude/plugins/cache/openai-codex/codex/*/scripts/codex-companion.mjs 2>/dev/null | tail -1)
node "$COMPANION" task --effort xhigh --write "<plan>"
```

Phase 2 runs in the background by default — Claude notifies you when Codex finishes and you can keep talking in the meantime.

## The three phases

| Phase | Model | Effort | Role |
|---|---|---|---|
| 1 — Plan | Claude Fable (fallback: Opus) | high | Produces step-by-step implementation plan; pauses for your approval |
| 2 — Execute | Codex | xhigh | Implements the plan; burns ChatGPT limits, not Claude limits |
| 3 — Review | Claude Fable (fallback: Opus) | max | Reviews output against plan; flags issues without rewriting |

## Skipping phases

- `just plan` → Phase 1 only
- `skip review` → Phases 1 and 2 only
- If Codex is unavailable → falls back to Claude execution (noted in output)

## License

MIT
