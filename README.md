<h1 align="center">Polywave for Codex CLI</h1>

<p align="center">
  <img src="https://raw.githubusercontent.com/blackwell-systems/polywave-protocol/main/assets/logo.png" alt="Polywave" width="600" />
</p>

<p align="center">
  <a href="https://github.com/blackwell-systems"><img src="https://raw.githubusercontent.com/blackwell-systems/blackwell-docs-theme/main/badge-trademark.svg" alt="Blackwell Systems" /></a>
  <a href="https://buymeacoffee.com/blackwellsystems"><img src="https://img.shields.io/badge/buy%20me%20a%20coffee-donate-yellow.svg" alt="Buy Me A Coffee" /></a>
</p>

<p align="center"><strong>Parallel AI agents that don't break each other's code.</strong> Now on <a href="https://github.com/openai/codex">Codex CLI</a>.</p>

> **Status:** Early implementation. The hook harness and hook installer exist, but this is not a runnable Polywave implementation yet. See [IMPLEMENTATION-NOTES.md](IMPLEMENTATION-NOTES.md) for the current mapping and enforcement notes, and [ROADMAP.md](ROADMAP.md) for the active implementation plan.

## What is this?

This repo will provide the Codex CLI implementation of the [Polywave protocol](https://github.com/blackwell-systems/polywave-protocol): an Agent Skill, custom agent definitions, and enforcement hooks that let Codex coordinate parallel agents with disjoint file ownership and worktree isolation.

## Repositories

| Repository | Purpose |
|-----------|---------|
| [polywave-protocol](https://github.com/blackwell-systems/polywave-protocol) | Protocol specification |
| [polywave](https://github.com/blackwell-systems/polywave) | Claude Code implementation |
| **polywave-codex** (this repo) | Codex CLI implementation |
| [polywave-go](https://github.com/blackwell-systems/polywave-go) | Go engine and `polywave-tools` CLI |
| [polywave-web](https://github.com/blackwell-systems/polywave-web) | Web UI (optional) |

> **Using Claude Code?** Start at [polywave](https://github.com/blackwell-systems/polywave).

## Prerequisites

- [Codex CLI](https://github.com/openai/codex) installed
- Git 2.20+
- jq 1.6+
- `polywave-tools` CLI (see below)

## Current Install

The current installer installs the hook package, the `polywave` skill, the progressive-disclosure reference pack and injector scripts, and direct Codex ports of the current Polywave custom agents: `polywave-scout`, `polywave-wave-agent`, `polywave-planner`, `polywave-scaffold-agent`, `polywave-critic-agent`, and `polywave-integration-agent`. The full runnable Polywave flow is still in progress.

```bash
./install.sh
```

Use `--write-user-hooks` only if `~/.codex/hooks.json` does not already exist and you want the installer to create it.

Verify the installed Codex artifacts with:

```bash
./scripts/verify-codex-install
```

To remove installed hook artifacts:

```bash
./uninstall.sh
```

## Planned Polywave Flow

After the full Codex implementation exists, the intended project flow is:

```bash
# 1. Install polywave-tools CLI (pick one)
brew install blackwell-systems/tap/polywave-tools
go install github.com/blackwell-systems/polywave-go/cmd/polywave-tools@latest

# 2. Initialize your project
cd your-project
polywave-tools init

# 3. Install the Codex skill/agents/hooks
~/code/polywave-codex/install.sh

# 4. Verify the Codex installation
polywave-tools verify-install
```

After the Codex implementation exists, restart Codex and run:

```
$polywave scout "add a caching layer to the API client"
```

## Primary Execution Surface

The primary product surface is the active Codex CLI loop with the installed `$polywave` skill.

Use the skill in the live Codex session for real Polywave orchestration. The repo-local scripts are supporting tools for development, debugging, and fallback automation; they are not the primary interface.

## Progressive Disclosure

> **Codex limitation:** Codex does not support Claude Code's automatic subagent prompt rewrite path (`updatedInput` on agent launch), so progressive disclosure references must be loaded explicitly by the `$polywave` orchestrator or the fallback launcher scripts.

The Codex port now carries the same basic disclosure structure as the Claude implementation:

- core orchestrator skill in `SKILL.md`
- on-demand protocol references in `references/`
- deterministic routing scripts in `scripts/inject-context` and `scripts/inject-agent-context`
- deterministic prompt builders in `scripts/build-scout-prompt` and `scripts/build-wave-agent-prompt`

Because Codex does not expose Claude's agent-prompt rewrite hooks, the live `$polywave` skill must call these scripts explicitly inside the active loop before delegating to Polywave custom agents. The prompt builders now centralize that assembly so the live loop and fallback launchers use the same generated prompt shape.

The exact primary-path procedure is documented in `references/live-loop-playbook.md` and is now the expected in-loop orchestration contract.

## Minimal Scout Flow

What exists now is the first concrete scout path, not a full orchestration layer.

Current proven state on the primary path: a live `$polywave scout` proof run wrote a real IMPL manifest and `polywave-tools finalize-scout` passed.

Current state on wave execution: the corrected fallback worker path now completes end to end in a clean proof repo. Two primary-loop worker models have now been tested and both are blocked by current Codex runtime behavior: in-session spawned workers fail at git worktree metadata writes, and nested `codex exec --cd <worktree>` launches from inside the active Codex loop fail earlier with `failed to initialize in-process app-server client: Operation not permitted`. See implementation notes for the exact evidence and current boundary.

1. Install `polywave-tools` and run `./install.sh` from this repo.
2. Add the target-repo guidance snippet to the target repository if needed:

```bash
./scripts/print-target-agents-snippet
```

3. In Codex, invoke the installed `polywave` skill and provide three explicit inputs:
   - repository root
   - feature description
   - absolute IMPL output path

The current scout contract is:
- `polywave-scout` writes the IMPL manifest
- the orchestrator validates the IMPL through `polywave-tools`
- this is the required path for claiming a real scout run

Development/fallback launcher:

```bash
scripts/run-polywave-scout --repo-dir /path/to/repo --feature "add a caching layer"
```

## Minimal Wave Flow

Development/fallback launcher:

```bash
scripts/run-polywave-wave /path/to/docs/IMPL/IMPL-feature.yaml --wave 1 --repo-dir /path/to/repo
```

Current behavior:
- calls `polywave-tools prepare-wave --json-only`
- launches one Codex run per prepared agent worktree
- calls `polywave-tools finalize-wave` unless `--skip-finalize` is set
- keeps `polywave-tools` as the authority for worktree prep and merge/finalize

## Development

Run the current hook fixture suite with:

```bash
scripts/run-hook-fixtures
```

To capture real Codex hook payloads, temporarily add the optional audit snippet printed by:

```bash
scripts/print-codex-config
```

Payloads are written under `${CODEX_HOME:-$HOME/.codex}/polywave/audit`.

Note: `codex exec` on Codex CLI 0.130.0 did not emit audit payloads for a simple shell command during local probing, even with hooks enabled. Validate against the interactive Codex path before relying on runtime hook coverage.

## License

[MIT OR Apache-2.0](LICENSE)
