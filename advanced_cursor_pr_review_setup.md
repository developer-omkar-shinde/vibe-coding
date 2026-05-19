# Advanced Cursor PR Review Workflow

This setup creates a professional-grade PR review workflow using:

- Cursor rules
- Cursor commands
- Git worktrees
- GitHub CLI (`gh`)
- Automated shell scripts
- Shell aliases
- Architecture context files
- Specialized review commands

Goals:

- isolated PR review environments
- zero branch switching
- reusable AI review workflows
- better review consistency
- faster PR reviews
- cleaner git workflow
- architecture-aware AI reviews

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
- better review consistency

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
- Do not invent hypothetical issues without evidence from the code
- Do not suggest changes unless there is a clear benefit

## High Risk Areas

Pay extra attention to:
- auth flows
- payment flows
- shared hooks
- global state
- caching logic
- websocket logic
- API abstraction layers
- shared UI components
- analytics/tracking logic

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

# 2) Main Review Command

File:

```txt
.cursor/commands/review-pr.md
```

Purpose:

- reusable PR review command
- standardized review workflow

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

Inside Cursor:

```txt
/review-pr
```

---

# 3) Specialized Review Commands

## Performance Review

File:

```txt
.cursor/commands/review-performance.md
```

Example:

```md
Review this PR specifically for frontend performance concerns.

Focus on:
- unnecessary re-renders
- expensive computations
- memoization issues
- bundle size impact
- lazy loading
- image optimization
- API request duplication
- large list rendering
- unnecessary effects
- state management inefficiencies

Only report meaningful performance concerns.
```

Usage:

```txt
/review-performance
```

---

## Accessibility Review

File:

```txt
.cursor/commands/review-accessibility.md
```

Example:

```md
Review this PR specifically for accessibility issues.

Focus on:
- missing labels
- keyboard navigation
- focus handling
- semantic HTML
- screen reader support
- color contrast risks
- aria attributes
- interactive elements
- modal accessibility
- form accessibility

Only report real accessibility concerns.
```

Usage:

```txt
/review-accessibility
```

---

## Security Review

File:

```txt
.cursor/commands/review-security.md
```

Example:

```md
Review this PR specifically for security concerns.

Focus on:
- XSS risks
- unsafe HTML rendering
- auth issues
- token handling
- sensitive data exposure
- insecure API usage
- permission checks
- unsafe redirects
- local storage risks
- analytics/privacy leaks

Only report evidence-based security concerns.
```

Usage:

```txt
/review-security
```

---

# 4) Architecture Context Files

Purpose:

- teach Cursor your codebase architecture
- improve architecture-aware reviews
- improve consistency suggestions
- reduce incorrect review comments

Recommended files:

```txt
.cursor/context/frontend-architecture.md
.cursor/context/state-management.md
.cursor/context/api-patterns.md
.cursor/context/component-patterns.md
```

Example:

```md
# Frontend Architecture

## State Management
- Zustand for global state
- React Query for server state
- Local state preferred when possible

## Component Structure
- shared/ui for reusable UI components
- feature-based folder structure
- hooks separated by domain

## API Patterns
- centralized API client
- React Query hooks for data fetching
- avoid direct fetch calls in components
```

These files dramatically improve Cursor review quality.

---

# 5) Review Automation Script

File:

```txt
.cursor/scripts/review-pr.sh
```

Purpose:

- create isolated worktree
- fetch latest main branch
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

git fetch origin

if [ -d "$WORKTREE_PATH" ]; then
  git worktree remove "$WORKTREE_PATH" --force
fi

git worktree add "$WORKTREE_PATH" main

cd "$WORKTREE_PATH" || exit

gh pr checkout "$PR_NUMBER"

gh pr view "$PR_NUMBER"

git diff --stat main...HEAD

cursor -n .
```

Make executable:

```bash
chmod +x .cursor/scripts/review-pr.sh
```

---

# 6) Create Shell Alias

Open:

```txt
~/.zshrc
```

Add:

```bash
alias review-pr='/Users/prometteur/Documents/rebet/trivelta-webapp/.cursor/scripts/review-pr.sh'
alias pr-clean='review-pr --clean'
```

Reload:

```bash
source ~/.zshrc
```

---

# 7) Review Workflow

## Start Review

```bash
review-pr 2526
```

This automatically:

1. fetches latest main branch
2. removes old worktree if exists
3. creates isolated worktree
4. checks out PR branch
5. shows PR details
6. shows diff stats
7. opens Cursor in separate window

---

## Inside Cursor

Run:

```txt
/review-pr
```

Optional focused reviews:

```txt
/review-performance
/review-accessibility
/review-security
```

---

## Cleanup Review Workspace

```bash
pr-clean 2526
```

or:

```bash
review-pr --clean 2526
```

This removes:

- review worktree
- isolated review environment

---

# 8) Recommended Review Style

For large PRs:

Use focused review prompts.

Example:

```txt
/review-pr focus on:
- mobile responsiveness
- loading states
- API changes
```

Focused reviews usually produce much better AI review quality.

---

# 9) Benefits of This Setup

- No branch switching
- No git stash required
- Current work remains untouched
- Full repository context for Cursor
- Better review consistency
- Faster PR reviews
- Isolated review environments
- Architecture-aware reviews
- Cleaner git workflow
- Reusable review workflows
- Better signal-to-noise ratio in AI reviews

---

# 10) Final Workflow

## Start review

```bash
review-pr 2526
```

## Inside Cursor

```txt
/review-pr
```

## Optional focused reviews

```txt
/review-performance
/review-accessibility
/review-security
```

## Cleanup

```bash
pr-clean 2526
```

