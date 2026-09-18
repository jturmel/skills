---
name: merging-dependency-updates
description: Use when asked to process, rebase, merge, clear, or babysit open Dependabot, Renovate, or Snyk pull-request queues in a GitHub repository.
---

# Merging Dependency Updates

## Overview

Clear eligible bot dependency updates without racing the base branch. One controller owns every mutation; earlier snapshots and worker reports are evidence to refresh, never merge authorization.

## Authorization and Goal

An explicit request to process a named repository authorizes the complete loop without per-merge confirmation. Audit or status requests stay read-only. Stop mutations immediately if the user says pause or stop.

For an authorized run, inspect goal state when goal tooling exists. Create one repository-scoped goal when none exists, with no token budget; its objective is to reach the final audited terminal state defined by this skill while preserving protections and leaving failing PRs unrepaired. Reuse a matching goal. Never replace an unrelated unfinished goal; continue normally and report that goal mode could not start. Continue normally if goals are unavailable. Goal mode adds persistence, not permission.

Keep the goal active while any PR is Ready, Pending, or Needs rebase. Complete it only after the final live audit proves every candidate terminal. Follow the goal tool's own blocked-status rules.

## Controller and Workers

Only the controller may comment, label, edit a rebase checkbox, or merge. Mutations are serial. Read-only discovery and polling may use an available lightweight worker at low reasoning, preferring Luna, Grok 4.6, Haiku 4.5, then Flash 3.8. Validate every worker result live before mutation.

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

1. Refresh and classify the complete candidate queue.
2. Select one Ready PR.
3. Immediately reread its author, draft state, head SHA, mergeability, approvals, required checks, and all observed test/verification failures.
4. Merge only if every gate still passes, using the repository's merge policy and the exact head SHA as an expected-head guard.
5. Refresh every remaining candidate after the base changes.
6. Request provider rebases for Needs rebase PRs, poll Pending work, and repeat.

A Clean failing PR stays in later refreshes. If it becomes conflicted, rebase it; after fresh checks, merge it only if Ready.

Never merge from cached state, bypass a stale-head rejection, mutate candidates concurrently, repair failing code, regenerate lockfiles, force checks to pass, dismiss reviews, change protection settings, or manually force-push a bot branch.

## Limits and Ambiguity

Allow at most three rebase requests without progress for one unchanged head. Do not duplicate an acknowledged in-progress request. Block checks after one hour without observable progress unless repository documentation defines another timeout.

After a timeout, lost response, or other ambiguous mutation result, reread live state before deciding whether to retry. A single blocked PR does not make the overall goal blocked while other candidates can progress.

## Completion Report

Finish only after a final live candidate search. Report merged PRs with provider, final SHA, merge result, and passing evidence; Clean failing PRs and their failures; Blocked PRs and exact reasons; and externally Gone PRs. Distinguish a cleared queue from a completed run that intentionally left failing or blocked PRs open.
