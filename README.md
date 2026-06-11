# cc-doubleteam

Three-phase project mode for Claude Code: Claude plans, Codex executes, Claude reviews — execution burns your ChatGPT limits, not your Claude limits.

## Why

You get Fable-quality planning plus Codex execution without burning your Claude usage on the heavy lifting. The models work in sequence, each at their best effort level.

## Prerequisites

- Claude Code installed
- Codex CLI: `npm install -g @openai/codex` then `codex login`
- Codex plugin in Claude Code: `/plugin marketplace add openai/codex-plugin-cc` then install

## Install

```sh
git clone https://github.com/responsiblparty/cc-doubleteam
cp -r cc-doubleteam/skills/doubleteam ~/.claude/skills/doubleteam
```

Restart Claude Code after install.

## Usage

- `/doubleteam <task>` — starts immediately with your task
- `/doubleteam` — Claude asks what the task is
- Claude will also proactively suggest it when a session is heading into a substantial build

## The three phases

| Phase | Model | Effort | Role |
|---|---|---|---|
| 1 — Plan | Claude Fable 5 (fallback: Opus) | high | Produces step-by-step implementation plan; pauses for approval |
| 2 — Execute | Codex 5.5 | xhigh | Implements the plan; burns ChatGPT limits, not Claude limits |
| 3 — Review | Claude Fable 5 (fallback: Opus) | max | Reviews output against plan; flags issues without rewriting |

## Skipping phases

- `just plan` → Phase 1 only
- `skip review` → Phases 1 and 2 only
- If Codex is unavailable → falls back to Claude execution (noted in output)

## License

MIT
