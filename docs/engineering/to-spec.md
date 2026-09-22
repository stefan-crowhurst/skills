## What it does

`to-spec` turns the conversation you have just had into a **[spec](https://www.aihero.dev/ai-coding-dictionary/spec)** and writes it to `.scratch/<feature-slug>/spec.md`.

It does not interview you. By the time you reach for it the deciding is already done, so it synthesises what is known from the thread, codebase, `CONTEXT.md`, and ADRs. The spec records decisions already made rather than opening a fresh round of questions.

## When to reach for it

You invoke this by typing `/to-spec`, and the agent will not reach for it on its own.

Reach for it when the build is too big for one agent [session](https://www.aihero.dev/ai-coding-dictionary/session) and has to survive being split across several.

| Where you are | What to run |
| --- | --- |
| You have not decided anything yet | [grill-with-docs](./grill-with-docs.md) first |
| Decided, and the work fits one [context window](https://aihero.dev/ai-coding-dictionary/context-window) | [implement](./implement.md), skip the spec |
| Decided, and the work spans several sessions | `/to-spec`, then [to-tickets](./to-tickets.md) |
| A [wayfinder](./wayfinder.md) map has cleared | `/to-spec` with the map path |

## Prerequisites

[setup-matt-pocock-skills](./setup-matt-pocock-skills.md) must have configured the local `.scratch/` layout and domain docs. The resulting spec carries `Status: ready-for-agent` and needs no separate triage pass.

## The spec is a decision record

The spec exists because context windows end. Everything settled while [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) is in one conversation that is about to be cleared. The spec is what survives that.

It does not validate or decide anything. It captures what was decided in the project's own vocabulary. Anything it asserts that you never actually said is a defect.

## Seams before prose

Before writing, `to-spec` sketches the **seams** where the feature will be tested and checks them with you. It prefers existing seams, takes the highest seam it can, and aims for one seam across the change.

## Common questions

**Where does the spec go?**

In `.scratch/<feature-slug>/spec.md`, with `Status: ready-for-agent` near the top. [to-tickets](./to-tickets.md) reads that local path directly.

**Why does it not ask me more questions?**

Synthesis is the defining constraint. If a decision is still open, return to [grill-with-docs](./grill-with-docs.md) instead of asking `to-spec` to invent an answer.

**Why does the spec say `ready-for-agent`?**

That status means the document is complete enough to hand off. It does not mean an agent should implement the whole feature. [to-tickets](./to-tickets.md) turns the spec into smaller issues.

**What do I feed it after a wayfinder map?**

Pass the map path, not an individual decision ticket. [wayfinder](./wayfinder.md) produces decisions, and `to-spec` collapses them into one buildable document.

## It's working if

- It starts writing instead of opening a fresh interview.
- It proposes the fewest useful test seams and gets your agreement before prose.
- It comes back in the project's nouns, not generic product language.
- Every decision is one you can remember making.
- The spec exists at the expected `.scratch/` path with `Status: ready-for-agent`.

## Where it fits

`to-spec` is the multi-session step in the main build chain:

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

Its neighbours upstream are [grill-with-docs](./grill-with-docs.md), which does the deciding, and [wayfinder](./wayfinder.md), whose cleared map merges here. Downstream, [to-tickets](./to-tickets.md) cuts the spec into local tracer-bullet issues. When you are unsure which flow fits, [ask-matt](./ask-matt.md) routes you.
