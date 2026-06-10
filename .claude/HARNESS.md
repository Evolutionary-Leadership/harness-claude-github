# Harness Context

This project was scaffolded from the
[`evolutionary-leadership/harness-claude-github`](https://github.com/evolutionary-leadership/harness-claude-github)
template repo (variant: **harness-claude-github**) using GitHub's
"Use this template" button. The template added automated CI/CD
infrastructure (feature branches, auto-merge, releases), not application
code. Understanding what it set up helps you work with it instead of
against it.

The template content itself is authored in
[`evolutionary-leadership/harness-forge`](https://github.com/evolutionary-leadership/harness-forge)
and synced from there into this template repo on every harness release.

## Architecture

### Branch naming drives everything

```
claude/<codename>-<sessionId>  ← you work here (random codename)
       ↓ first push: slug commit from set-feature-name.sh, or any code push
       ↓ (GitHub Action)
feature/<name>                 ← created automatically from dev
       ↓ (/mergedev)
dev                            ← PR auto-merged
```

- The session branch starts with a random codename
  (`claude/<adjective-scientist>-<id>`). To get a meaningful name, Claude
  runs `bash .claude/scripts/set-feature-name.sh <slug>` as its first
  action; it writes `.harness-feature` and pushes.
- The feature name is resolved as: use the slug in `.harness-feature` if
  present and valid, otherwise fall back to the codename (the `claude/`
  prefix and `-<sessionId>` suffix stripped). See "Feature naming" below.
- Pushing to a `claude/` branch triggers the Action that creates/updates
  the corresponding `feature/<name>` branch.

### Feature naming

Feature branches are named after the work, not the random session codename.
The mechanism:

- **Source of truth:** a committed file `.harness-feature` holding a
  kebab-case slug. Claude sets it early via
  `bash .claude/scripts/set-feature-name.sh <slug>`, which sanitizes the
  input, writes the file, commits, and pushes.
- **Resolution (everywhere):** use the slug if `.harness-feature` is
  present and valid (`^[a-z0-9][a-z0-9-]{0,40}$`, and not `dev` or `main`),
  otherwise fall back to the codename. The shared resolver is
  `.claude/scripts/resolve-feature-name.sh`; the workflows
  (`claude-to-feature-branch.yml`, `claude-mergedev.yml`) apply the
  identical check.
- **Set it before the first push** so the feature branch is created with the
  good name from the start.
- **Graceful fallback:** if `set-feature-name.sh` is never called, the first
  code push still creates `feature/<codename>`. Naming is an improvement,
  never a requirement.
- **No leak to dev:** `.harness-feature` is removed by the mergedev workflow
  before the merge, so a future session cloned from dev never inherits a
  stale name. For this reason it must stay out of `.gitignore` (the
  workflows read it from the commit).

**Where do I look for X:**

| What | Where |
|------|-------|
| Provisioning trigger | A `claude/` push (the slug commit, or first code push) |
| Feature branch | `feature/<name>` |
| CI checks | Only on the PR to `dev`/`main` |
| Current feature name | `bash .claude/scripts/resolve-feature-name.sh` |

### Signal files

- **`.pr-description.md`**: Committing this file to the repo root triggers
  the GitHub Action to create a PR from `feature/<name>` → `dev` and
  auto-merge it. The `/mergedev` skill writes this file for you. If the
  frontmatter contains `review: true`, the PR is created but NOT auto-merged
  (used by the `/review` skill). If `hotfix: true`, the hotfix workflow
  handles it instead.
- **`.release-description.md`**: Committing this file triggers the release
  workflow to create a PR from `dev` → `main`, tag a version, and create a
  GitHub Release. The `/release` skill writes this file.
- **`.harness-feature`**: A committed one-line kebab-case slug naming this
  feature, written by `set-feature-name.sh`. The workflows and shell
  scripts resolve the feature name from it (with a codename fallback). It
  is removed before the merge to dev (by `claude-mergedev.yml`) so the name
  never leaks onto dev and into the next session. Unlike the other signal
  files it must stay tracked (not in `.gitignore`), because the workflows
  read it from the commit.

### `.harness-version` configuration

The `.harness-version` file supports these fields:

```yaml
harness: harness-claude-github
version: 0.3.38
repo: evolutionary-leadership/harness-forge
traits: nodejs, typescript, express
check: npm test && npm run lint
reviewers: teammate1, teammate2
```

- **`harness`**: variant identifier; matches the per-cell template repo
  this project was scaffolded from.
- **`version`**: harness version installed; used by `/harness-upgrade` to
  diff against the latest release.
- **`repo`**: the upstream forge repo (`evolutionary-leadership/harness-forge`),
  which hosts `VERSION`, `migrations/`, and `stacks/traits/`.
- **`check`**: CI command to run on PRs to dev. When configured, the
  `feature-branch-checks.yml` workflow runs this command, and mergedev uses
  `gh pr merge --auto` to wait for checks.
- **`reviewers`**: Default reviewers assigned when using `/review`.
- **`traits`**: stack-specific best-practice files installed under
  `.claude/traits/` and managed by `/harness-upgrade`.

**Prerequisites for CI checks:**
- Enable "Allow auto-merge" in GitHub repo settings (Settings → General)
- Add a branch protection rule for `dev` requiring the "check" status check
- Optionally add the same for `main` to gate releases and hotfixes

### Hooks

- **SessionStart**: Runs `.claude/scripts/session-start.sh` on every new
  session. On a `claude/` branch, it resolves the feature name and, if a
  matching `feature/<name>` branch already exists, merges previous work. It
  no longer pushes an init commit: a fresh session just prints naming
  guidance. The feature branch is created on Claude's first push, ideally
  the `set-feature-name.sh` slug commit (see "Feature naming"). You do not
  need `/feature` to start; just describe what you want to build and Claude
  names the session before its first push.
- **PreToolUse (Write/Edit/Bash)**: Runs
  `.claude/hooks/prevent-em-dash.sh`, which blocks any write that contains
  a U+2014 em dash.

## Managed trait files

Stack-specific best practices live in `.claude/traits/` as separate managed
files (e.g. `.claude/traits/nodejs.md`, `.claude/traits/typescript.md`).
These are fetched from the forge repo's `stacks/traits/` directory and
can be auto-updated via `/harness-upgrade`.

To install traits, add the trait names to `.harness-version`:

```
traits: nodejs, typescript, express, vitest, eslint, pnpm
```

Then run `/harness-upgrade`. It will fetch the matching trait files from
the forge and install them in `.claude/traits/`. On future upgrades, it will
show diffs and let you update to the latest best practices.

Add this line to your project's `CLAUDE.md` so the AI reads them:

```
Read `.claude/traits/` for stack-specific best practices before writing code.
```

Available traits and presets are listed in the forge repo's `stacks/` directory.

## Migration system

Each harness version has a structured migration file
(`migrations/X.Y.Z.yaml` in the forge repo) describing what changed. The
`/harness-upgrade` skill uses these to:

- **Filter by relevance**: only show changes that affect your variant and traits
- **Categorize by priority**: REQUIRED (infrastructure), RECOMMENDED (traits),
  INFORMATIONAL (other)
- **Show context**: what changed and why, not just raw diffs

Migration files are auto-generated by the `harness-version-bump.yml`
workflow in the forge whenever a feature merges to `dev`. They are never
manually authored.

## Harness-managed files

These files are maintained by the harness and replaced on
`/harness-upgrade`. Do not edit them; your changes will be overwritten.

| File | Purpose |
|------|---------|
| `.github/workflows/claude-to-feature-branch.yml` | Merges `claude/` branches into `feature/` branches |
| `.github/workflows/claude-mergedev.yml` | Creates PR from `feature/` to `dev` and auto-merges (or opens for review) |
| `.github/workflows/feature-branch-checks.yml` | Runs CI checks on PRs to dev (reads `check:` from `.harness-version`) |
| `.github/workflows/release.yml` | Creates release PR dev → main, tags version, creates GitHub Release |
| `.github/workflows/hotfix.yml` | Handles hotfix PRs to main, tags patch release, back-merges to dev |
| `.github/workflows/feature-merge-cleanup.yml` | Deletes feature branch after merge to dev |
| `.claude/scripts/session-start.sh` | Session startup hook |
| `.claude/scripts/list-skills.sh` | Skill discovery script |
| `.claude/scripts/resolve-feature-name.sh` | Resolves the feature name (slug from `.harness-feature`, else session codename); shared by the hooks, scripts, and workflows |
| `.claude/scripts/set-feature-name.sh` | Names the session's feature: sanitizes a slug, writes `.harness-feature`, commits, and pushes to trigger branch creation |
| `.claude/hooks/prevent-em-dash.sh` | Blocks writes containing U+2014 em dashes |
| `.claude/skills/getting-started/SKILL.md` | Orientation skill |
| `.claude/skills/feature/SKILL.md` | `/feature` skill |
| `.claude/skills/mergedev/SKILL.md` | `/mergedev` skill |
| `.claude/skills/review/SKILL.md` | `/review` skill: submit PR for team review |
| `.claude/skills/release/SKILL.md` | `/release` skill: ship dev to production |
| `.claude/skills/hotfix/SKILL.md` | `/hotfix` skill: emergency production fix |
| `.claude/skills/status/SKILL.md` | `/status` skill: team dashboard |
| `.claude/skills/changelog/SKILL.md` | `/changelog` skill: generate changelog |
| `.claude/skills/deps/SKILL.md` | `/deps` skill: handle Dependabot PRs |
| `.claude/skills/continue/SKILL.md` | `/continue` skill: resume in-progress feature |
| `.claude/skills/chat/SKILL.md` | `/chat` skill: conversation mode (no file changes) |
| `.claude/skills/endchat/SKILL.md` | `/endchat` skill: clean up the orphan feature branch left behind by `/chat` |
| `.claude/skills/rollback/SKILL.md` | `/rollback` skill: revert bad deploy |
| `.claude/skills/harness-upgrade/SKILL.md` | `/harness-upgrade` skill |
| `.claude/agents/docs-updater.md` | Documentation auditor agent (runs during mergedev) |
| `.claude/HARNESS.md` | This file |
| `.harness-version` | Version tracking |
| `.claude/traits/*.md` | Stack best practices (managed per `traits:` in `.harness-version`) |

## Harness-provided starting points

The harness created these files as a starting point. You own them, so edit
freely to match your project. On `/harness-upgrade`, these are diffed and
you choose whether to accept upstream changes.

| File | What to customize |
|------|-------------------|
| `.claude/settings.json` | Add your own hooks and tool permissions alongside the harness-provided ones |
| `.github/dependabot.yml` | Add entries for your package ecosystems (npm, pip, Docker, etc.) |

The harness ships `.claude/settings.json` with an `env` block that sets
`API_TIMEOUT_MS=900000` and `CLAUDE_CODE_MAX_RETRIES=15` to harden
sessions against stream idle timeouts. Keep these values (or raise them)
when you add your own keys; see "Avoiding stream timeouts" in
`claude-md-snippet.md` for context.

## Starter scaffold (write-once)

Some variants ship write-once scaffold files (e.g. `server.js`, a
`package.json`, a `.gitignore`) so the deploy pipeline has something to
build on the first push. **This variant ships none.** It has no deploy
target, so a starter app would have nowhere to run. The category is
documented here for consistency with `harness-claude-github-railway`.

If a future starter file ever ships in this variant, `/harness-upgrade`
will create it once on first install, never overwrite it on upgrade, and
never recreate it if you delete it.

## Project-owned files

Everything else belongs to the project. The harness does not touch:

- **`CLAUDE.md`**: Your project instructions. The harness provides
  `claude-md-snippet.md` as a starting point; copy what you need.
- **All application code**: Source files, configs, tests, etc.
- **Custom skills**: Any skill you add to `.claude/skills/` that isn't
  listed above.

## How to extend

### Adding a skill

Create `.claude/skills/<name>/SKILL.md` with YAML frontmatter (`name`,
`description`). Custom skills are not touched by `/harness-upgrade`.

### Adding an agent

Create `.claude/agents/<name>.md` with YAML frontmatter (`name`,
`description`, `allowed-tools`). Agents are autonomous specialists that
run in their own context via the Agent tool. Custom agents are not touched
by `/harness-upgrade`.

### Adding workflows

Prefer adding new workflow files in `.github/workflows/` over modifying
harness-managed ones. New files won't be touched by upgrades.

## Variants

Two per-cell template repos currently ship from the forge:

| Variant | Repo | What you get |
|---------|------|--------------|
| **`harness-claude-github`** *(this project)* | [evolutionary-leadership/harness-claude-github](https://github.com/evolutionary-leadership/harness-claude-github) | Feature branches + auto-merge, no deploy target |
| `harness-claude-github-railway` | [evolutionary-leadership/harness-claude-github-railway](https://github.com/evolutionary-leadership/harness-claude-github-railway) | + Railway preview environments per feature with isolated PostgreSQL and S3-compatible bucket |

Switching from `harness-claude-github` to
`harness-claude-github-railway` is not an automated migration; it
requires re-scaffolding from the new template and porting your
application code over.

## Upgrading (same variant)

Run `/harness-upgrade` to check for version updates within your current
variant. The skill uses structured migration files from the forge
(`evolutionary-leadership/harness-forge`) to show you exactly what
changed, filtered by your variant and installed traits. See
`.harness-version` for current version info.

### Version numbering

Harness versions use semver (`MAJOR.MINOR.PATCH`):
- **PATCH** bumps automatically on each feature merge to the forge's
  `dev` branch
- **MINOR** bumps are a developer decision for significant releases
- **MAJOR** is reserved for breaking architecture changes

## License

The Harness Companion is licensed under the **Apache License 2.0**.
See the `LICENSE` and `NOTICE` files in the root of this repository.

The NOTICE file must be preserved in any derivative works or forks.
It attributes this project to its origin:
[The Harness Companion](https://www.harnesscompanion.com)
by Evolutionary Leadership Coöperatie U.A.
