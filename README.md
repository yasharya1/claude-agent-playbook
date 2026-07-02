# Multi-Agent Orchestration Playbook

A drop-in Claude Code subagent setup: four specialist agents (`explorer`, `researcher`, `builder`, `reviewer`) plus the rules for when to spawn them.

**Live site:** (added after publishing)

## Use it in your own project

1. Copy `agents/*.md` into your project's `.claude/agents/` folder.
2. Copy the contents of `orchestration-rules.md` into your `CLAUDE.md` (or equivalent).
3. Adjust tool allowlists / model tiers to fit your project.

## What's in here

- `index.html` — the visual explainer (diagram, agent cards, illustrative timing comparison, decision guide)
- `agents/` — the four agent definition files, copy-paste ready
- `orchestration-rules.md` — when to spawn, when not to, worktree/fork rules
