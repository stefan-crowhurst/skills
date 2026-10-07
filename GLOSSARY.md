# Matt Pocock Skills

A collection of agent skills (slash commands and behaviors) loaded by Claude Code. Skills are organized into buckets and consumed by per-repo configuration emitted by `/setup-matt-pocock-skills`.

## Language

**Issue tracker**:
The local markdown convention under `.scratch/` where this repo stores specs, issues, maps, and conversation history. Skills like `to-tickets`, `to-spec`, and `triage` read from and write to these files.
_Avoid_: backlog manager, backlog backend, issue host

**Issue**:
A single tracked unit of work in a `.scratch/<feature-slug>/issues/<NN>-<slug>.md` file: a bug, task, or slice produced by `to-tickets`.
_Avoid_: ticket (use **Issue**; the one exception is a **Decision ticket**, see below)

**Decision ticket**:
A `wayfinder` unit: a child local issue file of a `.scratch/<effort>/map.md` holding a *question* whose resolution is a decision, not a slice of a build to execute. The **decision** qualifier is what keeps it distinct from an implementation issue.

**Triage role**:
A canonical state-machine string written to an **Issue**'s `Status:` line during triage (e.g. `needs-triage`, `ready-for-agent`). Each role maps to a local status string via `docs/agents/triage-labels.md`.

**House rules**:
The conventions governing how agents write and act in this repo: Use British English, no em-dashes, and no state-changing git actions without an explicit ask.
_Avoid_: preferences, style guide, per-user config

## Relationships

- An **Issue tracker** holds many **Issues**
- An **Issue** carries one **Triage role** at a time
- A **Decision ticket** is an **Issue** (a child of a `wayfinder:map`)

## Flagged ambiguities

- "backlog" was previously used to mean both the *tool* hosting issues and the *body of work* inside it. Resolved: the tool is the **Issue tracker**; "backlog" is no longer used as a domain term.
- "backlog backend" / "backlog manager". Resolved: collapsed into **Issue tracker**.
