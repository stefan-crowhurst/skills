## What it does

`setup-matt-pocock-skills` configures one repo for the engineering flow. It records the local `.scratch/` tracker convention, the five triage status strings, and the location and reading rules for domain docs under `docs/agents/`.

It is a prompt-driven skill, not a deterministic script. It explores the repo, shows the proposed markdown, waits for confirmation, and then writes the setup files and the `## Agent skills` block in the existing `CLAUDE.md` or `AGENTS.md`.

## When to reach for it

You invoke this by typing `/setup-matt-pocock-skills`, and the agent will not reach for it on its own.

Reach for it once per repo, before the first use of [triage](./triage.md), [to-spec](./to-spec.md), [to-tickets](./to-tickets.md), or [wayfinder](./wayfinder.md).

## Prerequisites

It writes markdown into the repo for you to commit:

| It writes | Where |
| --- | --- |
| `issue-tracker.md` | `docs/agents/` |
| `domain.md` | `docs/agents/` |
| `triage-labels.md` | `docs/agents/`, when [triage](./triage.md) is installed |
| An `## Agent skills` block | the existing `CLAUDE.md` or `AGENTS.md` |

## The three decisions

The tracker decision is fixed: issues, specs, wayfinder maps, and conversation history live as local markdown under `.scratch/`. The skill does not inspect remotes or ask you to choose a host.

| Decision | What it records | When it asks |
| --- | --- | --- |
| **Local tracker** | `.scratch/<feature-slug>/spec.md` and one issue file per ticket | It confirms the convention, but there is no alternative |
| **Triage status strings** | `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix` | Only when [triage](./triage.md) is installed |
| **Domain docs** | single-context or multi-context layout | Only when it finds monorepo signals |

## Common questions

**Where do the issues and specs go?**

Specs go in `.scratch/<feature-slug>/spec.md`. Implementation issues go in `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, with `Status:`, `Type:`, `Blocked by:`, `## Parent`, and `## Comments` in the local format.

**Does setup create remote labels?**

No. Triage roles are represented by the `Status:` string in each local issue file. The mapping lives in `docs/agents/triage-labels.md`.

**What does setup write to the instruction file?**

It edits the existing `CLAUDE.md` first, otherwise `AGENTS.md`, and updates an existing `## Agent skills` block in place. If neither file exists, it asks which one to create.

**Do I need to run it again?**

Only to restart the setup from scratch. The generated markdown is inspectable and can be edited directly when the repo's vocabulary changes.

## It's working if

- `docs/agents/issue-tracker.md` and `docs/agents/domain.md` exist, plus `triage-labels.md` when [triage](./triage.md) is installed.
- The instruction file contains an `## Agent skills` block pointing at those files.
- The local tracker seed describes `.scratch/` paths, `Status:` strings, and append-only `## Comments`.
- [to-spec](./to-spec.md) can write `spec.md` without asking where it belongs.
- [to-tickets](./to-tickets.md) can write one issue file per ticket without asking which host to use.

## Where it fits

`setup-matt-pocock-skills` is the **run-once setup** for the engineering flow. Its neighbours are [triage](./triage.md), which uses the status vocabulary, [to-spec](./to-spec.md) and [to-tickets](./to-tickets.md), which write `.scratch/` artifacts, and [wayfinder](./wayfinder.md), which uses the local wayfinding operations. [ask-matt](./ask-matt.md) routes the whole set.
