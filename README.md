# codex-custom

This repository is intended to be used as a custom `CODEX_HOME` for the Codex desktop app.

It contains:
- `config.toml`: top-level Codex configuration
- `agents/`: agent role definitions
- `skills/`: local skills available to Codex
- `AGENTS.md`: repository-level instructions layered into the prompt stack

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
skills/
rules/
memories/
sessions/
```

## Notes

- `auth.json`, `state_5.sqlite`, and similar local runtime files may be created or updated by the app after launch.
- If you move the repository later, update `CODEX_HOME` to the new absolute path and restart Codex again.
- If Codex appears to ignore changes, verify the environment variable is attached to the app process you are launching, not just to your shell.
