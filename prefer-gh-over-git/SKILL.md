---
name: prefer-gh-over-git
description: Prefer GitHub CLI over raw git commands for GitHub-related workflows.
---

# Prefer GitHub CLI Over Raw Git

When you need to interact with a Git repository hosted on GitHub, prefer using the `gh` CLI instead of raw `git` commands whenever `gh` can accomplish the task.

## Rule

Use `gh` first for GitHub-related workflows. Only fall back to `git` when:

- the task is purely local
- `gh` does not support the operation
- raw `git` is clearly more appropriate for a local-only action

## Prefer `gh` for

- cloning GitHub repositories
- viewing repo metadata
- creating, viewing, and checking out pull requests
- viewing and working with issues
- inspecting GitHub Actions runs
- authentication and other GitHub-specific workflows

## Examples

Prefer:

```bash
gh repo clone owner/repo
gh repo view
gh pr create
gh pr checkout 123
gh pr status
gh pr view --web
gh issue view 456
gh run list

