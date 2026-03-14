# Updating Your OpenClaw Fork

This guide shows how to keep your fork up to date with the main OpenClaw repository.

## 1. One-Time Setup

If you already cloned your fork, open it and add the original repo as `upstream`.

```bash
git remote -v
git remote add upstream https://github.com/openclaw/openclaw.git
git remote -v
```

Expected remotes:

- `origin` -> your fork
- `upstream` -> `openclaw/openclaw`

## 2. Keep Your Local `main` in Sync

Run this whenever you want the latest changes from OpenClaw:

```bash
git checkout main
git fetch upstream
git rebase upstream/main
```

If there are conflicts:

```bash
git status
# fix files manually
git add <file>
git rebase --continue
```

If you need to stop the rebase:

```bash
git rebase --abort
```

## 3. Update Your Fork on GitHub

After your local `main` is rebased to `upstream/main`, push it to your fork:

```bash
git push origin main --force-with-lease
```

Why force push? A rebase rewrites commit history, so `--force-with-lease` safely updates your fork.

## 4. Keep Feature Branches Current

When working on a branch (example: `my-feature`), update it from your synced `main`:

```bash
git checkout my-feature
git rebase main
```

Then push the updated branch:

```bash
git push origin my-feature --force-with-lease
```

## 5. Optional: GitHub UI "Sync fork"

You can also sync in the GitHub web UI:

1. Open your fork on GitHub
2. Go to branch `main`
3. Click `Sync fork`
4. Click `Update branch`

This updates your fork's `main` branch, but you may still want local `git fetch` + `git rebase` to keep your local repo clean.

## 6. Quick Command Set

If remotes are already configured, this is the standard refresh flow:

```bash
git checkout main
git fetch upstream
git rebase upstream/main
git push origin main --force-with-lease
```

## Notes for OpenClaw Contributions

- Create PRs from feature branches, not directly from `main`.
- Rebase your feature branch on top of latest `main` before opening or updating a PR.
- Use `--force-with-lease` rather than `--force` to avoid accidentally overwriting newer remote work.
