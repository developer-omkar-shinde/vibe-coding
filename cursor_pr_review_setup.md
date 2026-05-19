# Cursor PR Review Setup

This setup creates a complete automated PR review workflow using:

- Cursor rules
- Cursor commands
- Git worktrees
- GitHub CLI (`gh`)
- Review automation scripts
- Shell aliases

The goal is:

- isolated PR review environments
- no branch switching
- no git stash usage
- reusable Cursor review prompts
- automated PR review setup and cleanup

---

# 1) Cursor Rule File

File:

```txt
.cursor/rules/pr-review.mdc
```

Purpose:

- persistent PR review behavior
- automatically applied in Cursor
- reusable review standards

Example:

```md
---
description: Frontend PR review guidelines
alwaysApply: false
---

You are a senior software engineer reviewing pull requests.

## Review Goals

Review for:

- bugs
- regressions
- edge cases
- readability
- consistency
- unnecessary complexity
- performance issues
- accessibility issues
- security concerns

## Review Rules

- Prefer simple solutions over clever abstractions
- Match existing repository patterns and conventions
- Keep components small and focused
- Avoid unnecessary re-renders
- Avoid premature optimization
- Point out risky code paths
- Point out missing test coverage
- Inline CSS is not allowed
- Avoid duplicated logic
- Avoid dead code
- Avoid unnecessary comments
- Do not suggest large refactors unless necessary
- Do not praise unnecessarily
- Only provide actionable feedback

## React / Frontend Rules

- Prefer functional components
- Prefer hooks over class components
- Avoid prop drilling when existing patterns solve it
- Verify dependency arrays in hooks
- Check memoization usage carefully
- Check loading/error/empty states
- Check responsive behavior
- Check accessibility attributes
- Check keyboard interactions
- Check API error handling
- Check for race conditions in async logic

## PR Context Rules

To get full context:

1. Check the local repository
2. Identify the feature branch from the PR
3. Compare changes against the target branch
4. Inspect related files when necessary
5. Review affected shared components/utilities

Local repositories:

- trivelta-webapp: /Users/prometteur/Documents/rebet/trivelta-webapp
- trivelta-mobile-frontend: /Users/prometteur/Documents/rebet/trivelta-mobile-frontend
- trivelta-backend-services: /Users/prometteur/Documents/rebet/trivelta-backend-services
- trivelta-pam-frontend: /Users/prometteur/Documents/rebet/trivelta-pam-frontend

## Output Format

### Critical Issues

Only include:

- bugs
- regressions
- broken behavior
- security concerns
- major performance risks

### Improvements

Include:

- readability
- maintainability
- consistency
- test coverage
- architecture concerns

### Optional Suggestions

Include only non-blocking suggestions.

### Final Summary

Provide:

- overall risk level
- key concerns
- whether the PR is safe to merge
```

---

# 2) Cursor Command File

File:

```txt
.cursor/commands/review-pr.md
```

Purpose:

- reusable Cursor review command
- allows running `/review-pr`

Example:

```md
Review the current pull request thoroughly using repository context.

Steps:

1. Inspect all changed files
2. Inspect nearby related files when necessary
3. Compare against existing repository patterns
4. Identify regressions and risky logic
5. Check edge cases and error handling
6. Check accessibility concerns
7. Check loading, empty, and error states
8. Check performance concerns
9. Check test coverage and missing tests
10. Check for unnecessary complexity

Important rules:

- Only provide actionable comments
- Do not praise unnecessarily
- Avoid nitpicks unless impactful
- Prefer simple and maintainable solutions
- Match existing repository conventions

Use:

- current git branch
- local repository context
- repository rules from `.cursor/rules/pr-review.mdc`

Return output in this format:

## Critical Issues

## Improvements

## Optional Suggestions

## Final Summary
```

Inside Cursor chat:

```txt
/review-pr
```

---

# 3) Review Script

File:

```txt
.cursor/scripts/review-pr.sh
```

Purpose:

- automate worktree creation
- checkout PR automatically
- open Cursor automatically
- cleanup worktrees automatically

Example:

```bash
#!/bin/bash

if [ "$1" = "--clean" ]; then
  PR_NUMBER=$2

  if [ -z "$PR_NUMBER" ]; then
    echo "Usage: review-pr --clean <pr-number>"
    exit 1
  fi

  WORKTREE_PATH="$HOME/Documents/review-$PR_NUMBER"

  git worktree remove "$WORKTREE_PATH" --force

  echo "Removed worktree: $WORKTREE_PATH"

  exit 0
fi

PR_NUMBER=$1

if [ -z "$PR_NUMBER" ]; then
  echo "Usage: review-pr <pr-number>"
  exit 1
fi

REPO_ROOT=$(git rev-parse --show-toplevel)

WORKTREE_PATH="$HOME/Documents/review-$PR_NUMBER"

cd "$REPO_ROOT" || exit

git worktree add "$WORKTREE_PATH" main

cd "$WORKTREE_PATH" || exit

gh pr checkout "$PR_NUMBER"

cursor .
```

Make executable:

```bash
chmod +x .cursor/scripts/review-pr.sh
```

---

# 4) Create Shell Alias

Add this to:

```txt
~/.zshrc
```

```bash
alias review-pr='/Users/prometteur/Documents/rebet/trivelta-webapp/.cursor/scripts/review-pr.sh'
```

Reload zsh:

```bash
source ~/.zshrc
```

---

# 5) Start PR Review

Run:

```bash
review-pr 2526
```

This automatically:

1. Creates isolated git worktree
2. Checks out PR branch
3. Opens Cursor in review workspace

Then inside Cursor chat:

```txt
/review-pr
```

---

# 6) Cleanup PR Review Workspace

After finishing review:

```bash
review-pr --clean 2526
```

This automatically:

- removes the review worktree
- cleans isolated review workspace

No manual git worktree commands needed.

---

# 7) Benefits of This Setup

- No branch switching
- No git stash required
- Current work remains untouched
- Full repository context for Cursor
- Reusable PR review workflow
- Consistent review quality
- Isolated review environments
- Faster PR reviews
- Cleaner git workflow

---

# 8) Final Workflow

Start review:

```bash
review-pr 2526
```

Inside Cursor:

```txt
/review-pr
```

Cleanup:

```bash
review-pr --clean 2526
```
