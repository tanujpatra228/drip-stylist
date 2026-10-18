# Git Workflow

`main` is production. Every merge to `main` deploys to Cloudflare Workers.

## Rules

1. Never commit directly to `main`.
2. One branch per feature or fix, created from the latest `main`.
3. Merge into `main` only through a pull request with green CI.
4. Delete the branch right after it is merged.

## Branch names

| Prefix | Use | Example |
| --- | --- | --- |
| `feat/` | New feature | `feat/dress-me` |
| `fix/` | Bug fix | `fix/tag-colour-parsing` |
| `chore/` | Tooling, deps, config | `chore/deploy-workflow` |
| `docs/` | Documentation only | `docs/launch-plan` |

Use short, lowercase, hyphenated names.

## Day to day

```bash
# 1. Start from the latest main
git checkout main
git pull origin main
git checkout -b feat/closet-upload

# 2. Work and commit in small steps
git add .
git commit -m "Add bulk photo upload to R2"

# 3. Push and open a pull request into main
git push -u origin feat/closet-upload

# 4. Keep the branch current if main moved
git fetch origin
git merge origin/main

# 5. After the PR is merged, clean up
git checkout main
git pull origin main
git branch -d feat/closet-upload
```

The remote branch is deleted automatically on merge (see settings below).

## Pull requests

- Keep PRs small: one feature or fix, ideally under 400 changed lines.
- Title says what changes: "Add swipe review queue for tags".
- CI (typecheck, tests, build) must pass before merging.
- Use **Squash and merge** so `main` has one clean commit per feature.

## Hotfixes

Same flow, branch prefix `fix/`, highest priority review. Never patch production by hand.

## One-time GitHub settings

In **Settings**:

- **General → Default branch:** `main`.
- **General → Pull Requests:** allow squash merging only, and enable *Automatically delete head branches*.
- **Rules → Rulesets → New branch ruleset** targeting `main`:
  - Require a pull request before merging.
  - Require status checks to pass (the CI job, once it exists).
  - Block force pushes and deletions.
