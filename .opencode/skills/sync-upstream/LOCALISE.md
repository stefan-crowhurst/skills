# Localise

**Localise** is what this fork does to anything that arrives from upstream. The fork's whole identity lives in this file, so it is the checklist a sync run works through. Read it before resolving anything.

## The invariants

Every one of these still holds when the run ends. Incoming work that breaks one gets rewritten, not adopted.

1. **The tracker is local markdown.** Issues, specs, maps, and conversation history live under `.scratch/`, and no hosted tracker appears anywhere in the skills. Wording that describes GitHub Issues, GitLab, Linear, a `gh` or `glab` call, or a choice between trackers becomes the local convention instead: `Status:` lines for triage state, `Blocked by:` headers for edges, `.scratch/<feature-slug>/` for the layout.

2. **Nothing commits on your behalf.** A skill leaves its work uncommitted in the working tree and hands back. Wording that has a skill commit, push, open a branch or worktree, or open a pull request becomes hand-back wording: the work lands in the shared working tree, and the review, the commit, and the push stay yours.

3. **Docs cross-links are relative.** A link from one docs page to another is `./<name>.md` in the same bucket or `../<other-bucket>/<name>.md` in the other. An `https://aihero.dev/skills-<name>` link becomes relative. The test is where the target lives, not how the link is spelled: anything that exists in this repo (a docs page, a skill file such as `DEEPENING.md` or `template.sh`) becomes a relative path, even when upstream spells it as a `github.com/.../blob/...` URL. Truly external references stay absolute: dictionary links (`https://www.aihero.dev/ai-coding-dictionary/...`) and anything that lives only upstream. Hosted-tracker citations (issue and PR links) are dropped under invariant 1 rather than relinked.

4. **Upstream's structure is adopted as it stands.** Renames, bucket moves, graduations, and outright removals are upstream's call, so take them: what the glossary file is called, which bucket a skill sits in, which skills still exist. Following structure is what keeps the next merge small.

5. **Upstream's identity stays.** Its prose, its branding, its voice. The fork changes behaviour, not identity, so a sentence of the original is rewritten only where an invariant above forces it.

6. **No em-dashes, and British spellings throughout.** In every file the run touches. Fix an American spelling when you find one in the same run; do not rename identifiers or established names (`domain-modeling`, `colorFor` beside `getColor`).

7. **The fork's setup writes stay.** `setup-matt-pocock-skills` writes two blocks upstream's version does not: the `## House rules` block in every repo it configures, and a `## Running commands` note when that repo pins its toolchain with Nix (flakes, nix-shell, or devenv), naming the entry command for the flavour present. The note is detection, not choice: no Nix signals, no note. Wording that describes setup's writes without these blocks is incomplete rather than authoritative, and a whole-page rewrite of the skill puts them back after taking upstream's page as the base.

8. **Remote operations have a local alternative.** Upstream often keeps a local markdown mode but writes the hosted shape into the surrounding text. Take the local option where it exists, and build the fork's own where it does not. See [Local-only alternatives](#local-only-alternatives).

## Localising a hunk

Where both sides edited the same passage, work from upstream's side and re-apply the invariants:

1. Take upstream's text as the base. It carries the new content and the structural change.
2. Rewrite any tracker or commit wording to the invariant form above.
3. Swap `https://aihero.dev/skills-*` links and in-repo `github.com/.../blob/...` links for relative ones, and call the glossary file whatever upstream currently calls it.
4. Keep the rest, including upstream's new sentences and its dictionary links.

A **clean arrival** is localised the same way. Files that merge without a conflict still carry upstream's wording, and they are the easiest thing in a run to forget.

A **whole-page rewrite** needs a different move. When upstream rewrites a page wholesale (a repo-wide style pass, say), do not resolve it hunk by hunk. Take upstream's page in full as the new base and re-apply the invariants to the whole page, using the fork's pre-merge version only as the glossary of established localised phrasings. Same destination as a hunk localisation, without the archaeology.

## Local-only alternatives

Upstream often keeps a local markdown mode but writes the hosted shape into the surrounding text. Resolve `setup-matt-pocock-skills`, `to-tickets`, `triage`, and `wayfinder` to their local mode: `.scratch/` files, `Status:` lines, `Blocked by:` headers, and local research notes. Drop the label, child-issue, native-blocker, assignee, and PR branches rather than translating them.

Some upstream changes have no local mode. `implement-spec` is the clear case: worktrees, an integration branch, merges, and a draft PR are the remote shape. Keep the fork's shared working tree and hand-back instead. `implement`, `code-review`, `to-spec`, and `research` say "the tracker" generically; resolve that to `.scratch/`.

If a new remote-only operation appears, do not invent the fork's alternative alone. Describe what upstream assumes, propose the local shape, and agree it with the user before rewriting the skill. Once agreed, build it from the invariants and record the divergence in the flag list so the next sync expects the conflict. The shared working tree and hand-back in `implement-spec` are an agreed alternative, not a precedent for deciding new ones unilaterally.

## Upstream history

ADRs, changesets, release notes, and `CHANGELOG.md` describe the upstream project, including infrastructure this fork does not publish or use: the Claude Code plugin, managed marketplaces, hosted trackers, and release workflows. Keep the history, because rewriting it makes the next merge harder, but add a short fork note where a reader of this repo would otherwise take the claim as true. `CHANGELOG.md` is the exception: keep its words and change only em-dashes.

Passage-by-passage rewriting rather than word swapping is a writing job, and the house rules govern it: `Call the Skill tool with "writing-for-agents"`.

## When to flag rather than resolve

An invariant and an upstream change can both be honoured only if one of them bends. Pick neither. Leave that work undone and raise it in the run's flag list, so the choice stays yours.
