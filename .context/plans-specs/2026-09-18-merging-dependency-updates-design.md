# Merging Dependency Updates Skill Design

## Purpose

Create a reusable `merging-dependency-updates` skill that processes a GitHub repository's open dependency-update pull requests from Dependabot, Renovate, and Snyk. Once a user explicitly authorizes a repository run, the skill autonomously discovers, classifies, rebases, and polls eligible pull requests, requests user confirmation before each exact-head merge, and repeats until no actionable pull request remains.

The skill leaves clean pull requests with terminal failing checks open. It does not repair their code, override their checks, or treat them as blockers to completing the queue run.

## Supported Scope

The first version supports GitHub pull requests authored by verified Dependabot, Renovate, or Snyk bot identities. It does not process GitLab merge requests, dependency issues without pull requests, or human-authored dependency pull requests.

Candidate detection must use the authenticated pull-request author identity plus dependency-update evidence from the pull request. Labels, title prefixes, or branch names alone are insufficient because repositories can apply or imitate them.

Draft pull requests are not merge candidates. They remain open and appear in the final blocked report.

## Authorization Boundary

An explicit user request to process dependency updates in a named repository authorizes discovery, provider rebase requests, and polling for that repository. Before every merge, the skill must ask the user to approve that specific pull request at its freshly verified head SHA.

Queue-level authorization, approval for another pull request, or approval for an earlier head SHA is not merge approval. If the head or any merge gate changes after approval, refresh the pull request and request new approval only after it is Ready again. If the user declines, leave the pull request open and report it as blocked. If the user does not respond, leave it awaiting approval.

Automatic skill discovery does not itself authorize mutations. If the user asks only for an audit, status, explanation, or recommendation, the skill remains read-only. Repository scope must be unambiguous before the first comment, label change, checkbox update, or merge.

The controller must stop further mutations immediately if the user asks it to pause or stop.

## Goal-Mode Execution

This queue is a long-running task with a verification loop and a measurable stopping condition. At the start of an authorized run, the skill should use the runtime's goal mode, goal skill, or goal tool when one is available.

Before creating a goal, inspect current goal state when the runtime supports that operation:

- If no unfinished goal exists, create one for the named repository without inventing a token budget.
- If the current goal already covers the same repository queue, continue under it rather than creating a duplicate.
- If an unrelated unfinished goal prevents creation, do not replace or clear it. Continue the queue in the current session and report that goal mode could not be started.
- If goal capability is unavailable or disabled, continue normally; absence of goal mode is not a queue blocker.

Use an objective equivalent to:

```text
Process the verified Dependabot, Renovate, and Snyk pull-request queue for OWNER/REPOSITORY until every discovered candidate is merged, clean with terminal failing verification, blocked under the defined retry or inactivity limits, or closed or superseded externally. Preserve repository protections, do not repair failing pull requests, and validate the final live queue state.
```

Keep the goal active while candidates are Ready, Awaiting approval, Pending, or Need rebase. Complete it only after the final queue audit proves that every candidate is in a defined terminal state and the final report is ready. Follow the runtime goal tool's own blocked-status rules; an individual blocked pull request does not by itself make the overall goal blocked.

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
| Ready | Non-draft, cleanly mergeable, required approvals satisfied, all required checks successful, and no observed test or verification check failed | Refresh immediately, then request user approval for the exact head |
| Awaiting approval | Ready gates pass, but the user has not approved this pull request at its current head SHA | Ask once; do not merge or infer approval |
| Pending | Checks, mergeability, or bot rebase work is still in progress | Poll until the state changes or the inactivity limit is reached |
| Needs rebase | Conflicted or otherwise stale under the repository's merge policy | Invoke the verified provider adapter, then revisit after the head changes |
| Clean failing | Cleanly mergeable, no checks pending, and at least one test or verification check has a terminal failure | Leave open; refresh again after later merges to ensure it remains clean |
| Blocked | Draft, ambiguous author, missing policy evidence, insufficient permission, unsafe bot rebase, exhausted rebase attempts, or inactivity timeout | Do not merge; report the reason |
| Gone | Closed, superseded, or merged outside the controller | Record the live outcome and remove it from the actionable queue |

Neutral or skipped checks do not count as failures unless repository policy requires success from that check. Cancelled, timed-out, action-required, startup-failure, and stale conclusions count as terminal failures when they represent tests or verifications. A clean failing pull request is intentionally ignored for merging, not hidden from the final report.

## Merge Gate

Immediately before every merge, the controller must reread the pull request and checks for its exact current head SHA. It may merge only when all of these are true in that fresh read:

- the pull request is still open and not a draft;
- the verified author and repository scope still match;
- GitHub reports it mergeable under current repository policy;
- required approvals are satisfied;
- every required check for the current head is successful;
- no observed test or verification check for the current head has failed;
- the user explicitly approved merging this pull request at this exact head SHA;
- the head SHA is supplied to the merge operation as an expected-head or equivalent concurrency guard.

The skill follows the repository's configured merge policy. It does not invent a global squash, merge-commit, or rebase-merge preference. A stale-head rejection or any changed merge gate invalidates the user's approval and requires a complete refresh; it is never bypassed.

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

1. Refresh the full candidate queue.
2. Ask the user to approve one Ready pull request at its fresh head SHA, reread every gate after approval, then merge only if the approved head and every gate remain unchanged.
3. Refresh the entire queue because the base branch changed.
4. Request provider rebases for pull requests that now Need rebase.
5. Poll Pending pull requests with lightweight workers when available.
6. Repeat while any pull request can still become Ready.

A previously Clean failing pull request remains in refreshes. If a later merge makes it conflicted, it returns to Needs rebase. Once rebased, it is merged if checks pass or returns to Clean failing if they fail.

When no Ready, Pending, or Needs rebase work remains but at least one pull request is Awaiting approval, the controller reports each exact-head approval request and hands control back to the user. This is a nonterminal handoff: leave the goal active, do not mark the run complete or blocked, and resume with a fresh classification when the user responds.

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
- Do not repair code, regenerate lockfiles, force checks to pass, dismiss reviews, or change repository protection settings. Those are separate tasks requiring separate authorization.

## Final Report Contract

The final report must include:

- merged pull requests with provider, final head SHA, merge result, and passing-check evidence;
- clean failing pull requests left open, with the failing checks;
- blocked pull requests with exact reasons and the last observed state;
- externally closed or superseded candidates;
- confirmation that the final open-candidate search was reread after the last mutation.

Awaiting approval produces an interim handoff, not the final report. The final report must distinguish a fully cleared queue from a completed run that intentionally left clean failing or blocked pull requests open.

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
8. Goal mode available with no active goal, unavailable, and occupied by an unrelated unfinished goal.

After authoring, rerun the same scenarios with the skill and verify that the agent:

- keeps mutations serial;
- uses provider-specific rebase behavior;
- refreshes exact-head evidence before merging;
- asks for user confirmation before each merge, tied to the freshly verified pull request head SHA;
- leaves clean failing pull requests open;
- fails closed on ambiguous or unsafe states;
- stops at the defined retry and inactivity limits;
- starts or reuses an appropriate goal when possible without replacing unrelated work or expanding permissions;
- completes the goal only after the final live queue audit;
- produces the required final report.

Finally, run the skill validator and inspect the generated UI metadata for consistency with the entrypoint.
