## What it does

`to-tickets` takes a plan, a [spec](https://www.aihero.dev/ai-coding-dictionary/spec), or the conversation you are in, and breaks it into local issue files under `.scratch/<feature-slug>/issues/`. Each file declares its **blocking edges** in a `Blocked by:` header.

Every issue is a **tracer bullet**: a narrow but complete path through every layer of the change that can be verified on its own. That is the constraint that makes it different from cutting one layer at a time and integrating at the end.

## When to reach for it

You invoke this by typing `/to-tickets`. The agent will not reach for it on its own.

| Where you are | What to run |
| --- | --- |
| You have a spec and the build spans several sessions | `/to-tickets`, with the spec path if needed |
| The plan is only in the conversation | `/to-tickets` reads the thread directly |
| The whole change fits in one context window | [implement](./implement.md), skip the issues |
| Nothing is decided yet | [grill-with-docs](./grill-with-docs.md), then [to-spec](./to-spec.md) |
| A [wayfinder](./wayfinder.md) map has cleared | [to-spec](./to-spec.md) first, then `/to-tickets` |

Issues produced by `to-tickets` are agent-ready by construction. Do not run [triage](./triage.md) over them.

## Prerequisites

[setup-matt-pocock-skills](./setup-matt-pocock-skills.md) must have configured the local `.scratch/` layout. Each issue uses the canonical header:

```markdown
# 01: Short title
Status: ready-for-agent
Type: implementation
Blocked by: None
```

The body includes `## Parent`, `## What to build`, `## Acceptance criteria`, and `## Comments`.

## Tracer bullets, not layers

A horizontal slice ships one layer and leaves the system incomplete. A vertical slice ships one thin path through all layers at once, so it can be verified alone and owns everything it grades.

Before anything is written, `to-tickets` looks for prefactoring and orders it first. It then presents a numbered breakdown and quizzes you on granularity, blocking edges, and possible merges. Nothing is published until you approve it.

## Blocking edges

The edges are plain text in each local issue file:

| Artifact | Where the edges live | How you work them |
| --- | --- | --- |
| Spec | `.scratch/<feature-slug>/spec.md` | Read it as the parent |
| Issue | `Blocked by: 01, 02` | Scan local files, then work the first unblocked ticket |

Number issues in dependency order, blockers first. A ticket is on the frontier when every number in its `Blocked by:` line has `Status: done`.

## The wide-refactor exception

A **wide refactor** has a blast radius across the whole codebase, so no vertical slice can land green. Sequence it as **expand-contract**: add the new form, migrate call sites in batches with local checks green, then delete the old form after every batch is done. If batches cannot stay green independently, use an integration branch and make every batch block a final integrate-and-verify issue.

## Common questions

**Where do the issues go?**

One file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`. Never create a combined `tickets.md` file.

**Why is `Blocked by:` text instead of a relationship in a board?**

The local markdown files are the source of truth. The comma-separated numbers are easy to scan and keep the dependency graph in the same file as the issue.

**The breakdown is too fine. What should I do?**

Use the quiz step to merge issues. If the whole change fits one context window, skip this skill and use [implement](./implement.md).

**How do I run the published issues?**

Open one fresh session per issue, starting with the first file whose blockers are done. Claim it with `Status: claimed`, then set it to `Status: done` and append the resolution when the work is complete.

## It's working if

- Every issue answers what can be demonstrated or verified when it is done.
- The numbered list comes back with a `Blocked by:` line before publication.
- The first issue has no blockers and can start immediately.
- Each issue is a complete path through the relevant layers, not a layer assignment.
- Prefactoring appears before the feature issues.

## Where it fits

`to-tickets` is a step in the main build chain:

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

Upstream is [to-spec](./to-spec.md), which hands it a settled local spec. Downstream is [implement](./implement.md), which builds one issue per fresh session. When you are unsure which flow fits, [ask-matt](./ask-matt.md) routes you.
