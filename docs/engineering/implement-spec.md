## What it does

`implement-spec` takes a [spec](https://www.aihero.dev/ai-coding-dictionary/spec) and its [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) and lands the whole thing in one run. The orchestrating [agent](https://www.aihero.dev/ai-coding-dictionary/agent) hands each ticket to an implementer [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) working in the shared working tree, runs [code-review](./code-review.md) over the result, and leaves everything uncommitted for you to review, commit, and push.

It reads the tickets as a **task graph**, not a list. Blocking edges decide what can start, so at any moment there is a **frontier** of tickets whose blockers have all landed, and every ticket on the frontier runs at once. That is the difference from working the tickets one by one: the graph's shape, not its order in the tracker, sets the pace.

## When to reach for it

You invoke this by typing `/implement-spec`, and the agent won't reach for it on its own.

| Your situation | Reach for |
| --- | --- |
| A spec, split into tickets with blocking edges, that you want landed in one run | `/implement-spec` |
| One ticket at a time, in your own [context window](https://www.aihero.dev/ai-coding-dictionary/context-window), [clearing](https://www.aihero.dev/ai-coding-dictionary/clearing) between tickets | [implement](./implement.md) |
| A spec that isn't split into tickets yet | [to-tickets](./to-tickets.md) first |
| A small piece of work with no real graph to it | [implement](./implement.md) directly |

## Prerequisites

- **The local tracker.** The skill reads the tickets from the `.scratch/` layout [setup-matt-pocock-skills](./setup-matt-pocock-skills.md) configured. If none has been configured, it stops and tells you to run that first rather than guessing.
- **Tickets with blocking edges**, as [to-tickets](./to-tickets.md) writes them. Without edges the graph is flat and every ticket starts at once.
- **A [harness](https://www.aihero.dev/ai-coding-dictionary/harness) that runs subagents in the background.** The concurrency is the point; a harness that runs subagents one at a time gets a slower `implement-spec`. Because every implementer shares one working tree, never run two at once when their edits could overlap.

## The shared working tree

Everything lands in one working tree, uncommitted. Each implementer:

1. builds its ticket with [tdd](./tdd.md), red-green one slice at a time,
2. reports what it changed, so the orchestrator can verify it against the ticket.

Nothing is committed, pushed, branched, or turned into a pull request along the way. At the end the run hands the whole working tree back to you to review, commit, and push.

Implementers talk to the orchestrator through [context pointers](https://www.aihero.dev/ai-coding-dictionary/context-pointer) (the spec, the ticket, shared exploration notes, the working tree) rather than pasted summaries, which keeps each subagent's prompt small and the orchestrator's window free for the graph.

## Common questions

**How is this different from running `/implement` on each ticket myself?**

This is the question the skill exists to answer. Before it shipped, people kept building their own versions, and one user described the itch exactly: they wanted "subagents implement the tickets" instead of having "to individually create new session and tell them to implement a ticket one by one, when a spec may contain over 5 tickets." With `implement` you are the dispatcher: one [session](https://www.aihero.dev/ai-coding-dictionary/session) per ticket, clearing in between, and keeping track yourself of which tickets are unblocked. `implement-spec` hands that job to one orchestrating session. The price is that you no longer read each ticket's work as it lands; you review the uncommitted working tree at the end. To start a run, clear the context and type `/implement-spec` with a pointer to the spec. For a small change with no real graph, skip it and use `implement` directly.

**Does it commit anything, or open a pull request?**

No. Nothing is committed, pushed, branched, or turned into a pull request. The run ends with the whole spec implemented in the working tree and handed back for you to review, commit, and push. That is deliberate: in this fork no skill commits on your behalf, so the review and the commit stay yours.

**Its review and fix loop ran for hours, or kept "fixing" tickets that hadn't been built yet.**

Both come from `code-review` running outside the one slot the skill gives it. It compares the code against the whole spec, so it only makes sense once every ticket has landed; run it mid-run and every unbuilt ticket reads as a failure, the agent sets about building it, and that triggers another review. At the end, the skill runs `code-review` once and sends every finding to one fix subagent, but it doesn't yet say when to stop after that fix. One user reported a five-ticket feature where "the review and fix loop took roughly four hours". If you see a second broad review start, tell it to run focused checks for the fixed findings and stop. Expect that first review to find real problems: the run's output is a draft that the review finishes, not something to ship on its own.

**Does it drive tdd like implement does?**

It does now, though it didn't at first. Users running the in-progress version noticed that "the implementer subagents don't inherit the /tdd directive", so red-green dropped out the moment they scaled up from one ticket to a whole spec. Each implementer now builds its ticket with `tdd`. There is still no step where seams get agreed interactively, as there is in an `implement` session, so name the seams in the spec or the tickets if you want them pinned.

**Two implementers running in parallel collided on the same file, or picked different names for the same thing.**

Parallel implementers share one working tree, so collisions land immediately rather than waiting to be merged. A blocking edge written from ticket text is a guess about which files each ticket will touch, and two tickets on "different parts of the codebase" still share a message catalogue, a config registry, or a type. Each implementer sees only its own ticket and the shared notes, never the other's work in progress, so one user's web and mobile tickets added the same string as `blockedSince` and `blockedOn`. When two frontier tickets touch one shared surface, either add a blocking edge between them so they run one after the other, or have the exploration notes fix the exact names each ticket adds.

**Blocked tickets never start, even after their blocker is done.**

The frontier is read from the `Blocked by:` edges and the `Status:` lines in the local issue files under `.scratch/`, so it is only as fresh as those files. If a ticket should be free to start and is not, check that its blockers carry `Status: done` in their issue files; the orchestrator can also be told to compute the frontier from what it has already verified this run rather than re-reading the files.

**Does this replace Sandcastle or an AFK script?**

No. People ask because the skills now reach into implementation: "is Sandcastle still relevant? Your skills now seem to be able to handle implementation as well." `implement-spec` puts an agent in charge of orchestration inside one harness session, which needs no infrastructure and lets you watch and steer. For work that is truly [AFK](https://www.aihero.dev/ai-coding-dictionary/afk), a deterministic loop ([Sandcastle](https://github.com/mattpocock/sandcastle), a shell script, a CI job) is faster, cheaper, and more reliable, because no part of the orchestration can wander off.

**A ticket's key test was skipped, and it reported green.**

Scoped to one ticket, an implementer can quietly skip checks that depend on something outside its brief. Tests that read gitignored fixtures, local databases, or credentials can skip themselves silently. For a ticket whose verification depends on untracked material, run that check yourself before accepting the run.

## It's working if

- Several implementers are running at once whenever the graph allows, not one after another.
- A ticket starts as soon as its last blocker is done, not when the whole run ends.
- Every ticket's trace shows `tdd` running, with a failing test before the code.
- Nothing has been committed, branched, or pushed along the way.
- The run ends with every ticket resolved and the work sitting uncommitted for your review.

## Where it fits

`implement-spec` is the build step of the main chain, as the parallel alternative to running [implement](./implement.md) once per ticket:

```txt
grill-with-docs → to-spec → to-tickets → implement-spec → retro
```

Its neighbours are [to-tickets](./to-tickets.md), which declares the blocking edges it reads as a task graph, and [code-review](./code-review.md), which it runs over the working tree before closing out. [ask-matt](./ask-matt.md) is the router over the whole set when you are not sure which flow you are in.
