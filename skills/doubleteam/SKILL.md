---
name: doubleteam
description: Three-phase project mode — Fable plans (high effort), Codex executes (xhigh, burns ChatGPT limits not Claude limits), Fable reviews (max effort). Invoke as /doubleteam <task> or let Claude suggest it when a session is getting deep.
---

# Double Team — Three-Phase Project Mode

## When to invoke (proactive trigger)

Suggest `/doubleteam` without being asked when:
- The user is scoping a substantial build, refactor, or multi-file change
- A task will require both planning and non-trivial code execution
- The user says something like "let's build X", "I want to get X done", "let's really dig into X"
- You find yourself about to spawn a Fable agent AND do significant execution work in the same session

Say something like: *"This is a good candidate for `/doubleteam` — want me to run it in three-phase mode so execution burns Codex limits instead of Claude?"*

Do NOT suggest it for: quick lookups, single-file edits, debugging a known bug, anything that's just research.

## Invocation

Two modes:
- `/doubleteam <task description>` — run immediately with that task
- `/doubleteam` with no args — ask the user: "What's the task?"

## Execution protocol

Run the three phases in sequence. Do not skip or merge phases.

---

### Phase 1 — Plan (Claude Fable 5, effort: high)

Spawn a general-purpose subagent with `model: "fable"` to produce the implementation plan.

Prompt the subagent with:
- The full task description
- Relevant file paths, repo context, or constraints already known in the session
- Explicit instruction: apply high reasoning effort. Think carefully before committing to any design decision. Produce a concrete, step-by-step implementation plan. Include: files to create/modify, function signatures or data shapes that matter, sequencing, and any gotchas. No implementation — plan only.

Wait for the subagent to return. Present the plan to the user with a brief "Phase 1 complete — here's the plan:" header.

Pause and ask: "Good to proceed to execution, or want to adjust the plan first?"

---

### Phase 2 — Execute (Codex xhigh)

Once the user approves the plan, delegate execution to Codex via `codex:rescue`.

Compose the Codex prompt using the `codex:gpt-5-4-prompting` skill structure:
- `<task>`: the concrete implementation steps from Phase 1, translated into Codex operator language
- `<default_follow_through_policy>`: implement all steps; if a detail is ambiguous, choose the simplest correct interpretation and note it
- `<completeness_contract>`: all files in the plan must be touched; do not leave stubs
- `<action_safety>`: stay strictly within the scope of the plan; no unrelated refactors or cleanup

Pass `--effort xhigh` to the rescue invocation. Do not set `--model` (Codex uses its current default model).

After rescue returns, present the output to the user with a "Phase 2 complete — Codex executed:" header. Note that this burned Codex (ChatGPT) limits, not Claude limits.

---

### Phase 3 — Review (Claude Fable 5, effort: max)

Spawn a general-purpose subagent with `model: "fable"` to review the execution output.

Prompt the subagent with:
- The original task
- The Phase 1 plan
- The Phase 2 Codex output / changed files (provide paths)
- Explicit instruction: apply maximum reasoning effort. Be adversarial — assume there are bugs until proven otherwise. Review for correctness against the plan, edge cases missed, security issues, and anything that looks wrong or incomplete. Be specific — cite file and line. Do not rewrite; flag only.

Present the review findings with a "Phase 3 complete — review:" header.

Then summarize: what was built, what the review flagged, and what (if anything) needs a follow-up pass.

---

## Limits and resets

- If the user says "just plan" or "skip execution" — run Phase 1 only.
- If the user says "skip review" — run Phases 1 and 2 only.
- If Fable is unavailable, spawn with model: "opus" as fallback for Phase 1 and Phase 3.
- If Phase 2 fails or Codex is unavailable — fall back to executing in Claude (note the fallback and that it will burn Claude limits).
- After completion, offer: "Want another pass with `/doubleteam` on anything the review flagged?"
