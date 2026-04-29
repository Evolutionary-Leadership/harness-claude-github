# CLAUDE.md Snippet: Feature Development Workflow

Add the following to your project's `CLAUDE.md`. Adapt project-specific details.

---

## Harness infrastructure

This project's CI/CD was set up by the
[harness-forge](https://github.com/Evolutionary-Leadership/harness-forge)
harness. Read `.claude/HARNESS.md` for details on which files are
harness-managed (don't edit; they get overwritten on upgrade) and how to
extend the setup.

## Writing rules

- Never use em dashes (U+2014). Use commas, colons, semicolons, or parentheses instead. A PreToolUse hook will block any write containing an em dash

## Feature development workflow

The full lifecycle from idea to merged feature is automated via GitHub Actions.

### 1. Starting a new feature

Every new chat on a `claude/` branch is automatically treated as a new
feature, no `/feature` prefix needed. Just describe what you want to build.

On session start, the harness automatically:
1. Pushes an init commit to trigger the GitHub Action
2. The Action derives the feature name (strip `claude/` prefix and
   `-<sessionId>` suffix) and creates `feature/<name>` from dev

You can still use `/feature <description>` explicitly if you prefer, but
it's no longer required.

### 2. Pushing code

Push to the `claude/` branch. The GitHub Action merges it into the feature
branch and deletes the source claude/ branch.

### 3. Merging to dev

Use `/mergedev` or say "merge to dev". This writes `.pr-description.md`,
commits, and pushes. The GitHub Action creates a PR and auto-merges it.

### 3b. Submitting for review (instead of auto-merge)

Use `/review` to create a PR without auto-merge. The PR stays open for
team review. Reviewers are assigned from `.harness-version` if configured.

### 4. Automatic cleanup

When auto-merge succeeds, `claude-to-feature-branch.yml` deletes the source
`claude/` branch. The PR merge then triggers `feature-merge-cleanup.yml`,
which deletes the feature branch.

**Gotcha:** Don't push to a merged branch. After `/mergedev`, both branches
are deleted remotely. Pushing again re-creates everything from scratch.

**`/release` after `/mergedev` in the same chat is fine.** The release skill
works on `dev` (stash, switch, commit, push, return) and never pushes the
`claude/` branch, so it does not re-trigger feature branch creation. No
need to start a new chat for a release.

## Releasing to production

Use `/release` (with optional `major`, `minor`, or `patch` argument) to ship
dev to production. This creates a release PR from `dev` → `main`, tags the
version, and generates a GitHub Release with notes. For emergencies, use
`/hotfix` to go directly from main with a fast-track patch release.

## CI checks

Configure CI checks by adding a `check:` field to `.harness-version`:

```
check: npm test && npm run lint
```

When set, PRs to dev (and main) run the check command, and merges wait for
checks to pass. See `.claude/HARNESS.md` for prerequisites.

## Team configuration

Optional `.harness-version` fields:

```
reviewers: teammate1, teammate2
check: npm test && npm run lint
```

## Available skills

Run `/getting-started` to see all skills, or use these directly:
- `/feature`: start a new feature (optional; auto-initializes on session start)
- `/mergedev`: merge to dev (auto-merge)
- `/review`: submit PR for team review
- `/release`: ship dev to production
- `/hotfix`: emergency production fix
- `/status`: team dashboard
- `/changelog`: generate changelog
- `/deps`: handle Dependabot PRs
- `/continue`: resume in-progress feature
- `/rollback`: revert bad deploy
- `/chat`: think and brainstorm without modifying the repo (the session-start
  hook still creates an empty `feature/<name>` branch on GitHub; pair with
  `/endchat` to clean up)
- `/endchat`: clean up after `/chat` (deletes the orphaned `feature/<name>`
  branch and switches local back to `dev`)

## Dependency management

Dependabot is configured in `.github/dependabot.yml` to automatically check
for outdated dependencies and open PRs to update them. When you add a new
package ecosystem to the project (e.g., npm, pip, Docker, Bundler), add a
corresponding entry to `.github/dependabot.yml` so Dependabot monitors it.


## Stack best practices

If you have installed managed traits via the harness, add this line:

```
Read `.claude/traits/` for stack-specific best practices before writing code.
```

Trait files in `.claude/traits/` are managed by the harness and updated via
`/harness-upgrade`. Configure which traits to track in `.harness-version`:

```
traits: nodejs, typescript, express, vitest, eslint, pnpm
```
