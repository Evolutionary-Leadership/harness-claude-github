> Generated from `evolutionary-leadership/harness-forge@b9d1791`. Do not edit here. Edit in the source repo.

# harness-claude-github

**Talk to an AI, ship a feature. The branches, PRs, reviews, and releases
take care of themselves.**

This is the **Code Only** template of the
[Harness Companion](https://www.harnesscompanion.com), the
`claude-code + github + no-deploy` cell. It wraps [Claude
Code](https://www.anthropic.com/claude-code) and GitHub Actions into one
opinionated feature workflow, so the only thing you have to think about is
the code you want written.

It is built for projects that do not deploy to a cloud target: libraries,
CLIs, scripts, daemons, Electron apps, browser extensions, and the like.
You bring your own runtime; the harness brings the workflow on top.
(Building a web app that needs a live preview per feature? Use the
[Railway cell](https://github.com/Evolutionary-Leadership/harness-claude-github-railway)
instead.)

## The idea in one diagram

You describe the work in plain language. A branch-naming convention and a
few short prompts drive the rest:

```
claude/<codename>-<id>      Claude Code pushes here
        |
        v   GitHub Actions (triggered by the branch name)
feature/<name>              your feature, named after the work
        |
        v   /mergedev
dev                         PR auto-created and auto-merged, branch cleaned up
        |
        v   /release
main                        version tagged, changelog written, GitHub Release cut
```

No per-feature setup, no manual branch wrangling, no leftover branches to
sweep up afterward.

## Get started

1. Click **Use this template** at the top of this repo's GitHub page.
2. Give your new repo a name and pick its visibility.
3. Follow the wizard at
   [harnesscompanion.com](https://www.harnesscompanion.com) to wire up
   your GitHub token and local Claude Code setup.

Then open Claude Code in the new repo and just describe what you want to
build. The first session sets everything in motion.

## What you get

- **A `dev` and `main` branch flow** with auto-merge for features and a
  release flow that promotes `dev` to `main`, tags the version, and writes
  the changelog for you.
- **Feature branches named after the work, not a random codename.** Claude
  derives a kebab-case slug from your task and runs
  `bash .claude/scripts/set-feature-name.sh <slug>` before its first push;
  that slug (stored in `.harness-feature`) becomes `feature/<name>`. If
  naming is skipped, the first push falls back to the codename. See
  `.claude/HARNESS.md` ("Feature naming") for the resolver and fallback
  rules.
- **A `.claude/` toolkit** of skills, hooks, and agents tuned for the
  feature lifecycle. Drive it with short commands:

  | Command | What it does |
  |---|---|
  | `/feature` | Start a feature (optional; a new session auto-initializes one). |
  | `/mergedev` | Open a PR into `dev` and auto-merge it. |
  | `/review` | Open a PR for human review instead of auto-merging. |
  | `/release` | Promote `dev` to `main`, tag, and cut a GitHub Release. |
  | `/status` | Dashboard of active features, open PRs, and unreleased changes. |
  | `/rollback`, `/changelog`, `/deps`, `/continue` | Revert a release, generate notes, batch Dependabot PRs, resume work. |

  Run `/getting-started` any time to list them all.
- **GitHub Actions workflows** that wrap the lifecycle, plus Dependabot
  preconfigured to keep your Actions current.
- **A starter `claude-md-snippet.md`** to paste into your project's
  `CLAUDE.md` so Claude knows the workflow from day one.

## Make it yours

The harness ships opinionated defaults, and they are all yours to change.
Skills are markdown files you can edit or add to, workflows are plain YAML,
and your `CLAUDE.md` is never touched by the harness. When upstream ships
improvements, run `/harness-upgrade` to review and adopt them; your custom
skills and settings are preserved. `.claude/HARNESS.md` documents exactly
which files the harness manages.

## Provenance

The contents of this repo are auto-generated from
[`evolutionary-leadership/harness-forge`](https://github.com/evolutionary-leadership/harness-forge).
Edits made directly here will be overwritten on the next sync. File issues
and send improvements upstream to harness-forge.
