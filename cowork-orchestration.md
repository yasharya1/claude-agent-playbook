## Agent Orchestration (Cowork)

Delegate to subagents whenever a task is parallelizable — don't process things
one-by-one when they could run concurrently. Subagents in Cowork are isolated,
disposable Claude instances: each gets a clean context window, does its piece,
and reports its result back to you. They can't message each other or
coordinate mid-task — you're always the one relaying between them, which is
different from how Claude Code's "agent teams" work, if you've heard of that.

### When to spawn subagents

| Situation | What to do |
|---|---|
| A list of 10+ similar items to process — accounts, interviews, calls, tickets, contacts | Split across N subagents, each handling a fixed batch |
| Several genuinely independent subtasks (e.g. "pull the Gong summary, draft the battlecard update, check the HubSpot deal status") | One subagent per subtask, spawned together |
| One large document or a single deep analysis (e.g. reading one win-loss interview transcript) | Usually faster as one pass — don't split a single coherent task just to parallelize |
| Anything sequential, where step 2 needs step 1's output | Don't parallelize — run in order |

### Be specific about the split — vague instructions under-delegate

Left to its own judgment, Claude tends to under-split — spawning too few
subagents, each carrying too much work, which defeats the purpose. State the
batch size explicitly:

- Weak: *"summarize these 40 win-loss interviews using subagents"* → often
  becomes 3-4 agents doing 10+ each
- Strong: *"split these 40 interviews into batches of 5 and spawn one subagent
  per batch — calculate the number of subagents automatically"*

Once a ratio works for something you do repeatedly (interview research,
Gong call batches), save it as a Skill so you can trigger it with one command
instead of re-explaining the split every time — the skill-creator skill can
help turn a recurring task into one.

### A prompt you can paste as-is

> I have 40 win-loss interview transcripts in this folder. Split them into
> batches of 5, spawn one subagent per batch, and have each subagent write a
> one-paragraph win/loss theme summary for its batch. Save each subagent's
> output to a distinct file named `batch-1-summary.md`, `batch-2-summary.md`,
> etc. so results don't overwrite each other. When all batches are done,
> compile the summaries into one combined doc.

Swap in your own record type, batch size, and output format — what matters is
the structure (explicit batch size, one subagent per batch, distinct
filenames), not the specific wording.

### Avoid output collisions

If multiple subagents will each produce a file (a report, a doc, a sheet),
give each one an explicit, distinct filename or destination up front. Left to
just "save your result," they can silently overwrite each other if they land
on the same default name.

### Mind the token cost

Each subagent is a full Claude request, not a free clone — splitting a task
40 ways costs roughly 40x a single request's worth of tokens, not free
parallelism. A handful of subagents for a real batch job is normal;
reflexively splitting everything (including tasks that don't need it) adds up
fast and can eat into a monthly usage cap. If you're watching usage, batch
coarser (fewer, larger subagents) rather than finer.

### Tripwires — catch yourself under-delegating

Split the work when you notice:
- You're about to look up or summarize the 4th+ similar record in a row one at a time
- A request contains "each," "every," or "all N" and N is more than a handful
- You're drafting the same type of output (summary, outreach email, brief) repeatedly in sequence

### What this can't do

Subagents report only to you — there's no agent-to-agent messaging and no
real-time coordination between them. If a task genuinely needs agents to talk
to each other mid-task rather than just report back, Cowork can't do that —
that's a Claude Code-only capability. Don't try to work around it — treat it
as a hard limit.
