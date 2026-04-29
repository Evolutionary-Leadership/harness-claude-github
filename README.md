> Generated from `evolutionary-leadership/harness-forge@99d3818`. Do not edit here. Edit in the source repo.

# harness-claude-github

This is the Code Only template of the
[Harness Companion](https://www.harnesscompanion.com), the
`claude-code + github + no-deploy` cell.

It is aimed at projects that do not deploy to a cloud target:
libraries, CLIs, scripts, daemons, Electron apps, browser extensions,
and similar. You bring your own runtime; the harness provides the
opinionated Claude Code plus GitHub Actions feature flow on top.

## How to use

1. Click **Use this template** at the top of the GitHub repo page.
2. Give your new repo a name and pick its visibility.
3. Once the repo is created, follow the wizard at
   [www.harnesscompanion.com](https://www.harnesscompanion.com) to
   finish wiring up secrets and your local Claude Code setup.

## What you get

- A `dev` and `main` branch convention with auto-merge for features
  and a release flow that promotes `dev` to `main`.
- A `.claude/` directory with skills, hooks, and agents tuned for the
  feature lifecycle (`/feature`, `/mergedev`, `/review`, `/release`,
  and friends).
- GitHub Actions workflows that wrap the lifecycle.
- A starting `claude-md-snippet.md` to paste into your project's
  `CLAUDE.md`.

## Provenance

The contents of this repo are auto-generated from
[`evolutionary-leadership/harness-forge`](https://github.com/evolutionary-leadership/harness-forge).
Edits made directly here will be overwritten on the next sync.
File issues and send improvements upstream to harness-forge.
