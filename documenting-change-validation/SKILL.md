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

### Manual QA for Engineering

### Manual QA for Product Owners

## Visual Proof
```

Match headings case-insensitively. Preserve all other body content, including content added by other skills. Add missing sections without restructuring the body or creating duplicates.

For every web or native UI change, `## Visual Proof` is required. It must contain the captured artifacts or an explicit reason they are unavailable.

## Automated

Inspect changed files, repository instructions, project-native commands, CI, and documented workflows. Use only supported commands; never invent one or claim it ran when it did not.

Under `### Automated`:

- List exact commands and what each verifies.
- Report results from commands actually run.
- State when no reliable project-native automated path exists.

Prefer focused checks for the changed behavior, followed by broader checks when relevant.

## Manual QA for Engineering

Under `### Manual QA for Engineering`, state prerequisites, setup, fixtures, permissions, or environment assumptions; give numbered, reproducible steps with expected results; and explain when engineering manual testing genuinely does not apply.

Keep guidance specific to the diff. Inspect further or ask the user instead of fabricating coverage.

## Manual QA for Product Owners

Under `### Manual QA for Product Owners`, give numbered, user-facing acceptance steps with expected outcomes. Exclude engineering-only setup details; link back to the engineering section when product-owner verification depends on a prepared environment.

Keep acceptance guidance specific to the changed user experience. Explain when product-owner QA genuinely does not apply.

## Commit Hygiene

When the PR history contains extraneous, fixup, or fragmented commits, review the changes and group them into a small, reviewable set of logical commits.

- Keep independently understandable changes separate when that makes review clearer.
- Combine incremental or mechanical commits that do not represent a meaningful review boundary.
- Do not squash everything into one commit merely for fewer commits, and do not combine changes that should remain distinct.
- Before rewriting history, confirm the base branch and intended PR diff; afterward, verify the rewritten history preserves that diff.

## Visual Proof

Keep `## Visual Proof` separate from `## How to Test`. Attach or link only screenshots actually captured during the current work.

Use the appropriate Manual QA section for steps and expected results; use `## Visual Proof` for each artifact:

```markdown
- <surface> — <viewport/device> — <theme> — <mode>: <link or attachment>
```

- To capture proof, use the repository's supported local runner and load, generate, or recreate the minimum safe fixtures/state needed to reproduce the changed surface—even after implementation scaffolding was removed. Clean up temporary local setup when appropriate.
- For web UI changes, capture mobile, tablet, and desktop. Use documented breakpoints or label representative CSS-pixel widths.
- When dark mode is supported, capture the same views in dark mode.
- For print changes, emulate print media and capture near-A4 or Letter at the intended orientation.
- For native changes with local development, run the app and capture the affected surface; repeat in dark mode when supported.
- If visual proof does not apply or an applicable capture cannot be produced, say so explicitly and give the reason.

## GitHub PR visual proof

For GitHub PRs with screenshots, follow [GitHub PR visual proof](references/github-pr-visual-proof.md); otherwise use the generic format.

## Updates

On update, replace stale `Automated`, both Manual QA sections, and `Visual Proof` content; preserve other content and requirements.

Do not assume a platform or CLI. The active platform skill handles request operations; this skill supplies testing content or a ready-to-paste section.
