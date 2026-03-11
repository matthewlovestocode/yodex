# codex-custom

This repository is intended to be used as a custom `CODEX_HOME` for the Codex desktop app.

It contains:
- `config.toml`: top-level Codex configuration
- `agents/`: agent role definitions
- `skills/`: local skills available to Codex
- `AGENTS.md`: repository-level instructions layered into the prompt stack
- `.codex/agent-scratchpad.md`: per-turn orchestration scratchpad written during multi-agent turns
- `.codex/agent-scratchpad.template.md`: checked-in scaffold for the live scratchpad

## Prerequisites

- macOS
- Codex desktop app installed
- [LaunchControl](https://www.soma-zone.com/LaunchControl/) or another way to set per-app environment variables for GUI apps
- `git`

## Clone The Repository

Choose a location where you want to keep your Codex home directory, then clone:

```bash
git clone <your-repo-url> /path/to/codex-custom
```

Example:

```bash
git clone https://github.com/your-org/codex-custom.git ~/codex-custom
```

## Point Codex At This Repository

The Codex desktop app reads its home directory from the `CODEX_HOME` environment variable. Set that variable to the cloned repository path.

### Using LaunchControl

1. Open LaunchControl.
2. Find the Codex desktop app in the list of user agents or applications.
3. Add an environment variable named `CODEX_HOME`.
4. Set its value to the full absolute path of your cloned repository.

Example value:

```text
/Users/yourname/codex-custom
```

5. Save the LaunchControl change.

## Restart Codex

After changing `CODEX_HOME`, fully quit and reopen the Codex desktop app.

Do not rely on just closing the window. The app should be fully restarted so it reloads the environment and configuration files.

## Verify It Loaded This Repo

After restart, confirm Codex is using this repository as its home:

1. Open Codex and start a new session.
2. Ask it to inspect its active config or list files in `CODEX_HOME`.
3. Confirm it sees files such as:
   - `config.toml`
   - `agents/default.toml`
   - `agents/explorer.toml`
   - `agents/worker.toml`
   - `agents/reviewer.toml`
   - `agents/port-explorer.toml`
   - `agents/port-worker.toml`
   - `agents/port-reviewer.toml`

You can also ask Codex something like:

```text
What is my current CODEX_HOME and what agent configs are available?
```

If the app is pointed at this repo, it should report this repository path and the agent files above.

## Repository Layout

```text
config.toml
AGENTS.md
agents/
  default.toml
  explorer.toml
  worker.toml
  reviewer.toml
  port-explorer.toml
  port-worker.toml
  port-reviewer.toml
skills/
rules/
memories/
sessions/
```

## Porting Agents

This repo now includes dedicated agents for repo-to-repo transfer work:

- `agents/port-explorer.toml`: maps source behavior to target insertion points, supporting files, and parity requirements
- `agents/port-worker.toml`: implements a complete port bundle, including supporting config, tests, and wiring
- `agents/port-reviewer.toml`: reviews ports for parity drift, omitted dependencies, and target-specific regressions

The default coordinator is instructed to prefer these agents for source-to-target transfer work instead of relying on the generic `explorer` and `worker` path.

## Porting Handoff Templates

Use these as the coordinator's default handoff shape for repo-to-repo transfer work.

### `port-explorer`

```text
goal: map the complete source feature surface into the target repo before implementation
scope: read-only analysis of source repo, target repo, related tests/config, and insertion points; no file edits
files: source feature files, target subsystem files, related tests, config, fixtures, scripts, and docs
constraints: prioritize parity, identify supporting files, distinguish direct carryover from adaptation, avoid implementation
execution_mode: dry_run
source_repo: /absolute/path/to/source-repo
target_repo: /absolute/path/to/target-repo
scan_mode: heavy
done_criteria: produce a migration map with source surface, target surface, dependency gaps, parity checklist, adaptation notes, and implementation order
return_format: summary, source_surface, target_surface, dependency_gaps, parity_checklist, adaptation_notes, implementation_order, open_questions
```

### `port-worker`

```text
goal: reproduce the source behavior in the target repo using the migration map and parity checklist
scope: modify only the target repo files needed for a complete port bundle, including code, config, tests, fixtures, scripts, docs, and wiring
files: exact target paths from the migration map plus any required supporting files discovered during implementation
constraints: preserve source behavior unless an explicit adaptation is required, record intentional deviations, stop if parity is invalidated by architecture mismatch
execution_mode: apply
source_repo: /absolute/path/to/source-repo
target_repo: /absolute/path/to/target-repo
scan_mode: n/a
done_criteria: complete the port bundle, report parity checklist status, run the most relevant checks, and call out residual risks or blockers
return_format: summary, files_changed, parity_completed, intentional_deviations, checks_run, checks_not_run, residual_risks, blocked_by
```

### `port-reviewer`

```text
goal: review the completed port for parity drift, hidden omissions, and target-specific regressions
scope: read-only review of the target changes against the expected source behavior and parity checklist
files: changed target files, key source reference files, parity checklist, and relevant tests/config
constraints: prioritize concrete findings, distinguish confirmed parity failures from validation gaps, avoid implementation unless explicitly requested
execution_mode: dry_run
source_repo: /absolute/path/to/source-repo
target_repo: /absolute/path/to/target-repo
scan_mode: medium
done_criteria: identify any parity failures, missing supporting pieces, validation gaps, and the overall port risk
return_format: findings, parity_gaps, validation_gaps, assumptions, bottom_line
```

## Notes

- `auth.json`, `state_5.sqlite`, and similar local runtime files may be created or updated by the app after launch.
- During multi-agent turns, the coordinator is instructed to replace `.codex/agent-scratchpad.md` for the current prompt and keep it updated with handoffs, contracts, statuses, and short result summaries.
- The checked-in `.codex/agent-scratchpad.template.md` file defines the stable scaffold without making the repo dirty on every prompt.
- If you move the repository later, update `CODEX_HOME` to the new absolute path and restart Codex again.
- If Codex appears to ignore changes, verify the environment variable is attached to the app process you are launching, not just to your shell.

## Scratchpad Template

Multi-agent turns should use this shape in `.codex/agent-scratchpad.md`:

```md
# Agent Scratchpad

## Turn
- Timestamp: 2026-03-10T12:34:56-07:00
- User request: Add an orchestration scratchpad for agent handoffs
- Workflow: explore_implement_review
- Status: in_progress

## Local Plan
- Critical path kept local: coordinator owns sequencing and final integration
- Delegated tasks: use sub-agents for bounded exploration, implementation, or review
- Acceptance criteria:
  - scratchpad file is replaced once per user turn
  - each delegated agent section includes handoff, contract, status, and result

## Agent: agent-1
- Role: explorer
- Status: completed
- Fork context: true
- Write scope: read-only
- Spawned at: 2026-03-10T12:35:02-07:00
- Completed at: 2026-03-10T12:35:18-07:00
- Goal: find the narrowest orchestration hook for scratchpad logging
- Scope: inspect config and agent definitions only
- Files: config.toml, agents/default.toml, README.md
- Constraints: no file edits, concise evidence only
- Execution mode: dry_run
- Done criteria: identify whether a runtime hook exists and recommend the best insertion point
- Return format: summary, evidence, recommendation
- Notes: no runtime hook found; prompt-level enforcement is the viable path
- Integration decision: accepted
- Decision reason: evidence matches repository structure

## Validation
- Checks run: `rg -n "agent-scratchpad|Scratchpad contract" config.toml agents/default.toml README.md`
- Checks not run: none
- Residual risks: prompt-level compliance depends on coordinator behavior

## Final Integration
- Outcome: added prompt-level scratchpad instructions and documented the file path
- Open issues: none
```
