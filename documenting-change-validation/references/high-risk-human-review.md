# High-Risk Human Review

Use this assessment for every pull/merge request body created or updated with this skill. Its purpose is to surface one-way-door architectural and operational risks for focused human review, not to claim that unflagged code is safe.

## Inspect the diff

1. Resolve the PR base and head SHA. Inspect the complete merge-base-to-head diff, including renamed files and deleted lines; do not assess only the latest commit or staged changes.
2. Evaluate changed paths and changed code in all six categories below. Use repository context, types, schemas, routes, serializers, tests, and call sites to determine the change's meaning. Filename matching alone is insufficient.
3. Create a separate row for each distinct risk location. Consolidate adjacent lines only when they represent one risk and share one review decision.
4. Link the narrowest changed line range that supports the finding. Do not link an entire file when specific lines are available.
5. State what changed and the concrete question a human must verify. Do not merely restate the category or severity.

## Mandatory trigger categories

| Category | Flag when the diff changes | Severity |
| :--- | :--- | :--- |
| **Schema & Persistence** | Database migrations; ORM entities or models; table definitions; or local device storage schemas such as CoreData, Room, or SQLite. | 🚨 Critical |
| **API Contracts & External Interfaces** | JSON request or response structures, fields, endpoints, or HTTP status codes. Treat renames, removals, and incompatible behavior as breaking; treat backward-compatible additions as additive. | 🚨 Critical when breaking or removing; ⚠️ Warning when additive |
| **Security & Authorization** | Authentication middleware; RBAC, permission, or authorization checks; session handling; token verification; or logging that can expose PII. | 🚨 Critical |
| **Dependencies & Frameworks** | Additions, removals, or version changes in dependency manifests, including `package.json`, `Cargo.toml`, `go.mod`, `build.gradle`, `Gemfile`, and `pyproject.toml`. | ℹ️ Notice |
| **Query Safety & Data Access (ORM / SQL)** | Any query-safety trigger below. | Use the trigger-specific severity below |
| **External Services & Operational Resilience** | Outbound calls to third-party APIs, webhooks, remote data services, payment providers, messaging systems, or other network dependencies, including changes to how those calls are bounded and handled. | Use the trigger-specific severity below |

Dependency lockfiles can support a manifest finding, but do not create a separate row for generated lockfile churn unless it introduces a distinct risk.

Use **API Contracts & External Interfaces** for the contract this code publishes. Use **External Services & Operational Resilience** for how this code consumes or depends on a remote system; apply both when both risks are present.

### Query-safety triggers

- **N+1 Risk — 🚨 Critical:** ORM queries or database calls execute inside `for`, `while`, `map`, `forEach`, or equivalent iteration. Include indirect calls when the changed loop invokes a helper that queries the database.
- **Unbounded Fetching — ⚠️ Warning:** A query returns a collection without `LIMIT`, `take`, an explicit bound, or pagination parameters. Do not flag a proven single-record lookup as an unbounded list fetch.
- **Index Bypasses — ⚠️ Warning:** A `WHERE` predicate applies a transformation or function to a column, such as `LOWER(email)`, in a way that can bypass an ordinary index.
- **Raw Execution — 🚨 Critical:** Raw SQL uses string concatenation, interpolation, or otherwise unparameterized input. Parameterized raw SQL is not automatically critical under this trigger, though it can still be flagged under another applicable category.

### External-service triggers

- **External calls inside loops or unbounded concurrent fan-out — 🚨 Critical:** Added or changed code issues remote requests from iteration or launches requests without a concurrency bound. Flag the call site even when a helper hides the network operation.
- **Retrying non-idempotent writes or retries without bounds — 🚨 Critical:** A retry can duplicate a remote side effect, or attempts have no explicit maximum. Check for idempotency keys or an equivalent deduplication contract when the provider supports them.
- **Requests without explicit timeouts — ⚠️ Warning:** A remote call can wait on library or operating-system defaults instead of a deliberate connect and/or response deadline.
- **Missing status/error handling — ⚠️ Warning:** The changed path does not deliberately handle non-success responses, transport failures, or malformed remote data relevant to the operation.
- **New synchronous dependency in a request-critical path — ⚠️ Warning:** User-facing latency or availability now directly depends on a remote service completing successfully.
- **Missing rate-limit/backoff handling — ⚠️ Warning:** The integration can receive throttling responses but has no deliberate bounded response strategy, such as honoring provider retry guidance, queueing, or failing safely.
- **New external vendor/service dependency — ℹ️ Notice:** The diff introduces a new remote system whose operational ownership, credentials, quotas, cost, data handling, or availability expectations need confirmation.

Do not require retries, circuit breakers, fallbacks, or asynchronous execution for every external call. Flag the concrete failure mode visible in the diff and ask the reviewer to verify the appropriate resilience strategy for that operation. Security issues such as credentials or PII exposure remain 🚨 Critical under **Security & Authorization** as well.

## Permalinks

For GitHub, use immutable blob links to the exact modified lines:

```text
https://github.com/{owner}/{repo}/blob/{sha}/{filepath}#L{start_line}-L{end_line}
```

Use the head SHA for added or modified lines. For a risk represented only by deleted lines, use the base SHA and the deleted line range so the permalink resolves to the reviewed code. URL-encode the path when necessary. A single-line link can use `#L{line}`. If immutable repository metadata is unavailable, use the standard relative form `./{filepath}#L{start_line}-L{end_line}` and do not invent owner, repository, or SHA values.

The link label should identify the location, for example ``[`src/users/query.ts#L42-L47`](https://github.com/acme/app/blob/abc123/src/users/query.ts#L42-L47)``.

## Review context

Explain why the item needs human sign-off, using the actual diff and expected operating conditions. Good context names the failure mode or decision to verify, for example:

- `Query called inside loop—verify N+1 impact at production scale.`
- `Column modified on primary user table—verify a non-blocking migration and rollback plan.`
- `Response field removed—verify all external consumers have migrated before deployment.`
- `Authorization branch changed—verify denied roles cannot reach the operation.`
- `External request executed inside loop—verify bounded concurrency, rate-limit impact, and projected cost.`
- `Non-idempotent write is retried—verify duplicate side effects cannot occur.`
- `No explicit request timeout—verify remote latency cannot exhaust application workers.`
- `Synchronous vendor call added to request path—verify acceptable latency and failure behavior.`

Do not infer that a migration is blocking, an API change is breaking, or a log contains PII without evidence. When the diff creates a credible risk but its operational impact is uncertain, state the uncertainty as the human verification question.
