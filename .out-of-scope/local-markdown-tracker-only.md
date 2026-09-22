# The tracker is local markdown only

This repo tracks work in local markdown under `.scratch/`. Requests to add or restore hosted issue-tracker backends (GitHub Issues, GitLab, Linear, Jira, and similar) are out of scope.

## Why this is out of scope

A hosted backend hard-codes a remote shape into the skills: a CLI or API surface (commands, flags, authentication, output parsing) plus a network dependency the rest of the flow then assumes. Each backend is permanent maintenance surface, because it has to keep working as the tool's CLI and API evolve, and it has to keep being tested against `/to-spec`, `/to-tickets`, `/triage`, `/wayfinder`, and friends.

The local markdown convention carries the same information with none of that:

- `Status:` lines for triage state and operational status
- `Blocked by:` lines for blocking edges
- `## Comments` for conversation history
- `.scratch/<feature-slug>/` for the layout

A single user can read and edit all of it with no account, no token, and no network, which is the workflow this fork is built around. The reasons to want a hosted tracker (team collaboration, an existing organisational tool) describe a different workflow; the upstream project is the right home for those, not a backend bolted back onto these skills.

## Prior requests

- The upstream policy this file replaces, "Issue tracker integrations are limited to mainstream tools", which kept GitHub, GitLab, and Backlog.md as first-class options.
- #99 (upstream): "Add dex as an issue tracker backend".
