---
name: repo-porting
description: Plan and execute repo-to-repo feature or subsystem transfers with parity-focused agent handoffs. Use when Codex needs to move behavior, code, config, tests, fixtures, scripts, or docs from a source repo into a target repo, especially when shallow generic explore/implement passes keep missing dependencies or requiring rework.
---

# Repo Porting

Use this skill when a task is really a source-to-target migration, not a local edit. Treat the port as a bundled behavior transfer that may require source mapping, target-side recomposition, implementation, parity review, and scratchpad updates.

## Workflow

1. Identify the source repo, target repo, feature boundary, whether recomposition is likely, and any known intentional deviations.
2. Route the task through `port-explorer` first unless the change is trivially mechanical.
3. Require a migration map before implementation starts.
4. Route implementation through `port-worker` with the migration map and parity checklist.
5. Route validation through `port-reviewer` when the task is not trivially mechanical.
6. Update the scratchpad before delegation and after each agent result.

## Required Inputs

Collect or infer these before delegating:

- `source_repo`: absolute path or repo identifier
- `target_repo`: absolute path or repo identifier
- `feature_goal`: the behavior that must exist in the target after the port
- `recomposition_constraints`: any known target boundaries, primitives, or ownership rules that require the source behavior to be split, merged, or re-expressed
- `allowed_deviations`: explicit differences allowed by the user or target architecture
- `validation_expectations`: tests, commands, or checks that should prove parity if available

If a required repo path or feature boundary is missing, ask one concise question or infer the narrowest reasonable boundary and state the assumption.

## Handoff Order

### `port-explorer`

Use `scan_mode: heavy` by default. Downgrade to `medium` or `light` only when the source surface is already narrow and verified.

Send this contract:

```text
goal: map the complete source feature surface into the target repo before implementation, including any required recomposition into target-native boundaries
scope: read-only analysis of source repo, target repo, related tests/config, and insertion points; no file edits
files: source feature files, target subsystem files, related tests, config, fixtures, scripts, and docs
constraints: prioritize parity, identify supporting files, distinguish direct carryover from recomposition or adaptation, avoid implementation
execution_mode: dry_run
source_repo: <absolute path or repo identifier>
target_repo: <absolute path or repo identifier>
scan_mode: heavy
done_criteria: produce a migration map with source surface, target surface, dependency gaps, parity checklist, recomposition plan, adaptation notes, and implementation order
return_format: summary, source_surface, target_surface, dependency_gaps, parity_checklist, recomposition_plan, adaptation_notes, implementation_order, open_questions
```

Accept the result only if it identifies:

- the complete source surface
- target insertion points
- dependency or harness gaps
- a concrete parity checklist
- a recomposition plan whenever the target shape differs materially from the source
- a recommended implementation order

### `port-worker`

Use this only after the migration map is good enough that the worker should not need to rediscover the subsystem.

Send this contract:

```text
goal: reproduce the source behavior in the target repo using the migration map, recomposition plan, and parity checklist
scope: modify only the target repo files needed for a complete port bundle, including code, config, tests, fixtures, scripts, docs, and wiring
files: exact target paths from the migration map plus any required supporting files discovered during implementation
constraints: preserve source behavior unless an explicit adaptation is required, allow bounded recomposition into target-native structures, record intentional deviations, stop if parity is invalidated by architecture mismatch
execution_mode: apply
source_repo: <absolute path or repo identifier>
target_repo: <absolute path or repo identifier>
scan_mode: n/a
done_criteria: complete the port bundle, report parity checklist status and recomposition outcomes, run the most relevant checks, and call out residual risks or blockers
return_format: summary, files_changed, parity_completed, recomposition_completed, intentional_deviations, checks_run, checks_not_run, residual_risks, blocked_by
```

Reject partial “code-only” ports when supporting config, tests, fixtures, scripts, docs, or runtime wiring are still missing.

### `port-reviewer`

Use this after implementation unless the task is trivially mechanical and low risk.

Send this contract:

```text
goal: review the completed port for parity drift, hidden omissions, recomposition mistakes, and target-specific regressions
scope: read-only review of the target changes against the expected source behavior and parity checklist
files: changed target files, key source reference files, parity checklist, and relevant tests/config
constraints: prioritize concrete findings, distinguish confirmed parity failures from validation gaps, verify that recomposed target pieces still preserve source behavior, avoid implementation unless explicitly requested
execution_mode: dry_run
source_repo: <absolute path or repo identifier>
target_repo: <absolute path or repo identifier>
scan_mode: medium
done_criteria: identify any parity failures, recomposition gaps, missing supporting pieces, validation gaps, and the overall port risk
return_format: findings, parity_gaps, recomposition_gaps, validation_gaps, assumptions, bottom_line
```

Escalate any finding that shows:

- missing supporting files
- config drift
- renamed but unwired symbols
- harness mismatches
- source behavior lost while being split, merged, or re-expressed in target-native structures
- source behavior silently changed during adaptation

## Scratchpad Discipline

When multi-agent work is used:

1. Replace `/Users/matthewstewart/codex-custom/.codex/agent-scratchpad.md` before the first substantial delegation.
2. Record the workflow as `port_then_review` if the task uses the full source-to-target path.
3. Add one section per delegated agent with role, status, files, constraints, execution mode, timestamps, notes, and integration decision.
4. Update the scratchpad after each completed agent so parity assumptions and blocked states are auditable.

## Coordinator Rules

- Prefer one heavy exploration pass, one bounded implementation pass, and one parity review pass over repeated shallow loops.
- When source and target shapes differ, require an explicit recomposition plan instead of letting the worker infer structure ad hoc.
- If the same missing dependency or parity gap appears twice, stop and tighten the migration map instead of sending another shallow worker pass.
- Never finalize a port as complete while high-confidence parity gaps remain unresolved.
- In the final answer, summarize the parity outcome, intentional deviations, checks run, checks not run, and residual risks.

## Example Prompt

Use this when you want a no-change audit of the workflow:

```text
Use $repo-porting.

Dry run only. Do not change files.

Source repo: /absolute/path/to/source-repo
Target repo: /absolute/path/to/target-repo

Port this feature from source to target:
<describe the feature or subsystem clearly>

Requirements:
- Use the repo-porting workflow.
- Start with `port-explorer` using a heavy scan.
- Build a migration map and parity checklist before any implementation decision.
- Call out where the target requires the source behavior to be decomposed, merged, or re-expressed.
- Do not run `port-worker` in apply mode unless I explicitly approve a non-dry-run follow-up.
- Run `port-reviewer` against the proposed migration plan and any identified parity risks.
- Maintain the scratchpad at `/Users/matthewstewart/codex-custom/.codex/agent-scratchpad.md`.
- In the final answer, give me:
  - the source surface
  - the target surface
  - dependency gaps
  - the parity checklist
  - the recomposition plan
  - recommended implementation order
  - intentional deviations needed
  - major risks
```
