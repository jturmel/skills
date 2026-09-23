# Merging Dependency Updates Skill Design

## Purpose

Create a reusable `merging-dependency-updates` skill that processes a GitHub repository's open dependency-update pull requests from Dependabot, Renovate, and Snyk. At the start of an authorized run, the user chooses per-merge approval or automatic approval for all eligible merges. The skill then discovers, classifies, rebases, polls, and processes the queue until no actionable pull request remains.

The skill leaves clean pull requests with terminal failing checks open unless terminal-check evidence identifies a stale lockfile under the narrow repair workflow below. It never repairs application code, overrides checks, or treats clean failures as blockers to completing the queue run.

## Supported Scope

The first version supports GitHub pull requests authored by verified Dependabot, Renovate, or Snyk bot identities. It does not process GitLab merge requests, dependency issues without pull requests, or human-authored dependency pull requests.

Candidate detection must use the authenticated pull-request author identity plus dependency-update evidence from the pull request. Labels, title prefixes, or branch names alone are insufficient because repositories can apply or imitate them.

Draft pull requests are not merge candidates. They remain open and appear in the final blocked report.

## Authorization Boundary

An explicit user request to process dependency updates in a named repository authorizes discovery, provider rebase requests, and polling for that repository. Approval-mode selection is the first operational action, before goal inspection, repository preflight, or mutation. If the user did not already choose one, ask whether to approve each merge or automatically approve all eligible merges for this repository run.

In per-merge mode, approval is tied to one pull request's freshly verified head SHA. Approval for another pull request or an earlier head is invalid. If the user declines, leave the pull request open and report it as blocked; if the user does not respond, leave it awaiting approval.

In automatic mode, the initial choice authorizes every eligible merge in the named repository for the current run without further merge prompts. It never bypasses fresh gate verification or the expected-head guard. Pause or stop revokes remaining automatic approval for that run.

Automatic mode also authorizes narrowly scoped, evidence-backed lockfile repair commits for that repository and run. Per-merge mode requires separate repair-push approval tied to the PR, source head SHA, and proposed lockfile-only diff. Repair-push approval and merge approval are independent. A repair candidate qualifies only when terminal logs explicitly identify a stale, frozen, or out-of-sync lockfile, or the repository's documented locked-install/check command reproduces that failure; any unrelated terminal failure keeps the PR Clean failing.

Automatic skill discovery does not itself authorize mutations. If the user asks only for an audit, status, explanation, or recommendation, the skill remains read-only. Repository scope must be unambiguous before the first comment, label change, checkbox update, or merge.

The controller must stop further mutations immediately if the user asks it to pause or stop.

## Goal-Mode Execution

Per-merge mode does not use goal tooling. Do not inspect, create, reuse, update, complete, or block a goal for that run.

Automatic mode requires an active matching goal before repository preflight. After recording automatic mode, inspect goal state. Reuse a goal only when its objective explicitly covers the exact repository's dependency-update queue, or create one without inventing a token budget when none exists. Treat every other active goal as unrelated; never replace or clear it.

If goal capability is unavailable, goal creation fails, or an unrelated unfinished goal prevents startup, tell the user and fall back to per-merge approval before repository preflight. Do not perform automatic merges without the matching goal.

Use an objective equivalent to:

```text
Process the verified Dependabot, Renovate, and Snyk pull-request queue for OWNER/REPOSITORY until every discovered candidate is merged, clean with terminal failing verification, blocked under the defined retry or inactivity limits, or closed or superseded externally. Preserve repository protections, restrict repairs to evidence-backed lockfile-only changes, and validate the final live queue state.
```

In automatic mode, keep the goal active while candidates are Ready, Pending, Need rebase, or have authorized lockfile repair work. Complete it only after the final queue audit proves that every candidate is in a defined terminal state and the final report is ready. Follow the runtime goal tool's own blocked-status rules; an individual blocked pull request does not by itself make the overall goal blocked.

Goal mode adds persistence, not authority. It does not broaden repository scope, grant new credentials, bypass approvals, or relax any merge gate.

References:

- https://learn.chatgpt.com/use-cases/follow-goals
- https://learn.chatgpt.com/docs/long-running-work

