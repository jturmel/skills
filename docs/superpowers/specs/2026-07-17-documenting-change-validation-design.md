# `documenting-change-validation` Design

## Goal

Create a standalone, platform-neutral Codex skill that ensures every newly created or updated pull/merge request includes accurate testing guidance, while remaining additive to Git, GitHub, GitLab, Bitbucket, and other repository skills.

## Scope

The skill applies only when a pull/merge request is being created or updated. It owns the request body's `How to Test` section and these required subsections:

```markdown
## How to Test

### Automated

### Manual QA
```

Other skills remain responsible for platform operations and any additional request-body content.

## Workflow

1. Inspect the changed files and relevant repository instructions.
2. Identify project-native test and verification commands from established configuration, scripts, task definitions, documentation, and the work already performed.
3. Inspect the existing request body when updating a request.
4. Preserve all content outside `How to Test`.
5. Add the section if it is absent; otherwise update its `Automated` and `Manual QA` subsections without creating duplicates.
6. Keep testing guidance specific to the changed behavior.

## Output contract

### Automated

Include the exact supported commands and a short explanation of what each verifies. Include actual results when commands were run. If the repository has no usable automated verification path, say so explicitly instead of inventing a command.

### Manual QA

Include prerequisites, numbered steps, and expected outcomes for browser, application, operational, or other relevant manual checks. If manual testing genuinely cannot apply, state why explicitly.

If the diff or repository context is too ambiguous to support accurate instructions, inspect further or ask the user rather than fabricating coverage.

## Compatibility

The skill must not depend on a specific Git hosting platform or CLI. The active Git/GitHub/GitLab/etc. skill performs the request operation; this skill supplies or maintains the testing content.

## Validation scenarios

- A feature change with existing automated commands and browser-visible behavior.
- A backend or infrastructure change with automated checks but no meaningful manual UI flow.
- A repository without configured automated verification.
- An update to a request body containing unrelated sections and stale testing guidance.

Success means the final body contains one coherent `How to Test` section with both required subsections, accurate repository-native guidance, and all unrelated body content preserved.
