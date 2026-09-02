---
name: documenting-change-validation
description: Use when creating or updating a pull request, merge request, or equivalent change request and its body needs accurate automated, manual, and visual validation guidance.
---

# Documenting Change Validation

Maintain accurate testing and visual-evidence content in a pull/merge request body. Work alongside the repository's Git or hosting-platform skills; do not replace their request-creation, update, or other body-content workflows.

When creating or updating a pull/merge request, include its confirmed direct URL in the user-facing response. If the platform does not return or verify a URL, say that it is unavailable; never guess one.

## Required sections

Ensure the body contains these sections and headings:

````markdown
## ⚠️ High-Risk Items for Human Review

## How to Test

<details>
<summary><h3>Automated Testing Instructions</h3></summary>

[Exact commands, results, and what each command verifies]

</details>

<details>
<summary><h3>Engineering QA Instructions</h3></summary>

[Engineering prerequisites and numbered reproducible steps with expected results]

</details>

<details>
<summary><h3>Product Owner QA Instructions</h3></summary>

```text
[Numbered user-facing acceptance steps with expected outcomes]
```

</details>

## Visual Proof
````

Match headings and summary text case-insensitively. Preserve all other body content, including content added by other skills. Add missing sections without restructuring the body or creating duplicates.

Each testing subsection must use `<details>` without the `open` attribute so it is collapsed by default. Put its exact `<h3>` heading inside `<summary>` to preserve the former `###` heading size. Keep a blank line between `</summary>` and the body and before `</details>` so GitHub renders the contents correctly.

For any change that changes what any user can see or do in the system—web, native mobile, desktop, email, PDF, print, or another user-facing surface—`## Visual Proof` is mandatory. It must contain captured artifacts for the affected surface, or a specific evidence-based explanation after a documented capture attempt. Do not treat a UI change as exempt because it is small, difficult to reproduce, lacks ready-made fixtures, or requires local setup.

Before writing or updating the body, inspect the complete PR diff and follow [High-risk human review](references/high-risk-human-review.md). This assessment is mandatory for every change, even when no item is ultimately flagged.

## Automated Testing Instructions

Inspect changed files, repository instructions, project-native commands, CI, and documented workflows. Use only supported commands; never invent one or claim it ran when it did not.

Under `<summary><h3>Automated Testing Instructions</h3></summary>`:

- List exact commands and what each verifies.
- Report results from commands actually run.
- State when no reliable project-native automated path exists.

Prefer focused checks for the changed behavior, followed by broader checks when relevant.

## Engineering QA Instructions

Under `<summary><h3>Engineering QA Instructions</h3></summary>`, state prerequisites, setup, fixtures, permissions, or environment assumptions; give numbered, reproducible steps with expected results; and explain when engineering manual testing genuinely does not apply.

Keep guidance specific to the diff. Inspect further or ask the user instead of fabricating coverage.

## Product Owner QA Instructions

Under `<summary><h3>Product Owner QA Instructions</h3></summary>`, give numbered, user-facing acceptance steps with expected outcomes. Exclude engineering-only setup details; link back to the engineering section when product-owner verification depends on a prepared environment.

Wrap the complete Product Owner body in a fenced `text` code block. This presents the instructions as one copyable field with GitHub's native code-block copy control. Do not put the heading inside the code block. Keep the text understandable when copied out of the PR, and use full URLs rather than Markdown links when a link is required.

Keep acceptance guidance specific to the changed user experience. Explain when product-owner QA genuinely does not apply inside the copyable text block.

## Commit Hygiene

When the PR history contains extraneous, fixup, or fragmented commits, review the changes and group them into a small, reviewable set of logical commits.

- Keep independently understandable changes separate when that makes review clearer.
- Combine incremental or mechanical commits that do not represent a meaningful review boundary.
- Do not squash everything into one commit merely for fewer commits, and do not combine changes that should remain distinct.
- Before rewriting history, confirm the base branch and intended PR diff; afterward, verify the rewritten history preserves that diff.

## Visual Proof

Keep `## Visual Proof` separate from `## How to Test`. Attach or link only screenshots actually captured during the current work.

### Mandatory UI evidence

If the diff changes UI for even one category of user, make a serious attempt to capture visual proof before writing the PR body. Do not take the lazy path of omitting screenshots or immediately claiming that proof is unavailable.

- Use the repository's supported local development runner, browser/device tooling, or other documented preview path.
- Reuse existing fixtures and application state when they cover the changed surface.
- If they do not, create the smallest safe fixtures and state needed to reach the feature, role, data, loading, empty, error, authenticated, or other relevant view. Restore temporary scaffolding when practical, even if the implementation no longer needs it.
- Capture the affected UI after reaching the real route or runtime path. A screenshot of an unrelated page, placeholder, failing setup screen, or source code is not visual proof.
- Continue troubleshooting missing data, authentication, permissions, routing, or local-runner setup until the evidence attempt is genuinely blocked. Record the commands, state/fixture work, and concrete blocker if a capture still cannot be produced.
- Never use "no fixtures," "too hard to run locally," "small CSS change," or similar convenience reasoning as the explanation for skipping an attempt.

Use the appropriate QA Instructions section for steps and expected results; use `## Visual Proof` for each artifact:

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

## High-Risk Items for Human Review

Keep `## ⚠️ High-Risk Items for Human Review` immediately before `## How to Test`. Use this exact block when the assessment flags one or more items:

```markdown
## ⚠️ High-Risk Items for Human Review

> Automated risk assessment flagged the following architectural or operational touchpoints for human verification.

| Area | Severity | Review Context / Risk |
| :--- | :--- | :--- |
| **[Category]** | [🚨 Critical / ⚠️ Warning / ℹ️ Notice] | [`filename#Lline`](pr-diff-link)<br><br>[Concise description of the change and explicit risk context] |
```

Add one row for each distinct risk location. If the assessment finds no matching items, render the heading followed by:

```markdown
*No high-risk architectural boundaries, API breaks, query safety issues, or external-service reliability risks detected.*
```

In each review-context cell, show only the filename and changed line or range—never the full path—as the link label. Follow it with `<br><br>` so a blank line renders before the risk explanation. Do not omit the section, use blob or repository-file links, or report category names without identifying the exact changed lines and why human sign-off is required.

## Updates

On update, replace stale `Product Owner QA Instructions`, `Engineering QA Instructions`, `Automated Testing Instructions`, `Visual Proof`, and `High-Risk Items for Human Review` content; preserve other content and requirements.

Do not assume a platform or CLI. The active platform skill handles request operations; this skill supplies testing content or a ready-to-paste section.