## Agent Architecture and Model Routing

One controller owns the queue and is the only agent allowed to mutate GitHub state. It may delegate read-only discovery, pull-request classification, and polling to lightweight workers. Mutating work remains serial because every merge can change the mergeability of all remaining pull requests.

For worker tasks, prefer an available fast model at low reasoning effort. Preferred model families are:

1. Luna low
2. Grok 4.6 low
3. Haiku 4.5 low
4. Flash 3.8 low

The skill must use only models exposed by the active runtime. It must not fail merely because a preferred family is unavailable. The controller remains responsible for validating worker observations against live GitHub state before every mutation; a worker's earlier snapshot is never merge authorization.

## Repository Preflight

Before mutating anything, the controller must establish:

- the exact repository and default branch;
- the open dependency-update pull requests and their verified bot authors;
- the repository's allowed or preferred merge method;
- branch-protection or ruleset requirements, including required approvals and status checks;
- whether current credentials can comment, label, and merge;
- the exact current head SHA of every candidate.

If required-check policy cannot be determined, the controller must not merge. It may continue gathering evidence, but unresolved check requirements are reported as blocked rather than guessed.

## Pull-Request State Classification

Every refresh assigns each candidate to one of these states:

| State | Predicate | Action |
| --- | --- | --- |
| Ready | Non-draft, cleanly mergeable, required approvals satisfied, all required checks successful, and no observed test or verification check failed | Refresh; request exact-head approval in per-merge mode or merge under automatic mode |
| Awaiting approval | Per-merge mode, Ready gates pass, but the user has not approved this pull request at its current head SHA | Ask once; do not merge or infer approval |
| Pending | Checks, mergeability, or bot rebase work is still in progress | Poll until the state changes or the inactivity limit is reached |
| Needs rebase | Conflicted or otherwise stale under the repository's merge policy | Invoke the verified provider adapter, then revisit after the head changes |
| Lockfile repair candidate | Clean, no checks pending, and terminal logs explicitly identify a stale/frozen/out-of-sync lockfile or the documented locked-install/check command reproduces it, with no unrelated terminal failure | Read `references/lockfile-repairs.md` |
| Awaiting repair approval | Per-merge mode and the exact-head lockfile-only repair diff is ready | Ask for separate repair-push approval tied to PR, source SHA, and diff |
| Clean failing | Cleanly mergeable, no checks pending, and a terminal failure is ambiguous, unrelated to the lockfile, or otherwise ineligible for repair | Leave open; refresh again after later merges to ensure it remains clean |
| Blocked | Draft, ambiguous author, missing policy evidence, insufficient permission, unsafe bot rebase, exhausted rebase attempts, or inactivity timeout | Do not merge; report the reason |
| Gone | Closed, superseded, or merged outside the controller | Record the live outcome and remove it from the actionable queue |

Neutral or skipped checks do not count as failures unless repository policy requires success from that check. Cancelled, timed-out, action-required, startup-failure, and stale conclusions count as terminal failures when they represent tests or verifications. A clean failing pull request is intentionally ignored for merging, not hidden from the final report.

Ordinary or ambiguous failures remain Clean failing. A candidate qualifying for repair leaves that state only after its repair is pushed; classify it Pending until fresh checks finish. In automatic mode, keep the matching goal active during repair and fresh checks. In per-merge mode, Awaiting repair approval is a nonterminal handoff.

## Merge Gate

Immediately before every merge, the controller must reread the pull request and checks for its exact current head SHA. It may merge only when all of these are true in that fresh read:

- the pull request is still open and not a draft;
- the verified author and repository scope still match;
- GitHub reports it mergeable under current repository policy;
- required approvals are satisfied;
- every required check for the current head is successful;
- no observed test or verification check for the current head has failed;
- the selected approval mode authorizes the merge: exact-head approval in per-merge mode or run-scoped automatic approval;
- the head SHA is supplied to the merge operation as an expected-head or equivalent concurrency guard.

The skill follows the repository's configured merge policy. It does not invent a global squash, merge-commit, or rebase-merge preference. A stale-head rejection or any changed merge gate requires a complete refresh; it is never bypassed. In per-merge mode, it also invalidates the prior exact-head approval.

