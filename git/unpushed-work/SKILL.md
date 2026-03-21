---
name: unpushed-work
description: Audit local branches and .worktrees checkouts to find work that has not been pushed to origin or merged, then present a clear list of branches or detached commits that exist only on the local machine.
---

# Orphaned Work Audit

## When to use this skill

Use this skill when the user wants to:

- verify that local work has been pushed to `origin`
- audit local branches before cleanup or machine migration
- inspect `.worktrees` for branches or detached commits that may only exist locally
- identify potentially orphaned work that is not safely backed up remotely

## Goal

Find every local branch and every branch or detached HEAD checked out under `.worktrees/`, determine whether that work is safely represented on `origin` or already merged, and show the user a final list of items that are still only local.

## Rules

- Prefer `gh` for remote and GitHub-aware operations when it is useful, but use `git` for local repository inspection.
- Never delete, reset, rebase, or modify branches during this audit.
- Never push automatically.
- Treat missing `origin` as a hard stop and report it clearly.
- Treat missing `.worktrees` as valid; continue with local branch checks.
- Deduplicate findings so the same branch is not shown twice.
- Show the user the final list even if it is empty.
- Be conservative: if you cannot prove the work exists remotely or is merged, include it in the list.

## High-level procedure

1. Verify the current directory is inside a Git repository.
2. Verify an `origin` remote exists.
3. Fetch remote refs from `origin` without changing local branches.
4. Audit all local branches in the main repository.
5. Audit each directory under `.worktrees/`.
6. Build a single list of branches or commits that are not safely represented on `origin`.
7. Present the results clearly.

## Detailed procedure

### 1) Repository preflight

Run:

```bash
git rev-parse --is-inside-work-tree
git remote get-url origin
git fetch origin --prune
```

If any of these fail:

- stop the audit
- explain exactly what is missing or broken ### 2) Audit all local branches in the main repository

Enumerate local branches:

```bash
git for-each-ref --format='%(refname:short)' refs/heads
```

For each local branch `<branch>`:

#### A. Check whether it has an upstream on origin

Try:

```bash
git rev-parse --abbrev-ref --symbolic-full-name "${branch}@{upstream}"
```

If upstream exists and points to `origin/...`, compare it with the local branch.

Get ahead/behind counts:

```bash
git rev-list --left-right --count "${branch}@{upstream}...${branch}"
```

Interpret carefully:

- if local is ahead of upstream, add `<branch>` to the orphan-risk list
- if local is only behind, it is not orphaned
- if local matches upstream exactly, it is safe

#### B. If no upstream exists, check whether an origin branch of the same name exists

Run:

```bash
git show-ref --verify --quiet "refs/remotes/origin/${branch}"
```

If `origin/<branch>` exists, compare:

```bash
git rev-list --left-right --count "origin/${branch}...${branch}"
```

Interpret carefully:

- if local branch contains commits not on `origin/<branch>`, add it to the list
- if local is fully contained in `origin/<branch>`, it is safe

#### C. If there is no matching remote branch, check whether the branch is already merged

Use a conservative merged check against likely integration branches that exist locally or remotely.

Candidate bases, in order:

- `origin/HEAD`
- `origin/main`
- `origin/master`
- `main`
- `master`

Resolve the first ones that exist, then test:

```bash
git merge-base --is-ancestor "${branch}" "<base>"
```

Interpret carefully:

- if the branch tip is an ancestor of a base branch, treat it as merged and safe
- otherwise add `<branch>` to the orphan-risk list

### 3) Audit `.worktrees/`

If `.worktrees/` does not exist, skip this section.

If it exists, loop over each direct child directory under `.worktrees/`.

For each worktree directory `<wt>`:

#### A. Verify it is a Git worktree

Run inside that directory:

```bash
git rev-parse --is-inside-work-tree
git rev-parse --git-dir
git rev-parse HEAD
git symbolic-ref -q --short HEAD
```

Capture:

- absolute or relative worktree path
- current `HEAD` commit
- current branch name, if any
- detached HEAD status

#### B. If the worktree is on a branch

Let the checked out branch be `<branch>`.

Check for remote presence and divergence using the same logic as the main repository audit:

- upstream check
- matching `origin/<branch>` check
- merged check against likely base branches

If the branch has commits not represented on `origin` and is not merged, add an entry that includes:

- worktree path
- branch name
- reason, such as:
  - `ahead of origin`
  - `no remote branch`
  - `not merged`

#### C. If the worktree is in detached HEAD state

Let the checked out commit be `<sha>`.

Determine whether that commit is reachable from any remote branch on `origin`:

```bash
git branch -r --contains "<sha>"
```

Interpret carefully:

- if any `origin/*` branch contains the commit, treat it as safe
- otherwise add the worktree to the orphan-risk list as a detached commit not present on `origin`

Also try to identify any local branch pointing at that commit:

```bash
git for-each-ref --format='%(refname:short) %(objectname)' refs/heads
```

If a matching local branch exists, include it in the report. If not, report it as detached-only local work.

### 4) Deduplication

Deduplicate by these keys:

- for normal branches: branch name
- for worktree branch entries: `path + branch name`
- for detached worktrees: `path + HEAD sha`

If the same branch appears in both the main repo and a worktree, keep one entry and enrich it with all observed locations.

### 5) Output format

Always present a final section named:

```md
## Branches or worktrees not safely pushed to origin
```

If there are no findings, say:

```md
No local-only branches or detached worktree commits were found. Everything checked is either pushed to origin or already merged.
```

If there are findings, present each item with:

- branch name or detached commit SHA
- location
- status
- evidence

Example format:

```md
- `feature/foo`
  - location: main repo
  - status: local commits not on `origin/feature/foo`
  - evidence: branch is ahead of remote

- `feature/bar`
  - location: `.worktrees/bar`
  - status: no remote branch and not merged
  - evidence: no `origin/feature/bar`; tip is not ancestor of `origin/main`

- detached `abc1234`
  - location: `.worktrees/experiment`
  - status: commit not contained in any `origin/*` branch
  - evidence: detached HEAD reachable only locally
```

## Recommended shell patterns

Use safe, read-only commands like:

```bash
git for-each-ref --format='%(refname:short)' refs/heads
git show-ref --verify --quiet "refs/remotes/origin/<branch>"
git rev-list --left-right --count "<left>...<right>"
git merge-base --is-ancestor "<candidate>" "<base>"
git branch -r --contains "<sha>"
git symbolic-ref -q --short HEAD
git rev-parse HEAD
```

## Things to avoid

Do not:

- run `git push`
- run `git branch -D`
- run `git reset --hard`
- rewrite history
- assume `.worktrees/` entries are on branches
- assume a missing upstream means the branch is safe or unsafe without checking merge/reachability

## Success criteria

This skill is complete only when the user is shown a consolidated list of:

- local branches with unpushed commits
- local branches with no corresponding remote branch and not merged
- worktree branches with unpushed or unmerged work
- detached worktree commits not reachable from any `origin/*` branch

The purpose of this skill is to make orphaned local work visible before it is lost.
