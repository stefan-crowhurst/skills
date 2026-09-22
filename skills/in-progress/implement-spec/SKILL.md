---
name: implement-spec
description: "Implement a specification in code."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The goal is the entire spec implemented in the working tree, left uncommitted for the user to review.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. This means there is always a **frontier** of tickets which are ready to be grabbed.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and the working tree. Don't duplicate information already available via pointers.

**Implementer subagents** should run in the background where possible for **maximum concurrency**, but they share one working tree: never run two concurrently if their edits could overlap.

## Steps

1. Read the spec and tickets. Read enough to understand the task graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future subagents. This lets **implementer subagents** focus on implementation rather than exploration.

3. Do **not** create branches, worktrees, commits, or PRs. All work lands in the shared working tree.

4. Use **implementer subagents** to implement the tickets in the current **frontier**, working directly in the working tree.

5. As each **implementer subagent** completes, verify its work against the ticket it implements.

6. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets. This allows for maximum concurrency.

7. Once all tickets are complete, run /code-review on the working tree. Fix all issues raised by the code review in a single **implementer subagent**.

8. Hand back: report what was implemented and the code review outcome, and leave all changes uncommitted for the user to review, commit, and push.
