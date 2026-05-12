# Polywave for Codex CLI

<p align="center">
  <img src="https://raw.githubusercontent.com/blackwell-systems/polywave-protocol/main/assets/logo.png" alt="Polywave" width="600" />
</p>

[![Blackwell Systems](https://raw.githubusercontent.com/blackwell-systems/blackwell-docs-theme/main/badge-trademark.svg)](https://github.com/blackwell-systems)

**Parallel AI agents that don't break each other's code.** Now on [Codex CLI](https://github.com/openai/codex).

> **Status:** In development. See [IMPLEMENTATION-NOTES.md](IMPLEMENTATION-NOTES.md) for the current mapping and gaps.

## What is this?

This repo provides the Codex CLI implementation of the [Polywave protocol](https://github.com/blackwell-systems/polywave-protocol): an Agent Skill, custom agent definitions, and enforcement hooks that let Codex coordinate parallel agents with disjoint file ownership and worktree isolation.

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

## Install

```bash
# 1. Clone and install
git clone https://github.com/blackwell-systems/polywave-codex.git ~/code/polywave-codex
~/code/polywave-codex/install.sh

# 2. Install polywave-tools CLI (pick one)
brew install blackwell-systems/tap/polywave-tools                                     # Homebrew (recommended)
go install github.com/blackwell-systems/polywave-go/cmd/polywave-tools@latest   # Go install

# 3. Initialize your project
cd your-project
polywave-tools init

# 4. Verify
polywave-tools verify-install
```

**5. Restart Codex**, then run:

```
$polywave scout "add a caching layer to the API client"
```

## License

[MIT OR Apache-2.0](LICENSE)
