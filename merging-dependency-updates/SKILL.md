---
name: merging-dependency-updates
description: Use when asked to process, rebase, merge, clear, or babysit open Dependabot, Renovate, or Snyk pull-request queues in a GitHub repository.
---

# Merging Dependency Updates

## Overview

Clear eligible bot dependency updates without racing the base branch. One controller owns every mutation; earlier snapshots and worker reports are evidence to refresh, never merge authorization.

## Approval Mode and Goal

An explicit request to process a named repository authorizes discovery, provider rebase requests, and polling. Approval-mode selection is the first operational action, before goal inspection, repository preflight, or mutation. If the user already chose a mode, record it without asking again; otherwise ask whether to approve each merge or automatically approve all eligible merges for this repository run.

- **Per-merge approval:** Do not inspect, create, reuse, update, complete, or block a goal for this run. Ask before each merge for that pull request's freshly verified head SHA.
- **Automatic approval:** The user's choice authorizes every eligible merge in the named repository for this run without another merge prompt. After recording that choice, establish a matching active goal before repository preflight.

1. Call the runtime's registered goal tool directly. In OMP, invoke `goal({op:"get"})`; do not probe `xd://goal` or conclude goal support is absent because that is not a mounted OMP device. `/goal` is OMP's user-facing slash command, not a substitute for calling an available goal tool.
2. Reuse a goal only when its objective explicitly covers the exact repository's dependency-update queue. If it is paused, call `goal({op:"resume"})`. Never replace, clear, or repurpose an unrelated unfinished goal.
3. If no unfinished goal exists, call `goal({op:"create", objective:"Process the <owner>/<repo> dependency-update queue to terminal outcomes under this skill."})`. Omit any token-budget field, and name the exact repository in the objective.
4. If the native goal tool is unavailable, or goal inspection/resume/creation fails, or an unrelated unfinished goal prevents startup, do not try guessed device paths or claim a literal `/goal` prompt created a goal. Tell the user and fall back to per-merge approval without beginning repository preflight. The user may create a repository-scoped goal with OMP's `/goal` command; verify it with a fresh goal-tool `get` before resuming automatic mode.

In automatic mode, keep the goal active while any PR is Ready, Pending, Needs rebase, or a Lockfile repair candidate, or has authorized repair work. Complete it only after the final live audit proves every candidate terminal. Follow the goal tool's own blocked-status rules.

Audit or status requests stay read-only. Stop mutations immediately if the user says pause or stop; doing so revokes automatic approval for the remainder of that run.

## Controller and Workers

Only the controller may comment, label, edit a rebase checkbox, or merge. Mutations are serial. Read-only discovery and polling may use a lightweight worker at low reasoning, preferring Luna, Grok 4.6, Haiku 4.5, then Flash 3.8 among models exposed by the active runtime. Skip unavailable families; use another available lightweight worker or keep the work in the controller rather than failing the run. Validate every worker result live before mutation.

## Preflight

Resolve the exact repository, default branch, merge method, branch/ruleset requirements, required approvals and checks, permissions, and current head SHAs. Identify candidates by authenticated Dependabot, Renovate, or Snyk author plus dependency-update evidence; labels, titles, and branch names alone do not prove identity.

Drafts and unverifiable authors are Blocked. Unknown check policy is Blocked; a confirmed policy with no required checks is known, not unknown.

## State Contract

| State | Observable predicate | Action |
| --- | --- | --- |
| Ready | Open, non-draft, cleanly mergeable, approvals satisfied, required checks successful, no observed test/verification failure | Refresh; request exact-head approval in per-merge mode or merge under automatic mode |
| Awaiting approval | Per-merge mode, Ready gates pass, but the user has not approved this PR at its current head SHA | Ask once; do not merge |
| Pending | Checks, mergeability, or acknowledged bot work is in progress | Poll to change or inactivity limit |
| Needs rebase | Conflicted or stale under repository policy | Read `references/provider-rebases.md`, then invoke its adapter |
| Lockfile repair candidate | Clean, no checks pending, and a terminal failure explicitly identifies a stale/frozen/out-of-sync lockfile or the documented locked-install/check command reproduces it, with no unrelated terminal failure | Read `references/lockfile-repairs.md` |
| Awaiting repair approval | Per-merge mode and a verified repair diff is ready for this PR's current source head | Ask for repair-push approval tied to the PR, source SHA, and lockfile-only diff; do not push |
| Clean failing | Clean, nothing pending, and a terminal failure that is ambiguous, unrelated to the lockfile, or otherwise ineligible for repair | Leave open; refresh after later merges |
| Blocked | Unsafe or ambiguous state, permission failure, exhausted attempts, or timeout | Leave open and report why |
| Gone | Merged, closed, or superseded externally | Record live outcome |

