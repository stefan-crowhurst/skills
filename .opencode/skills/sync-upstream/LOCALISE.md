# Localise

**Localise** is what this fork does to anything that arrives from upstream. The fork's whole identity lives in this file, so it is the checklist a sync run works through. Read it before resolving anything.

## The invariants

Every one of these still holds when the run ends. Incoming work that breaks one gets rewritten, not adopted.

1. **The tracker is local markdown.** Issues, specs, maps, and conversation history live under `.scratch/`, and no hosted tracker appears anywhere in the skills. Wording that describes GitHub Issues, GitLab, Linear, a `gh` or `glab` call, or a choice between trackers becomes the local convention instead: `Status:` lines for triage state, `Blocked by:` headers for edges, `.scratch/<feature-slug>/` for the layout.

2. **Nothing commits on your behalf.** A skill leaves its work uncommitted in the working tree and hands back. Wording that has a skill commit, push, open a branch or worktree, or open a pull request becomes hand-back wording: the work lands in the shared working tree, and the review, the commit, and the push stay yours.

3. **Docs cross-links are relative.** A link from one docs page to another is `./<name>.md` in the same bucket or `../<other-bucket>/<name>.md` in the other. An `https://aihero.dev/skills-<name>` link becomes relative. The test is where the target lives, not how the link is spelled: anything that exists in this repo (a docs page, a skill file such as `DEEPENING.md` or `template.sh`) becomes a relative path, even when upstream spells it as a `github.com/.../blob/...` URL. Truly external references stay absolute: dictionary links (`https://www.aihero.dev/ai-coding-dictionary/...`) and anything that lives only upstream. Hosted-tracker citations (issue and PR links) are dropped under invariant 1 rather than relinked.

4. **Upstream's structure is adopted as it stands.** Renames, bucket moves, graduations, and outright removals are upstream's call, so take them: what the glossary file is called, which bucket a skill sits in, which skills still exist. Following structure is what keeps the next merge small.

5. **Upstream's identity stays.** Its prose, its branding, its voice. The fork changes behaviour, not identity, so a sentence of the original is rewritten only where an invariant above forces it.

6. **No em-dashes, and only British spellings.** In every file the run touches.

## Localising a hunk

Where both sides edited the same passage, work from upstream's side and re-apply the invariants:

1. Take upstream's text as the base. It carries the new content and the structural change.
2. Rewrite any tracker or commit wording to the invariant form above.
3. Swap `https://aihero.dev/skills-*` links and in-repo `github.com/.../blob/...` links for relative ones, and call the glossary file whatever upstream currently calls it.
4. Keep the rest, including upstream's new sentences and its dictionary links.

A **clean arrival** is localised the same way. Files that merge without a conflict still carry upstream's wording, and they are the easiest thing in a run to forget.

A **whole-page rewrite** needs a different move. When upstream rewrites a page wholesale (a repo-wide style pass, say), do not resolve it hunk by hunk. Take upstream's page in full as the new base and re-apply the invariants to the whole page, using the fork's pre-merge version only as the glossary of established localised phrasings. Same destination as a hunk localisation, without the archaeology.

Release history is the exception: `CHANGELOG.md` keeps upstream's narrative, versions, claims, and words as it wrote them, so the fork's history stays readable as upstream's. Only the no-em-dash rule rewrites it, punctuation only.

Passage-by-passage rewriting rather than word swapping is a writing job, and the house rules govern it: `Call the Skill tool with "writing-for-agents"`.

## When to flag rather than resolve

An invariant and an upstream change can both be honoured only if one of them bends. Pick neither. Leave that work undone and raise it in the run's flag list, so the choice stays yours.
