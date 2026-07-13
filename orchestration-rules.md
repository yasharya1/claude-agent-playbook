# Agent Orchestration Rules

Drop this into your project's `CLAUDE.md` (or equivalent) alongside the four agent files in `agents/`.

Spawn subagents aggressively when the task benefits — don't stay in the main context when a better tool exists. Agent teams (see below) are a separate, opt-in capability: they won't work until you set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in your own project's `.claude/settings.json` (Claude Code v2.1.178+). If they silently stop working after an update, check this flag first — it's experimental and may change.

## When to spawn subagents

| Situation | Agent to use |
|-----------|-------------|
| Simple file lookups, grep for symbols, find file paths | `explorer` — Haiku, read-only, cheap |
| Cross-file analysis, pattern understanding, "explain the gotchas" | `explorer` — override to `sonnet` at call site |
| External research, docs lookups, web searches | `researcher` — Sonnet; also has Google Drive access for hybrid tasks |
| 2+ independent files/modules to build in parallel | `builder` — Sonnet; pass `isolation: worktree` at call site for parallel builds, omit for solo |
| Validating a build matches spec, reviewing diffs before reporting done | `reviewer` — Sonnet, read-only |
| Any subtask that would generate >500 tokens of intermediate tool noise | Delegate to keep main context clean |

**Parallel rule**: if N subtasks are independent (no shared file writes, no mid-task dependency on each other), spawn N subagents simultaneously. Wall-clock = slowest one, not sum.

**Worktree rule**: any time 2+ builder agents write files in parallel, use `isolation: worktree`. Skip for read-only agents. Staggering matters at 3+ — launching 3+ `isolation: worktree` agents simultaneously races on `.git/config.lock`; stagger the launches or handle the lock error.

**Fork rule**: when a subagent genuinely needs the full conversation history (e.g. it must know what was already tried), use a fork — it inherits context and reuses the prompt cache.

**Tripwires — catch yourself under-delegating.** The failure mode isn't spawning the wrong agent, it's not spawning at all and grinding through it in the main context. Stop and delegate when:
- About to open a 3rd unfamiliar file to answer one question → spawn `explorer`.
- About to run the same grep/glob a 3rd time to map something → `explorer`.
- About to pull a large doc or API/web fetch into the main context → `researcher`.
- 2+ files to build with no shared writes → spawn `builder`s in parallel, don't grind through them serially.
- Just finished a multi-file build and about to report done → run `reviewer` first.

## When to use agent teams (vs subagents)

Use teams when agents need to coordinate mid-task — not just fan out and collect results. Signs you need a team:
- One agent's output feeds another *before* the parent is done
- Long sustained work where a subagent would exhaust its context (teams have auto-compaction)
- Tasks with 2+ phases requiring back-and-forth between specialists

## Mind what you hand off

A subagent's prompt and any files it reads become part of its own context. Don't route customer PII, credentials, or anything sensitive into scaffolding you haven't reviewed — scope tool access (read-only agents like `explorer`/`reviewer`) rather than defaulting every agent to full file access.

## When NOT to spawn

- Simple edits to 1–3 known files
- Conversational turns, analysis, or planning with no file operations
- Tasks where you'd need to ask someone mid-way — subagents can't pause to ask
- Short tasks (~5 min) — orchestration overhead isn't worth it

## Nesting limit

Hard cap at 5 levels. Don't build agent chains deeper than 3 in practice.

## Escape hatch

None of this is a mandate. If delegating is making a task slower, more confusing, or producing worse output than just doing it yourself — stop, drop back to doing it directly, and move on.
