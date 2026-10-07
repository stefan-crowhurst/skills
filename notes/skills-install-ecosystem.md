# Agent Skills install ecosystem, October 2026

Research findings. Date of writing: 2026-10-06. Every factual claim is traced to a primary source (official docs, the spec itself, first-party source code or manifests, official announcements). Secondary sources are labelled as such. Anything I could not confirm from a primary source is marked **unverified**.

---

## 1. The SKILL.md / Agent Skills standard

### Ownership and the spec

The format is the **Agent Skills** open standard. It was originally developed by Anthropic and released as an open standard; the specification is now maintained in the open at [github.com/agentskills/agentskills](https://github.com/agentskills/agentskills) (Apache 2.0 code, CC-BY 4.0 docs), published at [agentskills.io](https://agentskills.io) ([repo README](https://github.com/agentskills/agentskills)). Anthropic's own copy of the spec is a stub that redirects there: [anthropics/skills/spec/agent-skills-spec.md](https://github.com/anthropics/skills/blob/main/spec/agent-skills-spec.md) reads "The spec is now located at https://agentskills.io/specification". The repo was created 2025-12-16 and carried 25.9k stars at time of writing.

So: **Anthropic originated it, a community org (`agentskills`) now maintains it**, and there is a formal specification document kept in git. There is no standards body (no W3C/IETF/OMG) behind it; governance is "open to contributions from the broader ecosystem" via `CONTRIBUTING.md` and a Discord.

### What is actually specified

Per the [specification](https://agentskills.io/specification):

- **Directory layout**: a skill is a directory with a required `SKILL.md`; optional `scripts/`, `references/`, `assets/`; "any additional files or directories" allowed.
- **Frontmatter fields**, exactly six: `name` (required, max 64 chars, lowercase alphanumerics and hyphens, must match the parent directory), `description` (required, max 1024 chars), `license`, `compatibility` (max 500 chars), `metadata` (free-form string-to-string map), `allowed-tools` (space-separated, marked **Experimental**).
- **Body**: Markdown after the frontmatter, "no format restrictions".
- **Progressive disclosure**: metadata at startup (~100 tokens), body on activation (<5000 tokens recommended), resources on demand. Keep `SKILL.md` under 500 lines.
- **Validation**: a reference library, [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref), with `skills-ref validate ./my-skill`.

The integration guide is explicit that paths are **not** in the spec: "the Agent Skills specification does not mandate where skill directories live (it only defines what goes inside them)"; `.agents/skills/` is described as a convention that "has emerged" ([adding-skills-support](https://agentskills.io/client-implementation/adding-skills-support)).

### What is de facto, not specified

- **Install/discovery paths** (`.agents/skills/`, `.claude/skills/`, `~/.codex/skills`, etc.): convention only, see above.
- **Invocation control**: `disable-model-invocation` is a [Claude Code frontmatter extension](https://code.claude.com/docs/en/skills#frontmatter-reference) (also adopted by [Cursor](https://cursor.com/docs/context/skills)); `policy.allow_implicit_invocation` in `agents/openai.yaml` is a [Codex extension](https://developers.openai.com/codex/skills).
- **`agents/openai.yaml`**: an optional Codex-only sidecar for UI metadata (`interface.*`), invocation policy and tool dependencies, documented at [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills). Not in the spec. This repo carries one beside every `SKILL.md` ([.agents/invocation.md](.agents/invocation.md)).
- **Claude Code's other extensions**: `when_to_use`, `argument-hint`, `arguments`, `context: fork`, `agent`, `background`, `hooks`, `paths`, `shell`, `user-invocable`, `disallowed-tools`, `model`, `effort`, plus dynamic context injection and `${CLAUDE_SKILL_DIR}`-style substitutions. Claude Code's docs list these and say that **outside** Claude Code only the six spec fields are accepted, with a hard error otherwise ([skills frontmatter reference](https://code.claude.com/docs/en/skills#using-skill-frontmatter-outside-claude-code)).
- **Cursor extensions**: `paths` (and legacy `globs`), `disable-model-invocation`, `icon`, `color` ([Cursor skills docs](https://cursor.com/docs/context/skills)).

### Versioning

**There is no version field in the spec.** `metadata` is a free-form map and the spec's own example puts `version: "1.0"` there, which means version is unstandardised: each tool invents its own. The spec has no lockfile, no dependency manifest, and no release/changelog convention. The [docs index](https://agentskills.io/llms.txt) contains Overview, Specification, Client Showcase, Quickstart, Best practices, Optimizing descriptions, Evaluating skills, Using scripts, and Adding skills support: **no distribution, packaging or install page at all**. The integration guide punts distribution to implementers ("an API, a remote registry, or bundled assets"). This is the single most important structural fact in this report: **the standard is a file format, not a distribution system.**

### Who has adopted it

The [Client Showcase](https://agentskills.io/clients) lists roughly 40 products with official instructions links. First-party doc links verified for at least: [Claude Code](https://code.claude.com/docs/en/skills), [Claude](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview), [ChatGPT & Codex](https://developers.openai.com/codex/skills), [VS Code](https://code.visualstudio.com/docs/copilot/customization/agent-skills), [GitHub Copilot](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills), [Cursor](https://cursor.com/docs/context/skills), [Amp](https://ampcode.com/manual#agent-skills), [OpenCode](https://opencode.ai/docs/skills), [Gemini CLI](https://geminicli.com/docs/cli/skills/), [Goose](https://block.github.io/goose/docs/guides/context-engineering/using-skills/), [Junie (JetBrains)](https://junie.jetbrains.com/docs/agent-skills.html), [Roo Code](https://docs.roocode.com/features/skills), [Kiro](https://kiro.dev/docs/skills/), [Factory](https://docs.factory.ai/cli/configuration/skills), [OpenHands](https://docs.openhands.dev/overview/skills), [Letta](https://docs.letta.com/letta-code/skills/), [Spring AI](https://spring.io/blog/2026/01/13/spring-ai-generic-agent-skills/), plus Antigravity, TRAE, Snowflake Cortex Code, Databricks Genie Code, Tabnine, Qodo, Laravel Boost and others. Microsoft's [Agent Framework docs](https://learn.microsoft.com/en-us/agent-framework/agents/skills) also implement it.

---

## 2. skills.sh / vercel-labs/skills

### What it is

`vercel-labs/skills` is an MIT-licensed npm package (`skills`) plus a website. The README badges it as a **Vercel Labs Product** and calls it "The CLI for the open agent skills ecosystem" ([github.com/vercel-labs/skills](https://github.com/vercel-labs/skills)). Vercel announced it as a company release: [Introducing skills, the open agent skills ecosystem](https://vercel.com/changelog/introducing-skills-the-open-agent-skills-ecosystem) (2026-01-20), which introduced both `npx skills add <package>` and skills.sh as "a directory and leaderboard for skill packages".

Three components:

1. **CLI** (`npx skills`): add/remove/update/list/find/use/init, install by symlink (recommended) or copy, project or global scope, CI-friendly flags ([README](https://github.com/vercel-labs/skills)).
2. **Registry website** ([skills.sh](https://skills.sh)): leaderboard, topics, an "Official" curated set, packs, security audits, per-skill pages, and a documented **HTTP API** under `/api/v1/` ([skills.sh/docs/api](https://skills.sh/docs/api)).
3. **Local metadata**: a project lockfile `skills-lock.json` and a global lock `~/.agents/.skill-lock.json` (or `$XDG_STATE_HOME/skills/.skill-lock.json`), schema v3 with a `skillFolderHash` GitHub tree SHA ([src/skill-lock.ts](https://github.com/vercel-labs/skills/blob/main/src/skill-lock.ts), [AGENTS.md](https://github.com/vercel-labs/skills)). `skills experimental_install` restores from `skills-lock.json`; `skills experimental_sync` crawls `node_modules` for skills, which is a partial answer to "ship skills in npm packages". This repo already carries a `skills-lock.json` in that format at its root.

### Scope: agents, sources, registry

- **Agents**: the README's table lists ~80 agents, including Claude Code (`.claude/skills` / `~/.claude/skills`), Codex (`.agents/skills` / `~/.codex/skills`), OpenCode (`.agents/skills` / `~/.config/opencode/skills`), Cursor (`.agents/skills` / `~/.cursor/skills`), GitHub Copilot (`.agents/skills` / `~/.copilot/skills`), Windsurf (`.windsurf/skills` / `~/.codeium/windsurf/skills`), Zed (`.agents/skills` / `~/.agents/skills`), Amp/Replit/"universal" (`.agents/skills` / `~/.config/agents/skills`). **PromptScript is listed as project-only ("N/A")**, which is the direct cause of bug #1352 (below). *(The Zed and Windsurf path mappings are skills.sh's claim; I did not verify them against Zed's or Windsurf's own docs, so treat as **unverified** first-party.)*
- **Sources**: GitHub shorthand, full GitHub URL, a tree path to one skill, GitLab, Azure Repos, any git URL, local paths, private repos via existing git credentials/`gh`/SSH, and direct `SKILL.md` or archive download URLs with size caps.
- **Registry/discovery**: skills.sh indexes skills, ranks by anonymous install telemetry (opt out with `DISABLE_TELEMETRY=1` or `DO_NOT_TRACK=1`), offers fuzzy and semantic search (`/api/v1/skills/search`), a curated "official" first-party set (`/api/v1/skills/curated`, 342 skills from 87 owners in the sample response), and packs (sign in with Vercel to create one; installing does not require sign-in). Duplicate forks are flagged `isDuplicate` in the API.
- **Security scanning**: the [Security Audits](https://skills.sh/audits) page shows combined results from **Gen Agent Trust Hub, Socket and Snyk**; the [audit API](https://skills.sh/docs/api) also names Runlayer and ZeroLeaks. Audits are generated automatically after a skill's first install. The fork's upstream, `mattpocock/skills`, appears on that page: most skills "Safe / 0 alerts / Low Risk", with `improve-codebase-architecture` and `triage` at "Med Risk" from Gen and Snyk.

### Is it the "official" installer?

**No.** It positions itself as the ecosystem tool ("The CLI for the open agent skills ecosystem", "the open agent skills tool"), and the standard's own site endorses no installer: [agentskills.io](https://agentskills.io) links to the spec, examples and a client-integration guide, and to nothing about installing skills. anthropics/skills' README badges skills.sh but its documented install route is the Claude Code plugin marketplace (see section 3). skills.sh is the **most popular** installer (8.27M weekly npm downloads at time of writing, per [npmjs.com/package/skills](https://www.npmjs.com/package/skills)), not a canonical one.

### Maintenance / health signal (as of 2026-10-06)

From the GitHub repo page: 33.3k stars, 2.8k forks, 533 commits, **753 open issues**, **355 open PRs**, npm `skills` at v1.7.0 published 12 days prior (98 versions total, 104 dependents). Releases are frequent. Issue and PR turnaround on the specific bug below is poor.

### The PromptScript bug (#1352), confirmed

- [Issue #1352](https://github.com/vercel-labs/skills/issues/1352), opened 2026-06-03 by vimsucks, **still open on 2026-10-06**, labelled `bug`, **no assignee, no milestone**, two linked open PRs (#1362, #1561).
- Duplicates and root-cause write-ups: [#1424](https://github.com/vercel-labs/skills/issues/1424) and [#1496](https://github.com/vercel-labs/skills/issues/1496). Per #1496, `getUniversalAgents()` returns every agent whose `skillsDir === ".agents/skills"`, which includes PromptScript even though its `globalSkillsDir` is undefined and PromptScript is not installed; `ensureUniversalAgents()` then adds them all to the target list. The result on `-g` installs is a success block followed by a spurious "PromptScript: PromptScript does not support global skill installation" per skill, and a non-zero exit.
- Community impact is broader than this repo: [google/agents-cli#59](https://github.com/google/agents-cli/issues/59) attributes the same noise to vercel-labs/skills#1352.
- The competing fix PRs disagree on UX (one warns and continues, others silently skip), which is why they have sat unmerged. The brief says four unmerged fix PRs; I directly confirmed two linked from #1352 (#1362, #1561) and a third (#1571) discussed in #1496's thread. The exact count of four is **unverified**.

---

## 3. Official/native install mechanisms per harness

### Claude Code

Skills load from [several locations](https://code.claude.com/docs/en/skills#where-skills-live): enterprise managed settings, personal `~/.claude/skills/`, project `.claude/skills/`, nested `<subdir>/.claude/skills/`, `--add-dir` directories, **plugins**, and skills synced from a claude.ai account (downloaded into `~/.claude/skills/synced/`, read-only by design: edits there are overwritten by the next sync). Symlinked skill folders are supported and de-duplicated. Frontmatter is the richest of any harness (see section 1).

**Distribution is plugins.** [Plugins](https://code.claude.com/docs/en/plugins/install) bundle skills, agents, hooks and MCP servers; plugin skills are namespaced `/plugin-name:skill-name`. Install routes:

- In session: `/plugin install <name>@<marketplace>`, `/plugin marketplace add <owner/repo>`.
- From shell: [`claude plugin install <name>@<marketplace> --scope user|project|local`](https://code.claude.com/docs/en/plugins/cli-reference), plus `claude plugin update`, and `claude plugin validate . --strict` for authors.
- **Official marketplace** `claude-plugins-official` (source repo `anthropics/claude-plugins-official`) is preconfigured in every Claude Code install; there is also a reviewed [claude.com/marketplace](https://claude.com/docs/plugins/overview) directory where Anthropic reviews submissions before listing.
- Scopes: **user, project, local, managed**. Updates: plugin auto-update is a setting; marketplace catalogues can be refreshed or set to sync. Installed plugins are read-only managed bundles, not files you edit.

Anthropic's own skills repo recommends the plugin route: `/plugin marketplace add anthropics/skills`, then `/plugin install document-skills@anthropic-agent-skills` ([anthropics/skills README](https://github.com/anthropics/skills)). Other Anthropic routes: skills uploaded to a claude.ai account, and the **Skills API** with `package_skill.py`, both of which enforce only the six spec fields ([Claude Code docs](https://code.claude.com/docs/en/skills#using-skill-frontmatter-outside-claude-code)).

Verdict: **both project and global scope**. Plugin and managed bundles are read-only and auto-updating; personal and project skills are user-editable files.

### Codex (OpenAI)

Per [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills):

- Local skill locations: `REPO` scope `.agents/skills` at CWD, parents and repo root; `USER` scope `$HOME/.agents/skills`; `ADMIN` scope `/etc/codex/skills`; `SYSTEM` skills bundled with Codex. Symlinked skill folders are followed. Enable/disable per skill via `[[skills.config]]` entries in `~/.codex/config.toml`.
- **Distribution is plugins**: "Plugins distribute reusable skills and connectors through the universal plugin directory shared by ChatGPT and Codex", and "For installable distribution, prefer plugins". Plugins bundle one or more skills plus optional MCP connections.
- For local setup there is a bundled `$skill-installer` skill: `$skill-installer linear` installs curated skills from [openai/skills](https://github.com/openai/skills) or other repos. The docs say to use it for local experimentation and prefer plugins for reusable distribution.
- `agents/openai.yaml` is Codex's optional metadata sidecar: `interface.*` (display name, descriptions, icons, brand colour, default prompt), `policy.allow_implicit_invocation: false` to make a skill user-invoked only, and `dependencies.tools` (for example MCP servers).
- **Path discrepancy worth noting**: Codex's own docs list user scope as `$HOME/.agents/skills`, while skills.sh's agent table targets `~/.codex/skills` for Codex global installs. Cursor's docs also mention `.codex/skills` as a compatibility location. Which directory Codex treats as authoritative for global user skills is therefore **not settled from the sources I read**; treat both as load-bearing and test.

### opencode

Per [opencode.ai/docs/skills](https://opencode.ai/docs/skills):

- Loads from six locations: `.opencode/skills/` and `~/.config/opencode/skills/` (native), `.claude/skills/` and `~/.claude/skills/` (Claude-compatible), `.agents/skills/` and `~/.agents/skills/` (agent-compatible). Project-local paths are walked up to the git worktree.
- Recognised frontmatter is **only** `name`, `description`, `license`, `compatibility`, `metadata` (string-to-string map). Unknown fields are ignored. Note this list omits `allowed-tools`, which the Agent Skills spec marks experimental and which skills.sh's compatibility table claims OpenCode supports; the current opencode docs (last updated 2026-10-06) do not mention it, so treat the skills.sh table cell as **unverified**.
- Access control is per skill pattern in `opencode.json` (`permission.skill`: allow/deny/ask), overridable per agent. Skills load through a native `skill` tool.
- opencode also has a separate [plugin system](https://opencode.ai/docs/plugins/) (not the skill mechanism).

Verdict: **project and global scope**, user-editable files, no native installer command documented in that page; the ecosystem page and plugins cover extension beyond skills.

### Cursor

Per [cursor.com/docs/context/skills](https://cursor.com/docs/context/skills):

- Loads `.agents/skills/` and `.cursor/skills/` at project level, `~/.agents/skills/` and `~/.cursor/skills/` at user level. "For compatibility, Cursor also loads skills from Claude and Codex directories: `.claude/skills/`, `.codex/skills/`, `~/.claude/skills/`, and `~/.codex/skills/`." Nested skill directories are walked recursively; nested project skills are auto-scoped to their subtree.
- **"Skills aren't imported on their own."** To bring skills in from a GitHub repository you package them as a **plugin** and publish through a marketplace: import the repository in Customize (it needs a `.cursor-plugin/marketplace.json`), or add a team marketplace, then install the plugin. There is also "Publish a skill to your team" and an optional **Sync Skills for Cloud Agents** toggle that copies `~/.cursor/skills/` for cloud use.
- Frontmatter extensions: `paths` (legacy `globs`), `disable-model-invocation`, `icon`, `color`, `metadata`.

Verdict: **project and global scope**, user-editable files; third-party distribution runs through Cursor's plugin marketplaces, not a raw git install.

### GitHub Copilot (and `gh skill`)

Per [docs.github.com agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) and [Adding agent skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills):

- Project skills in `.github/skills`, `.claude/skills` or `.agents/skills`; personal skills in `~/.copilot/skills` or `~/.agents/skills`.
- Manual route: download a skill directory and move it into place (the docs point at `anthropics/skills` and `github/awesome-copilot`).
- **First-party installer: `gh skill`** in GitHub CLI, public preview, requires gh 2.90.0 or later ([gh skill manual](https://cli.github.com/manual/gh_skill), [gh skill install](https://cli.github.com/manual/gh_skill_install)). Subcommands: `search`, `preview`, `install`, `update`, `publish`, `list`.
  - `gh skill install OWNER/REPO SKILL` with `--agent` (github-copilot, claude-code, cursor, codex, gemini-cli, amp, junie, opencode and many more), `--scope project|user`, `--dir`, `--all`, `--force`, `--from-local`.
  - **Version pinning**: `SKILL@v1.2.0` or `@SHA`, plus `--pin` to lock and skip updates. `gh skill update --all` refreshes; pinned skills are skipped.
  - **Provenance**: install writes source repository, ref and tree SHA into the skill's `SKILL.md` frontmatter; `update` uses that metadata. `gh skill publish` validates against the Agent Skills spec and checks remote settings like tag protection, secret scanning and code scanning.
  - At project scope several agents (Copilot, Cursor, Codex, Gemini CLI, Antigravity, Amp, Cline, OpenCode, Warp) share `.agents/skills`, so one install serves them all.
  - GitHub's own docs carry a warning: "Skills are not verified by GitHub and may contain prompt injections, hidden instructions, or malicious scripts. Always inspect the content of a skill before installation using `gh skill preview`."

This is the strongest native alternative to skills.sh: first-party, multi-agent, version-pinned, provenance-carrying. The trade-off is that it is in **public preview and subject to change**.

### Amp

Per [ampcode.com/manual#agent-skills](https://ampcode.com/manual#agent-skills):

- Native install command: `amp skill add <source>` from a GitHub repo, git URL or local path; `--global` installs to `~/.config/agents/skills/`. Project skills go in `.agents/skills/`.
- A built-in `building-skills` skill can create or install skills conversationally.
- Skill sources and precedence is a documented 10-plus-entry list: `~/.config/agents/skills/`, `~/.agents/skills/`, `~/.config/amp/skills/`, project and parent `.agents/skills/`, those directories' `.claude/skills/`, `~/.claude/skills/`, `~/.claude/plugins/cache/`, `amp.skills.path`, built-ins, then personal and workspace skills repositories.
- Personal and workspace **skills repositories** are a first-party share mechanism: `amp skill import` / `amp skill update` manage shared imports, `amp skills repositories` lists clone commands, and repository owners can **require signed commits**.

### Zed and Windsurf

skills.sh's table maps Zed to `.agents/skills/` / `~/.agents/skills/` and Windsurf to `.windsurf/skills/` / `~/.codeium/windsurf/skills/`. I did not verify these against Zed's or Windsurf's own documentation: **unverified** as first-party claims. Neither appears in the Agent Skills Client Showcase with an instructions link, so their official skill-install story (if any) is unclear to me.

### Summary table

| Harness | Project scope | Global / user scope | Editable files or managed bundle | Native installer | Update story |
|---|---|---|---|---|---|
| Claude Code | `.claude/skills/` | `~/.claude/skills/`, enterprise managed dir | Both: files or read-only plugin | `/plugin install`, `claude plugin install` | Plugin auto-update; files are yours |
| Codex | `.agents/skills/` | `$HOME/.agents/skills/` (docs), `~/.codex/skills` (skills.sh) | Files, or plugin bundle | `$skill-installer`, plugins | Plugins via plugin directory |
| opencode | `.opencode/skills/`, `.claude/skills/`, `.agents/skills/` | `~/.config/opencode/skills/`, `~/.claude/skills/`, `~/.agents/skills/` | Editable files | none documented on the skills page | none documented |
| Cursor | `.cursor/skills/`, `.agents/skills/` | `~/.cursor/skills/`, `~/.agents/skills/` | Editable files, or team plugin | via plugin marketplace | Team marketplace, cloud sync |
| GitHub Copilot | `.github/skills/`, `.claude/skills/`, `.agents/skills/` | `~/.copilot/skills/`, `~/.agents/skills/` | Editable files | `gh skill install` | `gh skill update`, pins |
| Amp | `.agents/skills/` | `~/.config/agents/skills/` | Editable files | `amp skill add` | `amp skill update`, signed-commit repos |

---

## 4. Third-party alternatives and the wider tooling scene

### First-party and near-first-party installers

- **`gh skill`** (GitHub CLI): covered in section 3. It is the only installer besides skills.sh that ships with a widely deployed developer tool, and the only mainstream one with first-class version pinning and provenance.
- **Claude Code plugin marketplaces** and **Cursor plugin marketplaces** are per-harness distribution channels rather than cross-harness installers (see section 3).
- **ChatGPT/Codex plugin directory**: plugins are OpenAI's distribution unit for skills and connectors ([plugins docs](https://developers.openai.com/plugins/build/plugins)).

### Independent installers and registries

Each of these is a community or vendor project. Maintenance signals are what I could read from the repositories at time of writing; long-term viability is unproven for all of them.

| Project | What it is | Install model | Update / pinning | Trust posture | Maintenance signal |
|---|---|---|---|---|---|
| [skilld](https://github.com/skilld-dev/skilld) | "Open-source, privacy friendly skills.sh alternative", curated registry by harlan_zw | Native Rust binary (npm wrapper); `skilld install skilld:OWNER/REPO/SKILL`; `skilld run` prints SKILL.md to stdout without writing files | Tracks the commit each skill came from; `skilld update` | Checks its own digest and attestation before writing a file; emphasises reading the exact file before install | 310 stars, 488 commits, created 2026-02-03 |
| [skillmd / skillmds](https://github.com/skillmds/skillmd) | Toolchain + [skillmd.com](https://skillmd.com/) open registry, "25,000+ skills from 470+ publishers" | `npx skillmds add <slug-or-repo>`; also a Claude Code marketplace (`skillmds/skillmd`) | Registry pins skills to a commit | Every skill linted against the format and "security-scanned for what it can do"; `skillmd scan` reports scripts, network calls, secret access; install "lints first, never executes scripts" | MIT, npm-published, CI on GitHub Actions |
| [skillpm](https://github.com/sbroenne/skillpm) | "npm-native package manager for Agent Skills" | `npm install` then scan `node_modules`, wire into agent dirs **via the `skills` CLI** | npm semver and `package.json` lockfiles, reproducible installs | Inherits npm's audit model | 14 stars, 56 commits, created 2026-02-25 |
| [openskills](https://www.npmjs.com/package/openskills) | "Universal skills loader", `npx openskills` | Install from git repos to project or `~/.claude/skills` | `openskills update` | none notable | npm-published to 1.5.0 |
| [skills-npm](https://unpkg.com/skills-npm%401.2.0/README.md) | A **convention proposal**: ship skills inside npm packages, symlink on `prepare` | `npm i -D skills-npm; npx skills-npm setup` | Tracks the npm package version | Explicitly critiques git-only sources and version mismatch | small, early |
| [agent-skills-cli](https://github.com/Karanjot786/agent-skills-cli) / [agentskills.in](https://www.agentskills.in/) | Marketplace + CLI claiming "175,000+ skills, 45 AI agents" | `npm install -g agent-skills-cli; skills install @scope/name`, also git, npm and local sources | `.skillsrc` config for registries | none that I could verify; treat the scale claims as vendor marketing, **unverified** | one main author |
| [skillctl](https://skillctl.xyz/) | `pip install skillctl`, search/install/lint | Clone, symlink, register in a manifest | SHA-tracked re-indexing | "trusted registries" concept, quality lint 0-100 | small, v0.1.0 |
| [skills-lock](https://github.com/pcomans/skills-lock) | A **lockfile wrapper** around `npx skills` | `npx skills-lock add anthropics/skills --skill pdf` | Pins the **git commit SHA** in `skills.lock`; `skills-lock install` restores and verifies file integrity, reinstalling on drift | Integrity check against locked ref | small; explicitly exists because "npx skills has no --ref flag, no lockfile, and no way to pin" |

### Lockfile and packaging conventions, fragmented

There is no single lockfile convention. At least four coexist:

1. **`skills-lock.json`** (vercel-labs/skills, project-local, checked in) plus **`~/.agents/.skill-lock.json`** (global, schema v3 with `skillFolderHash`). This repo already carries a `skills-lock.json` in that shape. Restore is `skills experimental_install`.
2. **`skills.lock`** (pcomans/skills-lock): records source URL, path and commit SHA per skill, npm-style.
3. **`skills.lock`** (the [skills-lock org spec](https://github.com/skills-lock/skil-lock/blob/main/SPEC.md), YAML `schema_version: "0.1"`): a *capability-surface* lockfile recording what shell commands a skill runs, what network endpoints it reaches and what paths it reads and writes, for review-on-PR drift gating. Different purpose, same filename as item 2.
4. **Provenance in `SKILL.md` frontmatter** (gh skill): source repo, ref and tree SHA written into the installed skill.

On npm as a channel: there is no official npm convention for skills. `skillpm` and `skills-npm` propose one; the vercel CLI's `experimental_sync` crawls `node_modules`, which tacitly accepts skills shipped inside npm packages. The npm package `skills` itself is the installer, not a skill host.

### Harness-specific plugin marketplaces

- Claude Code: `.claude-plugin/marketplace.json` + `.claude-plugin/plugin.json`, official marketplace `claude-plugins-official`, plus a reviewed directory at claude.com/marketplace.
- Cursor: `.cursor-plugin/marketplace.json`, team marketplaces, "publish a skill to your team".
- ChatGPT/Codex: the universal plugin directory, with a submission and review process ([Submit and publish](https://developers.openai.com/plugins/deploy/submission)). There is even a guide for [submitting a Claude Code plugin](https://developers.openai.com/plugins/guides/submit-claude-plugin) to that directory.
- skillmd can also be added as a Claude Code marketplace.

### Is a canonical distribution mechanism emerging?

**Not in the standard.** agentskills.io defines the file format and an integration guide, and says nothing about distribution. What is emerging is three parallel, incompatible answers: (a) **skills.sh** as the de facto cross-harness installer and registry, (b) **`gh skill`** as GitHub's first-party answer with pinning and provenance, (c) **per-harness plugin systems** as the "managed bundle" channel (Claude Code, Cursor, ChatGPT/Codex). A fourth thread, npm-as-registry, is proposed by small projects and not standardised.

---

## 5. Ecosystem shape and risks

### Where it is fragmenting

- **Install and discovery paths**: the spec deliberately does not fix them ([adding-skills-support](https://agentskills.io/client-implementation/adding-skills-support)). Every harness invents its own plus a compatibility list. Codex's user scope (`~/.agents/skills` in Codex docs vs `~/.codex/skills` in skills.sh) is an example of two ecosystems not agreeing on one path.
- **Agent-specific frontmatter and sidecars**: `agents/openai.yaml` (Codex UI/policy), `disable-model-invocation` (Claude Code, Cursor), `when_to_use` / `context: fork` / `hooks` / `paths` / `shell` (Claude Code), `icon` / `color` / `globs` (Cursor). The practical rule the ecosystem has converged on is "unknown fields ignored" (opencode) or "hard error outside the six spec fields" (Claude Code's claude.ai/Skills-API path), which means a skill using Claude Code extensions cannot be uploaded to claude.ai but can be installed as a file.
- **Invocation semantics**: this repo's own `.agents/invocation.md` has to pair `disable-model-invocation: true` (Claude Code) with `policy.allow_implicit_invocation: false` (Codex) to express one concept, "user-invoked only", in two harnesses. That is the fragmentation in miniature.
- **Curation of subsets**: this fork's bucketed layout could not be expressed in Codex's single-path plugin manifest, per [.agents/adr/0002](.agents/adr/0002-ship-as-a-claude-code-plugin.md). Each plugin format selects skills differently (array of paths vs one path string), so a curated subset is not portable between them.
- **Lockfiles and pinning**: four conventions (section 4), plus per-tool update semantics.

### Is anyone pushing a cross-harness standard for distribution?

Not that I could find in primary sources. agentskills.io has no distribution working group or page. What exists is vendor-shaped: Vercel's registry plus CLI, GitHub's `gh skill`, and each harness's plugin system. The [Client Showcase](https://agentskills.io/clients) standardises on the format and points each product at its own install instructions, which is a deliberate choice to leave distribution to clients. (A community "skills.lock" capability-surface spec exists but is about auditing installed skills, not distributing them.)

### Security and trust concerns

Skills run with the agent's permissions and can ship executable `scripts/`, so distribution is a supply-chain surface. Primary and semi-primary evidence:

- **OWASP Agentic Skills Top 10, [AST02 Supply Chain Compromise](https://owasp.org/www-project-agentic-skills-top-10/ast02)** (Critical): "Skill registries and distribution channels lack the provenance controls common in mature package ecosystems (npm, PyPI, Cargo)." It cites dependency confusion via a skill's own `requirements.txt`/`package.json`, and account-takeover re-registration.
- **Trail of Bits, 2026-06-03** (cited in AST02, secondary): public skill marketplaces including skills.sh run a "ship-first, secure-later" model, and the scanners meant to backstop them were bypassed in under an hour. Their recommendation is the classic supply-chain one: curated internal marketplaces, pinned versions, controlled publishing.
- **Air Security, "The Story of Skills", 2026-06-22** (cited in AST02, secondary): a researcher-built malicious skill entered a ~36K-star community plugin marketplace via an accepted pull request, inherited its credibility and reached over 26,000 agents.
- **Air Security, "The Circus of Skills", 2026-06-24** (cited in AST02, secondary): of 142,836 live skills scanned, 17,822 (about 12.4%, 6.7M installs) depended on at least one untrusted external resource.
- **skills.sh-specific**: AST02 records researchers taking over the most popular video-generation skill on skills.sh (11,483 installs) by re-registering its deleted owner account, with the listing keeping its stars and installs.
- **vercel-labs/skills issue tracker** (primary): [#353](https://github.com/vercel-labs/skills/issues/353) namespace squatting in skill discovery, rated High/Critical by the reporter, still open; [#863](https://github.com/vercel-labs/skills/issues/863) `npx skills add openclaw/skills@bird` resolved to a different owner's artifact containing an obfuscated installer block; [#1469](https://github.com/vercel-labs/skills/issues/1469) installing `vercel/eve` wrote 1,831 monorepo files instead of one skill; [#1222](https://github.com/vercel-labs/skills/issues/1222) notes there is no post-install drift gate to catch a skill that changes behaviour after it lands in your repo.
- **GitHub's own docs** warn that skills "may contain prompt injections, hidden instructions, or malicious scripts" and are "not verified by GitHub".
- skills.sh does run multi-vendor audits (Socket, Snyk, Gen Agent Trust Hub, and per its API also Runlayer and ZeroLeaks) and surfaces them per skill, which is more than most alternatives do. Audits are point-in-time and generated after first install.

### Community commentary on depending on skills.sh

The PromptScript bug thread is the clearest signal: users on other projects ([google/agents-cli#59](https://github.com/google/agents-cli/issues/59)) treat vercel-labs/skills#1352 as an upstream defect they have to explain away, and independent tools cite the CLI's missing pinning as their reason to exist ("npx skills has no --ref flag, no lockfile, and no way to pin", [pcomans/skills-lock](https://github.com/pcomans/skills-lock)). The 753 open issues and 355 open PRs, and four months with no fix for a cosmetic-but-noisy global-install failure, are the maintenance risk in one picture. Against that: 8.27M weekly downloads and ~80 agent targets make it the de facto default, and the file-writing install model matches how people actually use these skills.

---

## What fits this fork

Constraints read from this repo: one canonical install story and wording, kept in [.agents/install-block.md](.agents/install-block.md); an **editable-files** product position ("skills.sh writes files you own and edit") that is explicitly exclusive with the managed read-only plugin bundle ("always say 'pick one'"); a **fork** of mattpocock/skills with its own name (`stefan-crowhurst-skills`), manifests ready but unpublished; a **bucketed** layout whose promoted/non-promoted split a plugin must express exactly ([.agents/adr/0002](.agents/adr/0002-ship-as-a-claude-code-plugin.md)); multi-harness targets (Claude Code, Codex, opencode and others); **low maintenance**; and a dev-only symlink script already in `scripts/link-skills.sh` that is explicitly "not a supported installer".

Ranked options. Presenting trade-offs only; this is not a decision.

### 1. Keep skills.sh as the canonical route, and add `gh skill` as the second documented route

Both write editable files into the user's project or home directory, which is the fork's stated model. Both install from this git repo as-is, so the bucketed layout needs no restructuring. Maintenance cost is wording in `.agents/install-block.md`, not code.

- For: widest harness coverage today (~80 agents for skills.sh, 30-plus for `gh skill`); `gh skill` adds what skills.sh lacks: `--pin`/`@SHA` version pinning, provenance written into `SKILL.md`, and spec validation on publish (`gh skill publish --dry-run` could become a cheap CI check for the promoted set). `gh skill` project scope already shares `.agents/skills` across Copilot, Cursor, Codex, Gemini CLI, Amp, Cline, OpenCode and Warp, so one command covers several harnesses.
- **Layout caveat, confirmed against [`gh skill install`](https://cli.github.com/manual/gh_skill_install)**: auto-discovery follows the `skills/*/SKILL.md` convention (one level under `skills/`, or the same shape nested under a prefix). This repo's bucketed layout is `skills/<bucket>/<skill>/SKILL.md`, two levels deep, so `--all` and name-based lookup may find nothing without exact `.../SKILL.md` paths (`gh skill install stefan-crowhurst/skills skills/engineering/ask-matt/SKILL.md`). The docs do accept "any `.../SKILL.md` path", so per-skill docs pages could each carry an exact path, but whole-set installs would need a scripted list or a layout change. **Verify against a real `gh skill` run before documenting this route.** Also note local installs via `--from-local` copy files rather than symlink them.
- Against: two routes dilutes "one install story, one wording"; `gh skill` is public preview and subject to change; skills.sh keeps bug #1352 and the trust concerns in section 5; `gh skill` defaults to agent `github-copilot` and scope `project`, so docs must spell out `--agent` and `--scope` or users get the wrong directory.

### 2. Publish the Claude Code plugin from this fork (upstream's model)

`.claude-plugin/plugin.json` and `marketplace.json` already validate (`claude plugin validate . --strict` is already a documented house step) and already list exactly the promoted set.

- For: zero-friction managed, auto-updating bundle for Claude users; the fork's own marketplace works without waiting for an official listing (`/plugin marketplace add stefan-crowhurst/skills`); upstream's experience (ADR 0002, 2026-08-05 update) shows the official marketplace route is viable and is `claude plugins install`-able.
- Against: per `install-block.md` this route is **exclusive** with skills.sh, so publishing it forces the "pick one" wording to become real and splits the audience; it serves Claude Code only, not Codex or opencode; a listing is SHA-pinned, so installed users see a release only when the pin moves (upstream's listing sat two commits behind `main`); plugin `version` must track `package.json` on every release.

### 3. Promote a clone-and-symlink route (generalise `scripts/link-skills.sh`)

Zero dependencies, fully editable, `git pull` is the update mechanism, works for every harness that reads `~/.claude/skills` or `~/.agents/skills`. It is exactly what the repo already does locally.

- For: lowest possible maintenance surface for the repo (the script exists and is tested by its author); immune to third-party installer bugs and outages; the fork model is literally "you own the files".
- Against: the script header currently disclaims it ("It is not a supported installer. Modifications to it, or requests for modifications, will not be approved"), so promoting it reverses a standing decision; it would need Windows and permission handling, per-skill selection, and docs; users must clone the repo and run a script.

### 4. Defer: native Codex plugin via a flat curated copy (ADR 0002 escape hatch (b))

Committing a generated flat directory of promoted skills would give Codex a managed bundle.

- For: native Codex distribution, matching what Codex's own docs prefer ("For installable distribution, prefer plugins").
- Against: a second source of truth and a sync burden per release; the alternative (restructuring `skills/` to promoted-only) has a large blast radius across `CLAUDE.md`, `scripts/link-skills.sh`, bucket READMEs and the `in-progress/` feedback loop. Revisit only if Codex adds a `skills` array or preserves symlinks on install.

### 5. npm packaging (skillpm / skills-npm style)

Not recommended for this fork. It adds a release surface and a versioning discipline to a repo whose product position is editable files in a git checkout, and there is no npm convention for skills to conform to. If reproducible installs are wanted, the `skills-lock.json` already in this repo's root is the lower-effort lever: `npx skills experimental_install` restores from it.

### Notes that apply to any option

- Whatever is chosen must be expressed once in `.agents/install-block.md` and propagated; the block already forbids two competing stories being documented side by side without the "pick one" rule.
- Any option that needs git operations (pushing a tag so a marketplace pin moves, publishing, committing a generated flat copy) is a **human** action under this repo's house rules.
- Keep the fork's invocation pair (`disable-model-invocation` + `agents/openai.yaml` `policy.allow_implicit_invocation`) intact: it is the only thing making "user-invoked" mean the same thing in Claude Code and Codex, and it is de facto rather than spec'd.

---

## Sources

Primary sources, consulted 2026-10-06:

- Agent Skills specification: https://agentskills.io/specification and https://github.com/agentskills/agentskills (docs index https://agentskills.io/llms.txt; client showcase https://agentskills.io/clients; integration guide https://agentskills.io/client-implementation/adding-skills-support)
- Anthropic's skills repo and spec stub: https://github.com/anthropics/skills and https://github.com/anthropics/skills/blob/main/spec/agent-skills-spec.md
- vercel-labs/skills: https://github.com/vercel-labs/skills (README, `src/skill-lock.ts`, `AGENTS.md`); skills.sh site and docs: https://skills.sh/docs/cli, /docs/api, /docs/faq, /audits; npm package https://www.npmjs.com/package/skills; Vercel announcement https://vercel.com/changelog/introducing-skills-the-open-agent-skills-ecosystem
- Bug #1352 and related: https://github.com/vercel-labs/skills/issues/1352, /1424, /1496; PRs referenced: https://github.com/vercel-labs/skills/pull/1362, /pull/1561, /pull/1571
- Other vercel-labs/skills issues: /353 (namespace squatting), /863 (ambiguous source), /1469 (monorepo install), /1222 (post-install drift gate)
- Claude Code: https://code.claude.com/docs/en/skills, /plugins/install, /plugins/cli-reference, /plugin-marketplaces, /discover-plugins; Claude platform https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
- Codex: https://developers.openai.com/codex/skills (plus /plugins/build/plugins)
- opencode: https://opencode.ai/docs/skills and https://opencode.ai/docs/plugins/
- Cursor: https://cursor.com/docs/context/skills
- GitHub Copilot: https://docs.github.com/en/copilot/concepts/agents/about-agent-skills and https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills; GitHub CLI https://cli.github.com/manual/gh_skill and /gh_skill_install
- Amp: https://ampcode.com/manual#agent-skills and https://ampcode.com/docs/customize/skills
- VS Code: https://code.visualstudio.com/docs/copilot/customization/agent-skills; Microsoft Agent Framework https://learn.microsoft.com/en-us/agent-framework/agents/skills
- Third-party tools: https://github.com/skilld-dev/skilld, https://github.com/skillmds/skillmd, https://github.com/sbroenne/skillpm, https://www.npmjs.com/package/openskills, https://github.com/pcomans/skills-lock, https://github.com/skills-lock/skil-lock/blob/main/SPEC.md, https://github.com/Karanjot786/agent-skills-cli, https://skillctl.xyz/, https://www.agentskills.in/
- Security: OWASP Agentic Skills Top 10 AST02 https://owasp.org/www-project-agentic-skills-top-10/ast02 (which cites, as secondary sources, Trail of Bits 2026-06-03 and Air Security 2026-06-22 and 2026-06-24; I did not read those original posts)

Local files read: `.agents/install-block.md`, `.agents/adr/0002-ship-as-a-claude-code-plugin.md`, `.agents/invocation.md`, `CLAUDE.md`, `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `scripts/link-skills.sh`, `skills-lock.json`, `package.json`.
