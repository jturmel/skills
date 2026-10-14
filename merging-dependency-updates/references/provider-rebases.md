# Provider Rebase Adapters

Read this reference only when at least one verified candidate is Needs rebase.

## Before Every Rebase Request

Inspect commits added after the bot-generated update. If regeneration could overwrite a non-bot commit, do not invoke the bot; classify the PR Blocked and report the commits. A lockfile-repair commit is a non-bot commit for this check. A clean passing PR containing deliberate human follow-up commits may still merge through the normal gate.

Record provider, head SHA, conflict/staleness evidence, prior request count for that unchanged head, and the proof that would establish progress.

## Dependabot

Post exactly:

```text
@dependabot rebase
```

Treat a provider acknowledgment as in-progress. A new head SHA or resolved conflict proves progress. Wait for fresh checks before classification.

Source: https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-pull-request-comment-commands

## Renovate

Prefer the PR body's rebase/retry checkbox. Otherwise add the repository's configured `rebaseLabel`; use `rebase` only when configuration does not override the default. Renovate normally removes the label after processing.

A new head SHA or resolved conflict proves progress. Do not edit Renovate commits or manually rebase its branch.

Sources:

- https://docs.renovatebot.com/updating-rebasing/
- https://docs.renovatebot.com/configuration-options/#rebaselabel

## Snyk

Use a rebase control explicitly advertised in the current Snyk-authored PR body. If none exists, post one plain-language request mentioning the verified author, for example:

```text
@snyk-bot please rebase this pull request
```

This mention is unverified, not a documented command. Only a new head SHA or resolved conflict counts as progress. After three requests without progress for the unchanged head, classify it Blocked with `no verified Snyk rebase mechanism`.

GitHub's native update-branch feature cannot resolve existing conflicts and is not a substitute.

Source: https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/keeping-your-pull-request-in-sync-with-the-base-branch

## Retry Accounting

Count another attempt only when making a new request against the same unchanged head. Do not spend an attempt while the provider has acknowledged work or checks are running. After an ambiguous request response, reread comments, labels, head SHA, and mergeability before retrying.
