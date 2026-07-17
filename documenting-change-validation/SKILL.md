---
name: documenting-change-validation
description: Use when creating or updating a pull request, merge request, or equivalent change request and its body needs accurate automated and manual testing guidance.
---

# Documenting Change Validation

Maintain the testing content in a pull/merge request body. Work alongside the repository's Git or hosting-platform skills; do not replace their request-creation, update, or other body-content workflows.

## Required section

Ensure the body contains one section with these headings:

```markdown
## How to Test

### Automated

### Manual QA
```

Match existing headings case-insensitively when updating a body. Preserve every section and detail outside `How to Test`, including content added by other skills. If the section is absent, add it without restructuring the rest of the body. Do not create duplicate testing sections.

## Automated

Inspect the changed files, repository instructions, task definitions, package scripts, CI configuration, and documented workflows. Use only test or verification commands the project already supports; never invent commands or claim a command was run when it was not.

Under `### Automated`:

- List exact commands and briefly state what each verifies.
- Report the result of commands actually run during the current work.
- Say explicitly that automated verification is not configured when no reliable project-native path exists.

Prefer focused checks for the changed behavior, followed by broader checks when relevant.

## Manual QA

Describe how a reviewer can exercise the changed behavior in the browser, application, command line, deployment environment, or other relevant interface.

Under `### Manual QA`:

- State prerequisites, setup, fixtures, permissions, or environment assumptions.
- Give numbered, reproducible steps.
- State the expected result for each meaningful check.
- Explain why manual testing is not applicable when the change genuinely cannot be exercised manually.

Keep the instructions specific to the diff. If the diff or repository context is too ambiguous to support accurate guidance, inspect further or ask the user instead of fabricating coverage.

## Updates

When updating an existing request, replace stale `Automated` and `Manual QA` content while preserving the rest of `How to Test` and the rest of the body. Keep additional testing requirements from other skills in place.

Do not assume GitHub, GitLab, Bitbucket, or any specific CLI. The active platform skill performs the request operation; this skill supplies the testing content or a ready-to-paste section when direct body editing is unavailable.
