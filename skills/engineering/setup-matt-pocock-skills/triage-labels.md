# Triage Status Strings

The skills speak in terms of five canonical triage roles. This file maps each role to the `Status:` string written in a local issue file.

| Role | `Status:` string | Meaning |
| --- | --- | --- |
| `needs-triage` | `needs-triage` | Maintainer needs to evaluate this issue |
| `needs-info` | `needs-info` | Waiting on the reporter for more information |
| `ready-for-agent` | `ready-for-agent` | Fully specified, ready for an AFK agent |
| `ready-for-human` | `ready-for-human` | Requires human implementation |
| `wontfix` | `wontfix` | Will not be actioned |

When a skill mentions a role, set the issue file's `Status:` line to the corresponding string. Edit the strings if this repo adopts a different local vocabulary.

## Operational statuses

Wayfinder tickets use two statuses beyond the five triage roles:

| `Status:` string | Meaning |
| --- | --- |
| `claimed` | A session has claimed the ticket and is working it |
| `done` | The ticket is resolved: the answer is appended under `## Comments`, or it was ruled out of scope |

These are not triage roles. `triage` leaves them alone, and the edit-the-strings note above applies only to the five triage strings.
