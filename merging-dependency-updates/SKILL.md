---
name: merging-dependency-updates
description: Use when asked to process, rebase, merge, clear, or babysit open Dependabot, Renovate, or Snyk pull-request queues in a GitHub repository.
---

# Merging Dependency Updates

## Overview

Clear eligible bot dependency updates without racing the base branch. One controller owns every mutation; earlier snapshots and worker reports are evidence to refresh, never merge authorization.

## Authorization and Goal

An explicit request to process a named repository authorizes the complete loop without per-merge confirmation. Audit or status requests stay read-only. Stop mutations immediately if the user says pause or stop.

For every authorized run, goal startup is the first action:

1. Inspect goal state when goal tooling exists.
2. If an unrelated unfinished goal exists, do not create, replace, or clear a goal. Record that goal mode could not start and continue normally.
3. Otherwise, if a matching goal exists, reuse it.
4. Otherwise, create one repository-scoped goal with no token budget; its objective is to reach the final audited terminal state defined by this skill while preserving protections and leaving failing PRs unrepaired.
5. If goal tooling is unavailable, record that fact and continue normally.

Do not begin repository preflight until this goal-start decision is recorded. Goal mode adds persistence, not permission.

Keep the goal active while any PR is Ready, Pending, or Needs rebase. Complete it only after the final live audit proves every candidate terminal. Follow the goal tool's own blocked-status rules.

## Controller and Workers

Only the controller may comment, label, edit a rebase checkbox, or merge. Mutations are serial. Read-only discovery and polling may use a lightweight worker at low reasoning, preferring Luna, Grok 4.6, Haiku 4.5, then Flash 3.8 among models exposed by the active runtime. Skip unavailable families; use another available lightweight worker or keep the work in the controller rather than failing the run. Validate every worker result live before mutation.

## Preflight

Resolve the exact repository, default branch, merge method, branch/ruleset requirements, required approvals and checks, permissions, and current head SHAs. Identify candidates by authenticated Dependabot, Renovate, or Snyk author plus dependency-update evidence; labels, titles, and branch names alone do not prove identity.

Drafts and unverifiable authors are Blocked. Unknown check policy is Blocked; a confirmed policy with no required checks is known, not unknown.

## State Contract

| State | Observable predicate | Action |
| --- | --- | --- |
| Ready | Open, non-draft, cleanly mergeable, approvals satisfied, required checks successful, no observed test/verification failure | Refresh and merge one |
| Pending | Checks, mergeability, or acknowledged bot work is in progress | Poll to change or inactivity limit |
| Needs rebase | Conflicted or stale under repository policy | Read `references/provider-rebases.md`, then invoke its adapter |
| Clean failing | Clean, nothing pending, at least one terminal test/verification failure | Leave open; refresh after later merges |
| Blocked | Unsafe or ambiguous state, permission failure, exhausted attempts, or timeout | Leave open and report why |
| Gone | Merged, closed, or superseded externally | Record live outcome |

Neutral or skipped checks are not failures unless policy requires success. Cancelled, timed-out, action-required, startup-failure, and stale test conclusions are terminal failures.

## Queue Loop

1. Complete the goal-start decision required above.
2. Refresh and classify the complete candidate queue.
3. Select one Ready PR.
4. Immediately reread its author, draft state, head SHA, mergeability, approvals, required checks, and all observed test/verification failures.
5. Merge only if every gate still passes, using the repository's merge policy and the exact head SHA as an expected-head guard.
6. Refresh every remaining candidate after the base changes.
7. Request provider rebases for Needs rebase PRs, poll Pending work, and repeat.

A Clean failing PR stays in later refreshes. If it becomes conflicted, rebase it; after fresh checks, merge it only if Ready.

Never merge from cached state, bypass a stale-head rejection, mutate candidates concurrently, repair failing code, regenerate lockfiles, force checks to pass, dismiss reviews, change protection settings, or manually force-push a bot branch.

## Limits and Ambiguity

Allow at most three rebase requests without progress for one unchanged head. Do not duplicate an acknowledged in-progress request. Apply a one-hour no-progress limit to Pending checks, mergeability, and acknowledged bot work unless repository documentation defines another timeout. Measure it from the last observable state change; when it expires, refresh once and classify the unchanged PR Blocked rather than polling forever.

After a timeout, lost response, or other ambiguous mutation result, reread live state before deciding whether to retry. PR-level Blocked is a terminal queue outcome, not overall-goal failure. If every candidate is Merged, Clean failing, Blocked, or Gone, complete the run and report those outcomes. Mark the overall goal blocked only when the goal tool's own repeated-blocker rule applies and the final terminal audit cannot be completed.

## Completion Report

Finish only after a final live candidate search. Report merged PRs with provider, final SHA, merge result, and passing evidence; Clean failing PRs and their failures; Blocked PRs and exact reasons; and externally Gone PRs. A final audit containing Blocked PRs still completes the run. Distinguish a cleared queue from a completed run that intentionally left failing or blocked PRs open.
