# Evidence-Backed Lockfile Repairs

Read this reference only for a Clean failing candidate whose terminal failure qualifies for lockfile repair.

## Qualify the Failure

A repair candidate requires one of these observable signals, and every terminal failure on the PR must be explained by the same stale-lockfile condition:

- Terminal-check logs explicitly identify a stale, frozen, or out-of-sync lockfile.
- The repository's documented locked-install or check command reproduces the lockfile failure.

Inspect all terminal failures first. If any terminal failure is not explained by the same stale-lockfile condition, keep the PR Clean failing. Ordinary or ambiguous failures also stay Clean failing; do not infer a lockfile cause from a dependency PR or a vague install error.

## Prepare and Validate

Define the dependency-manifest state from every tracked manifest, workspace descriptor, and package-manager configuration or input that the configured manager uses to generate the expected lockfile or lockfiles. Record how this input set was determined, plus each path and its exact blob ID (or content hash) at the repair source head. This ordered path-and-ID/hash tuple identifies the state. A changed tuple is a new state and must freshly qualify under the evidence criteria; a source-head change outside these inputs does not create another allowance.

On startup or resume, and at every candidate refresh, inspect commits after the provider-generated update for a non-bot repair commit that changes only expected lockfiles. Compare its parent commit's recorded or reconstructable dependency-input tuple with the current tuple; a match means the attempt is already consumed. If prior repair-attempt state or the parent tuple cannot be determined safely, classify Blocked rather than risk a second repair.

An attempt is consumed only when the repair commit is verified on the remote PR branch. Preparation discarded before push because the PR head or approved diff changed does not consume an attempt; reread live state, then rebuild and revalidate from the exact current head. Regeneration or validation failure immediately makes the candidate Blocked and is not retryable. If a pushed repair commit is verified but the stale-lockfile failure remains for the same dependency-manifest state, classify Blocked; allow at most one verified repair commit per recorded state.

Run the repair in an isolated checkout at the PR's exact current head SHA. Follow the repository's configured package manager, pinned tooling guidance, and documented commands. Determine the expected lockfile path or paths from that configuration and regenerate only those files. After all generation and validation commands finish, reread the local source HEAD and the complete worktree delta in both automatic and per-merge modes. Require the source HEAD to remain the recorded SHA and the exact delta to contain only the expected lockfile changes before creating or pushing the commit. Any unexpected lockfile, manifest, source, configuration, unrelated tracked or untracked file change, or broad/ambiguous dependency churn makes the candidate Blocked. Any validation-command mutation outside the expected lockfiles is Blocked.

Before push, a repository-supported package-manager integrity command or non-mutating locked/frozen install/check command must succeed. If no such command can be identified or run, classify Blocked; never publish an unvalidated repair. Rerun the previously failing command where supported.

## Authorize and Push

Prepare exactly one normal additive repair commit on top of the verified source head. The proposed diff must contain only expected lockfile changes. Assess writability from current permissions, branch rules, and branch ownership without an exploratory push. Each freshly prepared repair gets at most one actual normal push attempt, only after the immediate remote-head reread below; its fast-forward behavior is the final race guard. Existing provider-rebase protection still applies: provider regeneration that could overwrite non-bot commits is unsafe; an additive repair commit does not rewrite those commits.

- In automatic mode, the run-scoped choice authorizes this narrowly scoped repair commit for the named repository. The matching goal stays active while repair work remains.
- In per-merge mode, stop at **Awaiting repair approval** until the user separately approves pushing the exact proposed lockfile-only diff for the named PR at its exact source head SHA. This approval does not authorize merge; if the PR identity, source SHA, or proposed diff changes, approval is invalid and requires a freshly prepared repair and new approval.

Immediately before pushing, reread the remote PR head and require it to equal the repair's source SHA; in per-merge mode, also confirm the approved diff is unchanged. If the head or diff differs before the push, or a normal push is rejected for a fast-forward/head race, reread live state, discard the prepared repair, and reclassify. Discard means do not push or reuse that prepared checkout, commit, or diff. Since no repair commit was verified remotely, the discarded preparation does not consume the one-verified-repair-per-input-state allowance. If the candidate still qualifies, prepare again from live state and obtain fresh per-merge approval; that preparation gets its own single push attempt. Do not retry cached work. For an ambiguous push response, use an authoritative read of the remote PR branch ref and its commit ancestry after the push transaction; a cached UI or check snapshot is insufficient. Use exactly these outcomes: (a) the exact prepared repair commit is verified on the remote PR branch—consume the allowance and classify Pending; (b) the authoritative read definitively shows the commit absent—spend and discard this preparation without consuming the allowance, then reread and requalify before freshly preparing, with new per-merge approval before another push (including when the ref advanced elsewhere); (c) API, cache, transport, or other uncertainty prevents determining presence or absence—classify Blocked and do not retry. If permissions or branch protection reject a normal push, or writability remains ambiguous, leave the PR Blocked with the evidence. Never amend, impersonate the bot, manually rebase, rewrite, or force-push its branch.

## Resume Queue Processing

After a verified push, classify the PR as Pending and wait for checks on the new head. Merge only through the existing mode-specific approval and exact-head gates after fresh checks pass. If the stale-lockfile failure persists after the one repair for the same dependency-manifest state, classify Blocked. Report the repair commit SHA and the lockfile evidence in the final live audit.
