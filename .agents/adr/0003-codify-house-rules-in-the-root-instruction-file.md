# Codify the house rules in the root instruction file; `/setup-matt-pocock-skills` installs them

This decision is load-bearing. Do not tidy it away.

## Context

The fork already held conventions agents had to follow with nowhere obvious to look before breaking them. "Nothing commits on your behalf" lived in the README's "About this fork" as a description of what the skills do. "No em-dashes" lived as the last paragraph of [`CLAUDE.md`](../../CLAUDE.md) and as invariant 6 of [`LOCALISE.md`](../../.opencode/skills/sync-upstream/LOCALISE.md). British English was never written down at all, so model prose drifted from the human-written code it sat beside. An agent tidying the instruction file, or a sync run localising upstream wording, met these as stray paragraphs with no signal that they were deliberate.

## Decision

The **house rules** (see [GLOSSARY.md](../../GLOSSARY.md)) are codified in a `## House rules` section of the root instruction file, and `/setup-matt-pocock-skills` writes the same section into the root instruction file of every repo it configures. Three rules live there: no state-changing git actions without an explicit ask, **Use British English**, and no em-dashes.

That skill documents that it takes no per-user config: "Preferences belong in your `CLAUDE.md` as plain instructions." The stance bends for exactly these rules. They are fork identity, the same class as "Skills stay local" and "Nothing commits on your behalf", not preferences. Anything beyond them still belongs in the repo's own instruction file, written by its owner.

## Invariants this creates

- **Git runs only on an explicit ask.** State-changing actions (`add`, `commit`, `push`, branch, checkout, merge, rebase, reset, `stash`, `worktree`, `config`) are never assumed. Read-only git is fine. When the user or a skill's documented procedure asks for git operations, that run may perform exactly those, which is why `sync-upstream` may merge with `--no-commit` and `prototype` may commit a throwaway branch.
- **Spelling behaves like punctuation.** British spellings are rewritten in every file a run touches, the way em-dashes are. `CHANGELOG.md` keeps upstream's words as well as its narrative: only the no-em-dash rule rewrites it, punctuation only.
- **Identifiers are flagged, never churned.** Names the run coins go British; names that extend an established family keep that family's spelling; nothing existing is rewritten mid-task. Inconsistencies are flagged at hand-back so they can be fixed in a separate commit.
- **The pointer is the protection.** The `## House rules` section points at this file and forbids removal. Delete that pointer first and this document stops working.

## Considered options

- **Leave each rule to the repo owner's prose.** This is what the skill's "no per-user config" answer said, and it is why the rules drifted. A rule nobody writes down is a rule the next agent tidies away.
- **One ADR per rule.** Rejected: tidying one rule is the same mistake as tidying the other, so one decision record covers both.
- **An ADR with no pointer in the rules.** Rejected: an ADR nothing points at is found after the tidying, not before. The pointer's wording is the whole mechanism.
