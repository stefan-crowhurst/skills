# Contributing

Every issue is judged against the bar in [`SCOPE.md`](./SCOPE.md). Read it first.

## Where each request goes

| You have | Go to |
| --- | --- |
| A bug in a skill | A local issue file under `.scratch/<feature-slug>/issues/`, with the repro first |
| A change to an existing skill | The same, describing the change |
| A question, or "how do I...?" | Ask in the session; no issue needed |
| An idea for a new skill, or a skill to donate | Read [`.out-of-scope/new-skills.md`](./.out-of-scope/new-skills.md) first; new skills are not accepted |
| Support for another harness or issue tracker | Read [`.out-of-scope/local-markdown-tracker-only.md`](./.out-of-scope/local-markdown-tracker-only.md); hosted trackers are out of scope |

There is no pull-request flow. Describe the fix in a local issue, and hand back uncommitted work in the working tree for the maintainer to review, commit, and push.

## Issues written with AI

Agents are welcome to help, but you post the issue, so you own it:

- **One problem per issue.** Several findings from one session are several issues, or none.
- **Short.** If it does not fit on one screen, cut it.
- **Repro first.** The steps that show the problem come before any analysis.
- **No pasted specs.** No full proposals, plans, or rewritten `SKILL.md` files. Describe the problem; the fix is ours to design.
- **Not your project's issues.** If your agent files the specs or tickets for your own project here, it has the wrong repo. Run `/setup-matt-pocock-skills` in your project.
