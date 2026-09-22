## What it does

`wayfinder` takes an effort too big for one agent [session](https://www.aihero.dev/ai-coding-dictionary/session), where the **destination** is nameable but the route is not, and charts it as a local **map** of **decision tickets** under `.scratch/<effort>/`.

It plans, it does not build. Each ticket holds a question whose resolution is a decision, not an implementation slice. When the map clears, wayfinder hands off to [to-spec](./to-spec.md).

## When to reach for it

You invoke this by typing `/wayfinder`, and the agent will not reach for it on its own.

The trigger is narrow: the effort is larger than one session and the route is foggy.

| What you have | What to run |
| --- | --- |
| A feature you can settle in one sitting | [grill-with-docs](./grill-with-docs.md) |
| A multi-session effort with an unclear route | `/wayfinder` |
| A thread where decisions are complete | [to-spec](./to-spec.md) |
| A cleared map | [to-spec](./to-spec.md), then [to-tickets](./to-tickets.md) and [implement](./implement.md) |

## Prerequisites

[setup-matt-pocock-skills](./setup-matt-pocock-skills.md) must have configured the local tracker seed. The map is `.scratch/<effort>/map.md`; child issues are `.scratch/<effort>/issues/NN-<slug>.md`. Read the seed's **Wayfinding operations** section before creating or resolving anything.

## The map, the fog, and the frontier

The map is an index, not a store. It contains:

- **Destination**: what reaching the end looks like.
- **Decisions so far**: one line per completed ticket, pointing to its local file.
- **Not yet specified**: fog that cannot yet be phrased as a sharp question.
- **Out of scope**: work ruled beyond the destination; its ticket is closed with `Status: done`.

Each child issue starts with `Status: ready-for-agent`, `Type: wayfinder:<type>`, and `Blocked by: NN, NN`. Claiming means setting `Status: claimed` before work. Resolving means appending the answer under `## Comments`, setting `Status: done`, and adding a one-line gist to the map. A ticket ruled out of scope is also set to `Status: done`, with its gist going to the map's **Out of scope** section instead. The **frontier** is a scan of local files for non-terminal issues whose blockers are done and which are not claimed.

## The four decision-ticket types

| Type | Mode | Reach for it when |
| --- | --- | --- |
| `grilling` | HITL | The question can be settled by talking it through. |
| `prototype` | HITL | A rough artifact is needed to decide how something should look or behave. |
| `research` | AFK | A fact outside the working directory blocks a decision. |
| `task` | Either | Manual work must happen before a decision can be made. |

Research is the only exception to one ticket per session. Its subagent captures findings on a throwaway `research/<name>` branch, with a context pointer in the local issue file. Commits on that throwaway research branch are allowed.

## Common questions

**How is this different from `/grill-with-docs`?**

Session count. Use [grill-with-docs](./grill-with-docs.md) when the whole decision fits one conversation. Use wayfinder when it does not.

**Where does the map live?**

At `.scratch/<effort>/map.md`, with child issues in `.scratch/<effort>/issues/`. The map is local markdown, not a separate service.

**Can I work several tickets in parallel?**

The local convention assumes one editor. Work one ticket at a time unless you deliberately coordinate separate files and reconcile the map yourself.

**What happens after the map clears?**

Pass the map path to [to-spec](./to-spec.md). It collapses the linked decisions into a buildable spec, after which [to-tickets](./to-tickets.md) creates implementation issues.

**What if the route turns out to be small?**

If the opening breadth-first grill finds no fog, stop and ask whether to skip the map and proceed with [to-spec](./to-spec.md) or [implement](./implement.md).

## It's working if

- The destination is agreed before the first child issue exists.
- Every open child issue reads as a question, not a build instruction.
- The frontier comes from scanning `.scratch/<effort>/issues/` and checking `Blocked by:` and `Status:`.
- A resolved issue has its answer and disclaimer under `## Comments`, `Status: done`, and a line in the map's decisions.
- The fog shrinks as new sharp questions become child issue files.
- A cleared map hands you toward a spec, not implementation.

## Where it fits

`wayfinder` is a situational on-ramp for work too large and unclear for one session. It merges back into the main chain at [to-spec](./to-spec.md). Underneath, [grilling](../productivity/grilling.md) and [domain-modeling](./domain-modeling.md) resolve conversation tickets, [prototype](./prototype.md) resolves concrete design questions, and [research](./research.md) supplies facts in the background. [ask-matt](./ask-matt.md) routes the whole set.
