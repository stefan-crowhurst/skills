# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files in `.scratch/`. This is the only tracker convention used by the engineering skills.

## Layout

- One effort or feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md` and carries `Status: ready-for-agent` when it is ready to hand off.
- Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`, never a combined tickets file.
- Every issue file starts with these header lines:

  ```markdown
  # <NN>: <title>
  Status: <status>
  Type: <type>
  Blocked by: <comma-separated NN list or None>
  ```

- Every issue file has `## Parent` and `## Comments` sections. Conversation history and all later notes append under `## Comments`.
- `Type:` is `implementation`, `bug`, or `enhancement` for ordinary issues. Wayfinder child files use `wayfinder:research`, `wayfinder:prototype`, `wayfinder:grilling`, or `wayfinder:task`.
- `Status:` carries the triage state or operational status from `triage-labels.md`: `claimed` while a wayfinder ticket is being worked, `done` once it is resolved or ruled out of scope.

## When a skill says "publish to the issue tracker"

Create the needed directory under `.scratch/<feature-slug>/` and write the requested spec or issue file using this layout. Do not create a remote issue or a second tracker representation.

## When a skill says "fetch the relevant ticket"

Read the referenced local path. If the user gives only `NN`, resolve it by scanning the relevant `.scratch/*/issues/` directory and confirm the title before acting. Read the full file, including `## Comments`.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a file with one child issue file per decision ticket.

- **Map**: `.scratch/<effort>/map.md`, containing `## Destination`, `## Notes`, `## Decisions so far`, `## Not yet specified`, and `## Out of scope`.
- **Child ticket**: `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01`, with the question in the body, `Type: wayfinder:<type>`, and `Status: ready-for-agent` until claimed.
- **Blocking**: a `Blocked by: NN, NN` header line. A ticket is unblocked when every listed file has `Status: done`.
- **Frontier**: scan `.scratch/<effort>/issues/` for files with a non-terminal status, no unfinished blockers, and no `Status: claimed`; the first by number wins.
- **Claim**: set `Status: claimed` and save before any work.
- **Resolve**: append the answer and the AI disclaimer under `## Comments`, set `Status: done`, then append a one-line gist and the child path to `## Decisions so far` in `map.md`.
