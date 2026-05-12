# Polywave Codex Implementation Notes

## Overview

This is the OpenAI Codex CLI implementation of the Polywave protocol. It provides the same parallel agent coordination as the Claude Code implementation, adapted for Codex's skill, hook, and subagent systems.

## Platform Mapping

### Skills

| Claude Code | Codex CLI |
|---|---|
| `~/.claude/skills/polywave/SKILL.md` | `~/.agents/skills/polywave/SKILL.md` or `.agents/skills/polywave/SKILL.md` |
| Invocation: `/polywave scout` | Invocation: `$polywave` or explicit mention |
| Progressive disclosure via hooks | Progressive disclosure native (Codex loads SKILL.md on activation) |

Codex uses the same Agent Skills standard (SKILL.md format). The skill content needs adaptation for Codex-specific tool names and invocation patterns.

### Agent Definitions

| Claude Code | Codex CLI |
|---|---|
| `.md` files in `~/.claude/agents/` or `~/.claude/skills/polywave/agents/` | `.toml` files in `~/.codex/agents/` or `.codex/agents/` |
| Fields: markdown body with YAML frontmatter | Fields: `name`, `description`, `developer_instructions` (TOML) |
| `subagent_type: scout` | Codex spawns by agent name from custom agent definitions |

Each Claude Code agent prompt (scout.md, wave-agent.md, scaffold-agent.md, integration-agent.md, critic-agent.md, planner.md) needs conversion to a TOML agent definition with `developer_instructions` containing the prompt content.

### Hooks

| Event | Claude Code | Codex CLI | Status |
|---|---|---|---|
| SessionStart | Supported | Supported | Direct port |
| PreToolUse | Supported, matcher on tool name | Supported, matcher on tool name | Direct port |
| PostToolUse | Supported | Supported | Direct port |
| UserPromptSubmit | Supported, `additionalContext` | Supported, `additionalContext` | Direct port |
| Stop | Supported | Supported | Direct port |
| PermissionRequest | Supported | Supported | Direct port |
| **SubagentStart** | **Supported** | **Not available** | Gap: see below |
| **SubagentStop** | **Supported** | **Not available** | Gap: see below |

**Hook config location:** `~/.codex/hooks.json` or `~/.codex/config.toml` (not `settings.json`)

**Feature flag required:** `codex_hooks = true` in `config.toml`

**Key difference:** Codex matchers use `apply_patch` (aliased to `Edit|Write`) instead of Claude Code's `Write|Edit` tool names.

### Configuration

| Claude Code | Codex CLI |
|---|---|
| `~/.claude/settings.json` | `~/.codex/config.toml` |
| `polywave.config.json` (project) | `polywave.config.json` (project, same) |
| Permissions in `settings.json` | Sandbox mode and exec policy in `config.toml` |

## Enforcement Gaps and Workarounds

### Gap 1: No SubagentStart/SubagentStop hooks

**Impact:** Cannot mechanically inject environment variables or validate worktree isolation at subagent lifecycle boundaries.

**Claude Code hooks affected:**
- `inject_worktree_env` (SubagentStart): sets POLYWAVE_AGENT_WORKTREE, POLYWAVE_AGENT_ID, POLYWAVE_WAVE_NUMBER, POLYWAVE_IMPL_PATH, POLYWAVE_BRANCH
- `validate_agent_isolation` (SubagentStart): verifies agent is in correct worktree
- `validate_worktree_isolation` (SubagentStart): verifies worktree directory exists
- `validate_agent_completion` (SubagentStop): blocks if protocol obligations unfulfilled
- `emit_agent_completion` (SubagentStop): observability events
- `verify_worktree_compliance` (SubagentStop): verifies completion report and commits
- `polywave_critic_impl_commit` (SubagentStop): commits IMPL doc before critic stops

