# The canonical install block

One install story, one wording. `README.md`, `.changeset/*`, and every page under `docs/` must say **this** and nothing else. Change it here first, then propagate.

This fork does not publish the **Claude Code plugin**, so there is one install route: [skills.sh](https://skills.sh/stefan-crowhurst/skills), which copies editable skill files into the project.

## Claude Code: the plugin (not published from this fork)

<canonical-block name="claude-code">

The plugin is not published from this fork. The manifests under `.claude-plugin/` are kept ready (`stefan-crowhurst-skills`) for when it is, and are **not** an install route today.

</canonical-block>

## Codex, and other agents: skills.sh

[skills.sh](https://skills.sh/stefan-crowhurst/skills) copies editable skill files into the project. Use the whole-set form on `README.md`:

<canonical-block name="skills-sh-whole-set">

```bash
npx skills@latest add stefan-crowhurst/skills
```

Pick the skills you want, and which coding agents to install them on. **The installer lets you choose which skills to take: make sure `setup-matt-pocock-skills` is one of them.**

</canonical-block>

…and the single-skill form wherever one skill is named on its own. Note that **`docs/` pages are not a consumer of this block**: ai-hero renders the install widget above the body, so a page that writes the commands out duplicates it. See [writing-docs.md](./writing-docs.md).

<canonical-block name="skills-sh-one-skill">

```bash
npx skills@latest add stefan-crowhurst/skills --skill=<name>
```

```bash
npx skills@latest update <name>
```

</canonical-block>

`skills@latest` is the pinned spelling in all three. The pages under `docs/` used to carry their own copy of these commands; those blocks are now deleted rather than corrected, because the site renders the install commands itself.

## The two routes are exclusive

The plugin is a managed, read-only bundle you subscribe to. skills.sh writes files you own and edit. Installing both leaves the user with every skill twice: always say "pick one". Only skills.sh ships from this fork today; the rule applies if the plugin is ever published.

## Not the install story

`.claude-plugin/marketplace.json` makes the repo its own single-plugin marketplace (`/plugin marketplace add stefan-crowhurst/skills`, then `/plugin install stefan-crowhurst-skills@stefan-crowhurst`). It is kept ready for when the plugin ships, and is **not** documented to users today.
