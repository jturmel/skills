---
name: documenting-change-validation
description: Use when creating or updating a pull request, merge request, or equivalent change request and its body needs accurate automated, manual, and visual validation guidance.
---

# Documenting Change Validation

Maintain accurate testing and visual-evidence content in a pull/merge request body. Work alongside the repository's Git or hosting-platform skills; do not replace their request-creation, update, or other body-content workflows.

## Required sections

Ensure the body contains these sections and headings:

```markdown
## How to Test

### Automated

### Manual QA

## Visual Proof
```

Match headings case-insensitively. Preserve all other body content, including content added by other skills. Add either missing section without restructuring the body or creating duplicates.

For every web or native UI change, `## Visual Proof` is required. It must contain the captured artifacts or an explicit reason they are unavailable.

## Automated

Inspect changed files, repository instructions, project-native commands, CI, and documented workflows. Use only supported commands; never invent one or claim it ran when it did not.

Under `### Automated`:

- List exact commands and what each verifies.
- Report results from commands actually run.
- State when no reliable project-native automated path exists.

Prefer focused checks for the changed behavior, followed by broader checks when relevant.

## Manual QA

Under `### Manual QA`, state prerequisites, setup, fixtures, permissions, or environment assumptions; give numbered, reproducible steps with expected results; and explain when manual testing genuinely does not apply.

Keep guidance specific to the diff. Inspect further or ask the user instead of fabricating coverage.

## Visual Proof

Keep `## Visual Proof` separate from `## How to Test`. Attach or link only screenshots actually captured during the current work.

Use `### Manual QA` for reproducible steps and expected results; use `## Visual Proof` for screenshot entries. Add each captured artifact under `## Visual Proof` in this form:

```markdown
- <surface> — <viewport/device> — <theme> — <mode>: <link or attachment>
```

- For web UI changes, capture mobile, tablet, and desktop widths. Use documented breakpoints when available; otherwise choose representative widths and label their CSS-pixel values.
- When dark mode is supported, capture the same views in dark mode.
- For print CSS or output, emulate print media and capture a near-A4 or Letter result at the intended orientation.
- For native mobile changes with local development, run the app and capture the affected screen or flow; repeat in dark mode when supported.
- If visual proof does not apply or an applicable capture cannot be produced, say so explicitly and give the reason.

## GitHub PR visual proof

For GitHub PRs with screenshots, follow [GitHub PR visual proof](references/github-pr-visual-proof.md); otherwise use the generic format.

## Updates

When updating a request, replace stale `Automated`, `Manual QA`, and `Visual Proof` content while preserving everything else. Keep requirements from other skills.

Do not assume GitHub, GitLab, Bitbucket, or any specific CLI. The active platform skill performs the request operation; this skill supplies the testing content or a ready-to-paste section when direct body editing is unavailable.
