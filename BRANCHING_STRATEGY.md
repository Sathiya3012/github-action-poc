# Branching Strategy and Fixing Diverged Histories

## The Problem

When you squash merge a promotional PR from `staging` to `main`, the commit history diverges:
- `main` gets a single squashed commit
- `staging` still has the original merge commits

This causes future promotional PRs to include changes that were already merged to `main`.

**Note:** Since `main` and `staging` have branch protection rules (no force push, no deletion), we use merge commits instead of rebasing.

## Solution

### Automatic Solution (Recommended)

A GitHub Actions workflow (`.github/workflows/rebase-staging.yml`) has been created that automatically merges `main` into `staging` after promotional PRs are merged. This syncs the branches without requiring force push.

**How it works:**
- After a promotional PR is merged to `main`, the workflow triggers
- It merges `main` into `staging` with a merge commit
- This brings `main`'s squashed commit into `staging`'s history
- Future promotional PRs will only show new commits added after this merge

**Requirements:**
- The workflow needs write permissions. Go to: Settings → Actions → General → Workflow permissions → Select "Read and write permissions"

### Manual Fix (For Current Situation)

If you need to fix the current diverged state manually:

```bash
# 1. Make sure you're on staging
git checkout staging
git pull origin staging

# 2. Merge main into staging (no force push needed!)
git fetch origin main
git merge origin/main -m "chore: sync staging with main after promotional PR merge"

# 3. Push the merge commit
git push origin staging
```

This creates a merge commit that brings `main`'s changes into `staging` without rewriting history.

### Understanding the Merge Strategy

When you merge `main` into `staging`:
- Git creates a merge commit that combines both histories
- The squashed commit from `main` is now in `staging`'s history
- When creating a new promotional PR, Git compares `staging` to `main`
- Since `staging` now includes `main`'s commit (via the merge), only new commits will appear in the PR

**Note:** The old commits (like `b1`) will still be in `staging`'s history, but they won't appear in the promotional PR diff because `main` already has those changes (in squashed form).

## Best Practices

1. **After squash merging a promotional PR:**
   - The workflow will automatically merge `main` into `staging`
   - Or manually merge if needed: `git checkout staging && git merge origin/main && git push`

2. **Before creating new feature branches:**
   - Always create from an up-to-date `staging` branch
   - `git checkout staging && git pull origin staging`

3. **Creating promotional PRs:**
   - After the sync merge, new promotional PRs will only show commits added after the last sync
   - This prevents duplicate changes from appearing in promotional PRs

