---
name: sync-upstream
description: "Sync this fork with its upstream: merge, localise the result against this fork's invariants, and report what needs a decision. Use ONLY when the user asks to sync, update, or pull from upstream."
disable-model-invocation: true
---

# Sync Upstream

Merge `mattpocock/skills` into this fork and **localise** the result: adopt upstream's structure, re-apply this fork's invariants to its wording, and hand the merge back uncommitted. The run ends with a summary and a list of **flags**, never with a commit.

Read [LOCALISE.md](LOCALISE.md) before touching anything. It is what "localised" means, and it decides every rewrite this run makes.

Run git as the user who owns the repository. A session running as someone else sprinkles its own objects through `.git/objects`, and the owner's next commit then fails with `insufficient permission for adding an object to repository database`.

## Steps

1. **Establish the diff.** Add the `upstream` remote if it is missing, fetch it, and work out what is arriving: the commit list, how far ahead and behind each side sits, and every file the merge touches.

   Done when you can name the arriving commits and the file surface they cover.

2. **Merge, never committing.** `git merge --no-commit --no-ff upstream/main`. A merge that lands cleanly is still left uncommitted.

   Done when the index holds the merge and no commit exists. `git merge --abort` unwinds the whole thing if the state is wrong.

3. **Inventory the surface.** Every file the merge touched needs a classification, and the surface is wider than the conflicts: upstream's new files arrive clean and still need localising. Sort each file into **adopt** (upstream's structure wins: a rename, a bucket move, a graduation, a removal), **localise** (its wording runs into an invariant), or **both**.

   Done when nothing is unclassified and each file carries a resolution rule.

4. **Dispatch the mechanical surface to subagents.** Most of a run is mechanical: a rename carried through, a link swapped, a glossary filename updated. Send those out in parallel rather than doing them one at a time, one subagent per group of files that share a resolution shape, and hold the structural and semantic work back for yourself.

   Each brief carries three things: the recipe from [LOCALISE.md](LOCALISE.md), a per-file note stating what the two sides actually disagree about, and the constraint that a subagent edits files only and never touches git state (no `git add`, `git commit`, `git checkout`, `git config`). A brief without the per-file note produces confident guessing.

   Verify what comes back yourself. A subagent reporting clean is a claim, not evidence: re-read its files and run the sweeps.

   Done when every mechanical file is resolved and you have checked the output rather than trusted it.

5. **Resolve the structural and semantic surface yourself.** Renames, removals, and graduations set the shape of the repo, and any hunk where the fork's intent decides the answer is yours to make. Where an invariant and an upstream change cannot both hold, raise a flag instead of picking.

   Done when only flags remain undecided.

6. **Verify the invariants.** Sweep for what must not survive a run: conflict markers, `https://aihero.dev/skills-` links under `docs/`, em-dashes, American spellings, and the tracker and commit wording each invariant names. Then confirm discovery still works with `npx skills@1.5.9 add . --list`.

   Done when every sweep is clean and the discovered skills match what you expect.

7. **Summarise and raise flags.** Report what was adopted, what was localised and how, and the flags. Keep every flag a decision you can make in one line, and keep the list wider than git and trackers: anything the merge sets up for later belongs here.

   Flag at least these cases:

   - An invariant and an upstream change cannot both hold.
   - Upstream introduced something this fork's model has no place for: a hosted tracker, a harness integration, a behaviour that writes to git.
   - Upstream's text is now false for this fork: it claims the plugin ships, names a hosted tracker, or describes behaviour this fork changed.
   - The merge sets up the next one: a rename or removal this fork leans on, or wording that will collide again.
   - Upstream shipped something broken that this fork inherits: prose that no longer parses, a claim that contradicts itself, a stale reference.
   - The release path will surprise you: changesets that bump the version, or changelog text describing a fork that is not this one.

   Done when every arriving change category is accounted for and every flag is a one-line decision.

## Hand back

Leave the merge staged and uncommitted. The summary is the deliverable; the commit is the human's, because a run that commits on their behalf breaks the invariant it exists to protect.
