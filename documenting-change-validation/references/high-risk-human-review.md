# High-Risk Human Review

Use this assessment for every pull/merge request body created or updated with this skill. Its purpose is to surface one-way-door architectural and operational risks for focused human review, not to claim that unflagged code is safe.

## Inspect the diff

1. Resolve the PR base and head SHA. Inspect the complete merge-base-to-head diff, including renamed files and deleted lines; do not assess only the latest commit or staged changes.
2. Evaluate changed paths and changed code in all five categories below. Use repository context, types, schemas, routes, serializers, tests, and call sites to determine the change's meaning. Filename matching alone is insufficient.
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

Dependency lockfiles can support a manifest finding, but do not create a separate row for generated lockfile churn unless it introduces a distinct risk.

### Query-safety triggers

- **N+1 Risk — 🚨 Critical:** ORM queries or database calls execute inside `for`, `while`, `map`, `forEach`, or equivalent iteration. Include indirect calls when the changed loop invokes a helper that queries the database.
- **Unbounded Fetching — ⚠️ Warning:** A query returns a collection without `LIMIT`, `take`, an explicit bound, or pagination parameters. Do not flag a proven single-record lookup as an unbounded list fetch.
- **Index Bypasses — ⚠️ Warning:** A `WHERE` predicate applies a transformation or function to a column, such as `LOWER(email)`, in a way that can bypass an ordinary index.
- **Raw Execution — 🚨 Critical:** Raw SQL uses string concatenation, interpolation, or otherwise unparameterized input. Parameterized raw SQL is not automatically critical under this trigger, though it can still be flagged under another applicable category.

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

Do not infer that a migration is blocking, an API change is breaking, or a log contains PII without evidence. When the diff creates a credible risk but its operational impact is uncertain, state the uncertainty as the human verification question.