After each successful merge, the controller refreshes all remaining candidates before choosing the next mutation.

## Provider Rebase Adapters

The controller uses the provider that opened the pull request. It never substitutes a manual force-push for a supported bot workflow.

### Dependabot

Post the exact pull-request comment:

```text
@dependabot rebase
```

GitHub documents this as the supported Dependabot rebase command. Acknowledgment or a new head SHA proves that Dependabot received or processed the request; the controller still waits for fresh checks before merging.

Reference: https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-pull-request-comment-commands

### Renovate

Prefer the rebase/retry checkbox already present in the pull-request body. If it is unavailable, add the repository's configured `rebaseLabel`; the default label is `rebase`. Determine custom configuration from the repository or the live pull request instead of assuming the default.

Renovate normally removes its rebase label after processing. A changed head SHA or resolved conflict proves progress.

References:

- https://docs.renovatebot.com/updating-rebasing/
- https://docs.renovatebot.com/configuration-options/#rebaselabel

### Snyk

First use any rebase control or command explicitly advertised in the current Snyk-authored pull-request body. If none exists, post a plain-language request that mentions the verified author login, for example:

```text
@snyk-bot please rebase this pull request
```

This fallback is an unverified request, not a documented Snyk command. Only a new head SHA or resolved conflict counts as progress. The controller must not report the mention itself as a successful rebase.

If the request produces no progress after the retry policy, report the pull request as blocked with `no verified Snyk rebase mechanism`.

GitHub's native update-branch feature is not a substitute for this adapter when conflicts already exist because it cannot resolve those conflicts.

Reference: https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/keeping-your-pull-request-in-sync-with-the-base-branch

## Protecting Bot Branches

Before requesting a bot rebase, inspect commits added after the bot's generated update. If non-bot commits could be overwritten or discarded by provider regeneration, do not invoke the adapter. Report the pull request as blocked and identify the human-authored commits that require review.

A clean, passing pull request that contains deliberate human follow-up commits may still be merged when every merge gate passes. The restriction applies to destructive regeneration, not to otherwise eligible merges.

## Loop, Retry, and Completion Rules

The controller processes mutations serially:

1. Record the approval mode; in automatic mode, start or reuse the required goal or fall back to per-merge mode.
2. Refresh the full candidate queue.
3. For one Ready pull request, request exact-head approval in per-merge mode or use the run-scoped approval in automatic mode.
4. Reread every gate immediately before merging and merge only with the fresh expected head SHA.
5. Refresh the entire queue because the base branch changed.
6. Request provider rebases for pull requests that now Need rebase.
7. For an evidence-backed lockfile repair candidate, follow `references/lockfile-repairs.md`; keep repair and merge approvals separate and process its single normal additive push serially.
8. Poll Pending pull requests with lightweight workers when available.
9. Repeat while any pull request can still become Ready or has authorized repair work.

A previously Clean failing pull request remains in refreshes. If a later merge makes it conflicted, it returns to Needs rebase. Once rebased, it is merged if checks pass or returns to Clean failing if they fail.

In per-merge mode, when no Ready, Pending, Needs rebase, or authorized repair work remains but a pull request is Awaiting merge or repair approval, the controller reports each request and hands control back to the user. A repair request includes the PR, source SHA, and proposed lockfile-only diff. This is a nonterminal handoff; resume with a fresh classification before any mutation when the user responds.

For a single unchanged head, make at most three rebase requests that fail to produce progress. Avoid duplicate requests while the provider has acknowledged work or checks are running.

Pending checks, mergeability, and acknowledged bot work have a default 15-minute no-progress limit, measured from the last observable state change. A repository-specific documented timeout may replace this default. When the limit is reached, refresh once and classify the unchanged pull request as Blocked rather than polling forever.

The run completes when every discovered candidate is one of:

- merged;
- currently clean with terminal failing tests or verifications;
- blocked with a concrete reason;
- closed or superseded externally.

## Failure Handling