**Workaround:** 
- Env vars: bake worktree path, agent ID, wave number into the custom agent's `developer_instructions` at launch time (orchestrator constructs the TOML dynamically or uses polywave-tools to generate agent configs)
- Isolation validation: use PreToolUse on first Bash/apply_patch call to verify cwd is correct worktree
- Completion validation: use Stop hook to check completion report exists
- Observability: use Stop hook for event emission
- Critic commit: include explicit "commit IMPL doc before stopping" instruction in critic agent definition

### Gap 2: No updatedInput support

**Impact:** Cannot mechanically rewrite bash commands to prepend `cd $WORKTREE &&`.

**Claude Code hook affected:** `inject_bash_cd` (PreToolUse:Bash)

**Workaround:** 
- Cooperative: include explicit "always cd to your worktree before running commands" in wave agent `developer_instructions`
- Defensive: PreToolUse hook checks if bash command targets correct directory; deny with reason if not (can't rewrite, but can block and explain)
- Alternative: set the custom agent's working directory in its config if Codex supports per-agent cwd

### Gap 3: No additionalContext in PreToolUse

**Impact:** Cannot inject conditional references into agent prompts at launch time.

**Claude Code hook affected:** `validate_agent_launch` (PreToolUse:Agent, uses `updatedInput` for conditional reference injection)

**Workaround:**
- Pre-bake conditional references into the custom agent definition
- Use polywave-tools to generate agent TOML with all necessary context embedded

## File Structure

```
polywave-codex/
├── SKILL.md                    # Agent Skill (same format as Claude Code)
├── references/                 # On-demand reference docs
├── agents/                     # Custom agent definitions (TOML)
│   ├── scout.toml
│   ├── wave-agent.toml
│   ├── scaffold-agent.toml
│   ├── integration-agent.toml
│   ├── critic-agent.toml
│   └── planner.toml
├── hooks/                      # Enforcement hooks (bash/python scripts)
│   ├── hooks.json              # Hook registration config
│   ├── check_scout_boundaries
│   ├── check_wave_ownership
│   ├── validate_write_paths
│   ├── check_branch_drift
│   ├── warn_stubs
│   └── ...
├── scripts/                    # Utility scripts
├── install.sh                  # Installer
├── README.md
├── QUICKSTART.md
└── IMPLEMENTATION-NOTES.md     # This file
```

## Implementation Priority

### Phase 1: Minimal Viable (can run /polywave scout)
1. SKILL.md adapted for Codex invocation patterns
2. Scout agent definition (TOML)
3. Install script (symlinks to ~/.agents/skills/polywave/ and ~/.codex/agents/)
4. Basic hooks.json with feature flag setup

### Phase 2: Wave Execution (can run full wave lifecycle)
5. Wave agent definition (TOML)
6. Scaffold agent definition (TOML)
7. PreToolUse hooks: ownership checking, write path validation, scout boundaries
8. PostToolUse hooks: stub warnings, branch drift detection
9. Worktree isolation enforcement (cooperative + defensive)

### Phase 3: Full Parity
10. Integration agent, critic agent, planner agent definitions
11. Stop hook for completion validation
12. SessionStart hook for context injection
13. UserPromptSubmit hook for skill context routing
14. Observability event emission
15. Progressive disclosure references

### Phase 4: Polish
16. QUICKSTART with Codex-specific walkthrough
17. Installer handles config.toml feature flag
18. polywave-tools verify-install support for Codex
19. Test with real Codex execution

## Open Questions

1. Does Codex support per-subagent working directory? If so, we can set cwd to the worktree in the agent config instead of relying on cd injection.
2. Can custom agent TOML files be generated dynamically at runtime? The orchestrator needs to inject worktree paths and agent IDs into developer_instructions at wave prep time.
3. How does Codex handle subagent file writes? Does the sandbox boundary scope to the subagent's cwd or the parent session's cwd?
4. What is the max concurrent subagent count? (Default max_threads = 6)
5. Does Codex support `isolation: "worktree"` or equivalent per-agent isolation parameter?
