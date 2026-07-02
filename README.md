# Multi-Agent Orchestration Playbook

Explains why and when to split work across Claude subagents — one track for Claude Code, one for the Cowork desktop app, switchable with a toggle on the page. Built for Klue's AI Ops team.

**Live site:** https://yasharya1.github.io/claude-agent-playbook/

## Use it in your own project

**Claude Code:**
1. Copy `agents/*.md` into your project's `.claude/agents/` folder.
2. Copy the contents of `orchestration-rules.md` into your `CLAUDE.md` (or equivalent).
3. Adjust tool allowlists / model tiers to fit your project.

**Cowork (desktop app):**
1. Copy the contents of `cowork-orchestration.md` into Cowork → Settings → Global Instructions.

## What's in here

- `index.html` — the visual explainer (diagrams, delegation rules, illustrative timing comparisons, decision guides) for both tools
- `agents/` — the four Claude Code agent definition files, copy-paste ready
- `orchestration-rules.md` — Claude Code: when to spawn, when not to, worktree/fork rules
- `cowork-orchestration.md` — Cowork: batching rules, split-instruction examples, what it can't do