- After a timeout or ambiguous mutation response, reread live state before retrying.
- If a merge reports that the head changed, refresh metadata and checks; never retry with the old SHA.
- If a bot acknowledges a rebase but the head has not changed, continue polling without spending another retry.
- If permissions disappear mid-run, stop mutations and report all remaining candidates.
- If the bot author cannot be verified, do not comment, label, or merge.
- Do not repair application code, force checks to pass, dismiss reviews, or change repository protection settings. Lockfile regeneration is limited to the evidence-backed, lockfile-only workflow in `references/lockfile-repairs.md`.

## Final Report Contract

The final report must include:

- merged pull requests with provider, final head SHA, merge result, and passing-check evidence;
- lockfile repair commits with commit SHA and evidence, plus fresh-check outcome;
- clean failing pull requests left open, with the failing checks;
- blocked pull requests with exact reasons and the last observed state;
- externally closed or superseded candidates;
- confirmation that the final open-candidate search was reread after the last mutation.

Awaiting approval produces an interim handoff, not the final report. The final report must distinguish a fully cleared queue from a completed run that intentionally left clean failing or blocked pull requests open.

## Approved Lockfile-Repair Extension (2026-09-23)

The repair reference is loaded only for a candidate with explicit stale-lockfile evidence or a reproduced failure from the repository's documented locked-install/check command, and only when no unrelated terminal failure exists. Regenerate at the PR's exact head in an isolated checkout using configured tooling. Accept only expected lockfiles, validate integrity and the relevant failure where supported, and allow one attempt per dependency-manifest state. Branches must accept one normal additive commit; immediately reread and match the remote source SHA before pushing, with normal fast-forward as the final race guard. A changed head discards cached repair work for live reclassification. Ambiguous evidence, broad churn, validation failure, unwritable branches, or a repeated lockfile failure block repair.

This extension does not amend, rewrite, manually rebase, impersonate the bot, or force-push. The resulting non-bot repair commit is covered by the existing provider-rebase rule: do not request provider regeneration when it could overwrite that commit. The pre-edit RED evidence found 3/5 samples refused all repair due to ambiguous authorization and 2/5 inferred repair approval from automatic merge approval; attempted repairs omitted at least one separate approval, head-race check, stale-work discard, or one-attempt bound. Those five samples are the failing baseline and are not rerun.

## Skill Packaging

Create a concise `merging-dependency-updates/SKILL.md` and matching `agents/openai.yaml`. Keep the state machine and safety invariants in the entrypoint because they govern every run. Use a focused provider reference only if the validated skill would otherwise exceed a practical size; do not add scripts unless repeated deterministic parsing is shown to improve reliability.

The skill description should trigger on requests to process, rebase, merge, clear, or babysit Dependabot, Renovate, or Snyk dependency-update pull-request queues. It should not trigger for ordinary dependency upgrades implemented by a human or for read-only PR reviews unless the user asks to process the queue.

## Validation Strategy

Follow the writing-skills RED-GREEN-REFACTOR workflow.

Baseline scenarios must demonstrate failures without the skill, including:

1. Two passing pull requests where merging the first makes the second stale.
2. A clean pull request with a failed test that must remain open.
3. A conflicted Dependabot pull request alongside a passing Renovate pull request.
4. A Snyk pull request with no documented rebase command.
5. A bot branch containing human-authored commits that regeneration could overwrite.
6. A stale-head race between the final check read and merge attempt.
7. Pending checks that never finish.
8. Approval mode unspecified, per-merge mode selected, automatic mode selected with goal availability, and automatic mode blocked by missing or occupied goal tooling.

After authoring, rerun the same scenarios with the skill and verify that the agent:

- keeps mutations serial;
- uses provider-specific rebase behavior;
- refreshes exact-head evidence before merging;
- asks for a mode before preflight when unspecified;
- uses no goal tooling and requests exact-head confirmation in per-merge mode;
- uses goal mode and no further merge prompts in automatic mode;
- falls back to per-merge mode when automatic-mode goal startup cannot succeed;
- leaves clean failing pull requests open;
- fails closed on ambiguous or unsafe states;
- stops at the defined retry and inactivity limits;
- starts or reuses an appropriate goal only in automatic mode without replacing unrelated work or expanding repository scope;
- completes an automatic-mode goal only after the final live queue audit;
- produces the required final report.

Finally, run the skill validator and inspect the generated UI metadata for consistency with the entrypoint.
