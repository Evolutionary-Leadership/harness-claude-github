---
name: continue
description: Resume work on an in-progress feature branch. Lists active features and lets you pick one to continue.
disable-model-invocation: true
argument-hint: "[optional: feature name to continue]"
allowed-tools: Bash(git *), Bash(gh *), Read, Glob, Grep
---

# Continue an in-progress feature

List active feature branches and resume working on one. Fetches the latest code
and shows a summary of what's been done so far.

## Steps

### 1. List active feature branches

    git fetch origin --prune
    git branch -r | grep 'origin/feature/' | sed 's|origin/||'

For each branch, show:
- Branch name
- Latest commit message and date
- Whether there's an open PR

    for branch in $(git branch -r | grep 'origin/feature/' | sed 's|origin/||'); do
      echo "$branch"
      git log "origin/$branch" -1 --format="  Last commit: %s (%cr)"
    done

    gh pr list --base dev --state open --json number,title,headRefName --jq '.[] | "  PR #\(.number): \(.title) (\(.headRefName))"'

### 2. Select a feature

If `$ARGUMENTS` is provided, match it against the feature names. Otherwise,
present the list and ask the user which feature to continue.

### 3. Fetch and checkout

    FEATURE="feature/<name>"
    git fetch origin "$FEATURE"
    git checkout -b "claude/<name>-<sessionId>" "origin/$FEATURE"

Where `<sessionId>` is the current session identifier suffix from the branch
name you're on, or generate a short random suffix.

### 4. Show work summary

Provide a summary of the feature's current state:

    git log origin/dev.."origin/$FEATURE" --oneline
    git diff origin/dev.."origin/$FEATURE" --stat

Show:
- How many commits ahead of dev
- Files changed
- Key changes (read the diff summary)
- Any open PR and its review status
- What appears to be left to do (based on PR description or commit messages)

### 5. Ready to work

Tell the user:
- You're now on a working branch for this feature
- Summarize the current state
- Ask what they'd like to work on next
