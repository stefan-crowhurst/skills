## What it does

`implement` builds work that has already been decided. You point it at a [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket), a [spec](https://www.aihero.dev/ai-coding-dictionary/spec), or the plan you just agreed in the conversation, and it writes the code, drives [tdd](./tdd.md) at the seams, typechecks as it goes, runs [code-review](./code-review.md) at the end, and leaves the result uncommitted in your working tree for you to review, commit, and push.

It never reopens the plan. There is no interview, no clarifying round, no proposal of a different approach. Whatever was settled upstream is the input, and the skill's whole job is to turn that into working code, left uncommitted for you to review. That is what separates it from typing "build this" at a fresh [agent](https://www.aihero.dev/ai-coding-dictionary/agent), which will happily redesign the work while it builds it.

## When to reach for it

You invoke this by typing `/implement` yourself: the agent won't reach for it on its own. It ships with `disable-model-invocation: true`, so no other skill can call it either. Wherever [ask-matt](./ask-matt.md) or [to-tickets](./to-tickets.md) says "then `/implement` per ticket", that is an instruction to you, not something the agent will do unprompted.

Where the work currently lives decides whether this is the right skill:

| The work is… | Reach for |
| --- | --- |
| A local issue file | `/implement .scratch/feature/issues/02-slug.md`, one issue per [session](https://www.aihero.dev/ai-coding-dictionary/session), [clearing](https://www.aihero.dev/ai-coding-dictionary/clearing) context between issues |
| A spec, not yet split up, and the build spans sessions | [to-tickets](./to-tickets.md) first, then `/implement` per ticket |
| A spec, and the build is small | `/implement` directly against the spec |
| Only in the conversation you just had, and it's still small | `/implement` right there, in the same window |
| Not written down anywhere yet | [grill-with-docs](./grill-with-docs.md), or [grill-me](../productivity/grill-me.md) if there's no codebase |
| One concrete behaviour you want test-first, with no spec | [tdd](./tdd.md) directly |
| Already built, and you want it checked | [code-review](./code-review.md) directly |

The same-session case is worth naming because the skill's own first line doesn't cover it. `SKILL.md` says "the spec or tickets", which nudges the [model](https://www.aihero.dev/ai-coding-dictionary/model) to go hunting for a file that doesn't exist. If the plan lives only in the thread, say so when you invoke it.

## Prerequisites

`implement` leaves its work uncommitted in the working tree. It does not create a branch, commit, or push. Check you are on the branch you want the work to land on, and start from a clean tree so the run's changes stand alone.

If the issues came from [to-tickets](./to-tickets.md), they use the local `.scratch/` layout configured by [setup-matt-pocock-skills](./setup-matt-pocock-skills.md). `code-review` resolves the originating spec from `.scratch/` at close-out.

## What one run does

A run is five beats, in order:

1. Read the ticket or spec and work out the seams.
2. Drive [tdd](./tdd.md) at the pre-agreed seams, one red-green slice at a time.
3. Typecheck often, run single test files as it goes.
4. Run the full test suite once, at the end.
5. Run [code-review](./code-review.md) on the uncommitted working tree, then hand back for you to review, commit, and push.

One run covers one ticket. The tickets [to-tickets](./to-tickets.md) produces are tracer-bullet vertical slices sized to fit a single fresh [context window](https://www.aihero.dev/ai-coding-dictionary/context-window), so the intended rhythm is: clear context, implement one ticket, review and commit it yourself, clear again. Each ticket is self-contained, which is what makes the previous ticket's context disposable.

## Pre-agreed seams

The idea the skill runs on is the **seam**: the public boundary you observe behaviour at, without reaching inside. Tests live at seams. Working at a seam agreed before any code is written is what keeps the tests durable, because the implementation underneath can be rewritten without the tests moving.

The word "pre-agreed" is doing real work, and it is also the skill's weakest joint. Nothing inside `implement` agrees the seams. `tdd` is the skill that asks, and it refuses to write a test at an unconfirmed seam. So in practice the agreement happens either upstream in the spec, or in the first exchange of the run. If it happens nowhere, the precondition never fires and the run quietly becomes "just write the code". Naming the seams in the spec is what stops that.

## Common questions

**It finished, but my ticket is still open and the acceptance criteria are still unchecked.**

Correct, and expected. `implement` has no completion step. It ends at the hand-back, with the work sitting uncommitted, and does not update the local issue file. It also does not act on the findings `code-review` produced, and does not tick the `- [ ]` boxes on the originating issue. Set the issue to `Status: done`, append the resolution under `## Comments`, and reconcile the criteria yourself. This bites hardest on a dependency chain, because `to-tickets` defines the frontier as issues whose blockers are done. If nothing gets resolved, nothing becomes visibly unblocked.

**Can I point it at all my tickets at once, or run several in parallel?**

Not with `/implement`: one invocation, one ticket. For a whole spec in one run, use [implement-spec](./implement-spec.md), which fans the tickets out to [subagents](https://www.aihero.dev/ai-coding-dictionary/subagent) across the ready frontier in the shared working tree, and leaves everything uncommitted at the end for you to review, commit, and push. Running several `/implement` sessions side by side in one checkout is worse than unsupported: the sessions share one working directory, one index, and one HEAD, so their edits mix and neither hand-back is clean. Back when `implement` committed its own work, field reports described a `git commit --amend` in one session landing on another session's commit, a stash vanishing from `refs/stash`, and commits landing on the wrong branch, all in a single afternoon across three issues. Git worktrees are the community workaround, and note that `refs/stash` is shared across worktrees too, so worktrees alone do not fix the stash case.

**Does it commit anything, or open a pull request?**

No. Nothing is committed at all. `implement` leaves the work uncommitted in the working tree and hands back for you to review, commit, and push. When you later push the work up as a pull request, [pr](./pr.md) shapes its body.

**`code-review` says it cannot see my changes.**

That was the old bug, and it is fixed. `code-review` now detects uncommitted work and reviews it with `git diff <fixed-point>`, which covers staged and unstaged tracked changes, plus the contents of any untracked files it enumerates. So the hand-back flow reviews exactly what you are about to commit, with no interim commit needed. The committed-diff mode (`git diff <fixed-point>...HEAD`) still applies when the work has been committed.

Separately, some people deliberately do not want the review inside the run at all, because an agent reviewing the code it just wrote is biased toward its own solution. Running [code-review](./code-review.md) in a fresh session against a fixed point is a legitimate alternative, and is the same reason that skill runs its two axes in separate sub-agents.

**One ticket burned 150k tokens. Am I using it wrong?**

Probably the ticket is too big rather than the skill being misused. A run does codebase exploration, a red-green loop per seam, a full suite, and a review, so a non-trivial ticket exceeding 100k [tokens](https://www.aihero.dev/ai-coding-dictionary/token) is normal rather than a sign something broke. The lever is upstream: right-size the tickets in [to-tickets](./to-tickets.md) so each fits one fresh window. If a single ticket keeps blowing out, split it rather than raising the [effort](https://www.aihero.dev/ai-coding-dictionary/effort) level.

**`/implement #2` in a fresh session worked on something completely unrelated.**

`02` must be resolved against the relevant `.scratch/<feature-slug>/issues/` directory, not against an arbitrary numbered list. Pass the full local path and ask it to confirm the title before it begins.

## It's working if

- The session opens by reading the ticket or spec and restating what it will build, rather than asking you what to build.
- You can see an actual `/tdd` invocation in the trace, not just tests appearing in the diff.
- Typechecks and single test files run repeatedly during the run, and the full suite runs once near the end.
- The run reaches a clean hand-back without you prompting it to carry on: the work sits uncommitted in your working tree, and the code review is done.
- The diff is one ticket's worth of change: a vertical slice through every layer, not several tickets swept together.

## Where it fits

`implement` is the build step of the main chain:

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

Its neighbours are [to-tickets](./to-tickets.md), which produces the tickets it consumes and declares the blocking edges that decide their order; [tdd](./tdd.md), which it drives internally at each seam; and [code-review](./code-review.md), which it runs on the uncommitted working tree before handing back. It sits downstream of the planning skills and trusts them. It does not re-validate the shape of what it was handed, so a badly-structured map or a horizontally-layered ticket gets built as written.

That trust is why [wayfinder](./wayfinder.md) merges onto the chain at [to-spec](./to-spec.md) rather than looping its map straight into `implement`. Go straight to `implement` from a map only when the effort turned out genuinely small.

[ask-matt](./ask-matt.md) is the router over the whole set when you are not sure which flow you are in.
