> Generated from `evolutionary-leadership/harness-forge@eb55633`. Do not edit here. Edit in the source repo.

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
  | `/chat` | Talk it through; nothing is written. |
  | `/brainstorm` | Grill an idea relentlessly; keep it as a tracker issue, or drop it. |
  | `/feature` | Build a feature through five gated phases: grill, spec, tickets, implement, hand over. |
  | `/mergedev` | Open a PR into `dev` and auto-merge it. |
  | `/review` | Open a PR for human review instead of auto-merging. |
  | `/code-review` | Agent review of the diff on two axes: Standards and Spec. |
  | `/release` | Promote `dev` to `main`, tag, and cut a GitHub Release. |
  | `/status` | Dashboard of active features, open PRs, and unreleased changes. |
  | `/document` | Write a decision record, audit docs against your diff, or find the one home for a fact. |
  | `/rollback`, `/changelog`, `/deps`, `/continue` | Revert a release, generate notes, batch Dependabot PRs, resume work. |

  Run `/getting-started` any time to list them all, including the
  technique skills the flow chains (`/grilling`, `/tdd`, `/to-spec`,
  `/to-tickets`, `/implement`, `/diagnosing-bugs`, and more), adapted from
  [mattpocock/skills](https://github.com/mattpocock/skills) (MIT) and
  owned by the harness.
- **A documentation layout built for AI readers.** Nearly every reader of
  your docs is an agent starting a fresh session with no memory, and
  `CLAUDE.md` is the only part that loads automatically, every time. So the
  scaffold gives you `docs/README.md` as an index-manifest (every doc, what
  it owns), `docs/architecture/` for per-subsystem catalogs that declare
  which source files they describe, `docs/decisions/` for numbered decision
  records, `docs/runbooks/`, and skeletons for the glossary, security, and
  testing docs. `scripts/check-docs.mjs` is a zero-dependency checker that
  fails on broken links, unindexed docs, stale references, and surface
  tables that no longer match the code. Put it in your `check:` line and
  broken docs block auto-merge exactly like a type error. The `docs-updater`
  agent runs the same rules automatically on `/mergedev` and `/review`.
- **GitHub Actions workflows** that wrap the lifecycle, plus Dependabot
  preconfigured to keep your Actions current.
- **A starter `claude-md-snippet.md`** to paste into your project's
  `CLAUDE.md` so Claude knows the workflow from day one.

## How a team works in this repo

Every session opens by stating what it is: **chat** (talk, write
nothing), **brainstorm** (stress-test an idea; at most an idea issue on
the tracker), or **feature** (build, through gated phases). A feature is
grilled before it is built: `/feature` interviews you until the design
tree has no open questions, publishes a spec issue, slices it into
blocking-ordered tickets, implements ticket by ticket test-first, runs an
agent code review, and only then asks how you want it merged: `/mergedev`
(auto-merge) or `/review` (a PR your teammates approve, landed afterwards
with `/mergedev`).

The whole way through, the feature's state and reasoning live in a
committed **feature context** file on the feature branch. Stop any time,
on any day; a colleague runs `/continue`, reads the context beside the
branch list, and lands mid-flow with the decisions, rejections, and next
step intact. At merge time the context feeds the PR description and the
docs audit, then disappears; what deserves to outlive the feature is
promoted into `docs/` where the documentation standard owns it.

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