Neutral or skipped checks are not failures unless policy requires success. Cancelled, timed-out, action-required, startup-failure, and stale test conclusions are terminal failures.

## Queue Loop

1. Record the approval mode and complete automatic-mode goal startup or fallback as required above.
2. Refresh and classify the complete candidate queue.
3. Select one Ready PR.
4. Immediately reread its author, draft state, head SHA, mergeability, approvals, required checks, and all observed test/verification failures.
5. In per-merge mode, ask the user to approve that PR at the exact verified head SHA. Approval for another PR or head is invalid. If declined, classify it Blocked with `merge declined by user`; if unanswered, leave it Awaiting approval. In automatic mode, do not ask again.
6. Immediately before either mode merges, reread every gate. If the head or any gate changed, do not merge; return the PR to classification. Per-merge mode requires new approval after it is Ready again. Automatic mode may merge after a later fresh read passes every gate.
7. Merge using the repository's policy and the freshly verified head SHA as the expected-head guard.
8. Refresh every remaining candidate after the base changes.
9. Request provider rebases for Needs rebase PRs, poll Pending work, and repeat.

A Clean failing PR stays in later refreshes. If it becomes conflicted, rebase it; after fresh checks, merge it only if Ready.

When a Clean failing PR meets the lockfile repair evidence condition, classify it as a Lockfile repair candidate and follow the repair reference. Automatic mode authorizes only its narrowly scoped repair commit for the named repository and current run; per-merge mode requires separate repair-push approval. Repair approval never authorizes merge, and merge approval never authorizes repair. Keep an automatic-mode goal active while repair work or fresh checks remain.

In per-merge mode, when no Ready, Pending, Needs rebase, or authorized repair work remains but a PR is Awaiting merge or repair approval, report each request and hand control back to the user. For repair approval, include the PR, source SHA, and proposed lockfile-only diff. This is a nonterminal handoff; after the user's response, resume with a fresh classification before any mutation.

Never merge from cached state, bypass a stale-head rejection, mutate candidates concurrently, repair failing code, force checks to pass, dismiss reviews, change protection settings, or manually force-push a bot branch. Lockfile regeneration is limited to the evidence-backed workflow in `references/lockfile-repairs.md`.

## Limits and Ambiguity

Allow at most three rebase requests without progress for one unchanged head. Do not duplicate an acknowledged in-progress request. Apply a 15-minute no-progress limit to Pending checks, mergeability, and acknowledged bot work unless repository documentation defines another timeout. Measure it from the last observable state change; when it expires, refresh once and classify the unchanged PR Blocked rather than polling forever.

After a timeout, lost response, or other ambiguous mutation result, reread live state before deciding whether to retry. PR-level Blocked is a terminal queue outcome, not overall-goal failure. In per-merge mode, an Awaiting repair approval is a nonterminal handoff. After a repair push, classify Pending until fresh checks finish. If every candidate is Merged, Clean failing, Blocked, or Gone, complete the run and report those outcomes. In automatic mode, mark the overall goal blocked only when the goal tool's own repeated-blocker rule applies and the final terminal audit cannot be completed.

## Completion Report

Finish only after a final live candidate search. Report merged PRs with provider, final SHA, merge result, and passing evidence; repaired PRs with repair commit SHA and lockfile evidence; Clean failing PRs and their failures; Blocked PRs and exact reasons; and externally Gone PRs. A final audit containing Blocked PRs still completes the run. Awaiting merge or repair approval is not terminal and produces an interim handoff instead of this completion report. Distinguish a cleared queue from a completed run that intentionally left failing or blocked PRs open.
