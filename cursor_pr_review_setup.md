# Cursor PR Review Setup

This document captures the full workflow for reviewing GitHub pull requests in Cursor using:

- Cursor rules
- Cursor command
- Git worktrees
- GitHub CLI (`gh`)
- A small shell script
- A shell alias

---

## 1) Cursor rule file

**File:** `.cursor/rules/pr-review.mdc`

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

## 2) Cursor command file

**File:** `.cursor/commands/review-pr.md`

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

Inside Cursor chat, run:

```txt
/review-pr
```

---

## 3) Review script

**File:** `.cursor/scripts/review-pr.sh`

```bash
#!/bin/bash

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

Make it executable:

```bash
chmod +x .cursor/scripts/review-pr.sh
```

---

## 4) Shell alias

Add this to `~/.zshrc`:

```bash
alias review-pr='.cursor/scripts/review-pr.sh'
```

Reload your shell:

```bash
source ~/.zshrc
```

Now you can run:

```bash
review-pr 2526
```

---

## 5) Full workflow

Run this command from inside the repo:

```bash
review-pr 2526
```

It will:

1. Create a separate git worktree
2. Checkout the PR branch with `gh pr checkout`
3. Open Cursor in that worktree

Then inside Cursor chat:

```txt
/review-pr
```

---

## 6) Cleanup after review

Remove the worktree when done:

```bash
git worktree remove ~/Documents/review-2526
```

Force remove if needed:

```bash
git worktree remove ~/Documents/review-2526 --force
```

---

## 7) Why this setup is useful

- No branch switching in your main worktree
- No stash required
- Your current work stays untouched
- Cursor gets full local repository context
- Reviews become repeatable and consistent
- The same workflow works for every PR
