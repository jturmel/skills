# GitHub PR Visual Proof

Use this for a GitHub pull request with captured visual proof. Keep evidence out of tracked source unless explicitly requested. Upload only relevant, review-safe artifacts without credentials, personal data, or unrelated records.

1. Check `gh pr create --help` or `gh pr edit --help` for `--attach`. When available, write the complete PR body to a temporary Markdown file, reference local images inline in `## Visual Proof`, and pass each referenced image or video with `--attach`. The CLI uploads the files and rewrites matching local references to GitHub-hosted asset URLs. Preserve existing attachment URLs when editing a body.
2. Under `## Visual Proof`, default to this Markdown matrix for screenshots. Keep the size or platform in the left column and Light/Dark on the top axis. Use `N/A` only when that state is unsupported or inapplicable. Replace the example paths with the actual local files and attach each one.

   ```markdown
   | Size / platform | Light | Dark |
   | --- | --- | --- |
   | Mobile — 390 px | ![mobile light](./proof/mobile-light.png) | ![mobile dark](./proof/mobile-dark.png) |
   | Tablet — 768 px | ![tablet light](./proof/tablet-light.png) | ![tablet dark](./proof/tablet-dark.png) |
   | Desktop — 1440 px | ![desktop light](./proof/desktop-light.png) | ![desktop dark](./proof/desktop-dark.png) |
   | Print — Letter | ![print](./proof/print-letter.png) | N/A |
   ```

   For example, `gh pr create --repo OWNER/REPO --base main --head FEATURE --title 'Describe the outcome' --body-file body.md --attach './proof/mobile-light.png#Mobile: light mode'` uploads that file and replaces its matching reference in the body. Repeat `--attach` for each artifact. For an existing PR, use `gh pr edit PR_URL --body-file body.md --attach './proof/mobile-light.png#Mobile: light mode'` with the complete revised body.
3. Retrieve the published body and head with `gh pr view PR_URL --json body,url,headRefOid`. Verify every intended artifact has a GitHub-hosted URL, images render, recordings are playable, no local proof references remain, and the proof describes the validated revision. A local path or review-app URL is not an attachment.
4. Uploads can partially succeed even when the command exits nonzero and the PR was created or updated. Inspect the existing PR, repair only missing uploads, and verify again; do not create a duplicate PR. If `--attach` is unavailable or uploading fails, use another authenticated GitHub upload mechanism. If none is available, report the exact blocker and mark visual proof incomplete; do not commit temporary evidence as a workaround.

Keep `Size / platform` as the left header; use actual widths, device or platform names, and paper size in its rows. For native proof, use rows such as `iPhone 15 — iOS` or `Pixel 9 — Android`. Preserve unrelated PR-body content.

Render the `## ⚠️ High-Risk Items for Human Review` block required by the main skill immediately before `## How to Test`, not next to this `## Visual Proof` section. The risk table links to exact lines in the PR's **Files changed** view; do not reuse screenshot asset URLs or code blob URLs for those locations.

GitHub supports Markdown images and tables in PR bodies: <https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#images> and <https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/organizing-information-with-tables>.
