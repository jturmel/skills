# GitHub PR Visual Proof

Use this for a GitHub pull request with captured visual proof. Keep evidence out of tracked source unless explicitly requested. Upload only relevant, review-safe artifacts. Never upload credentials, customer data, personal data, or other sensitive content, such as tokens in address bars, internal hostnames, or unrelated records.

1. Check `gh pr create --help` or `gh pr edit --help` for `--attach`. When available, write the complete PR body to a temporary Markdown file, reference local images inline in `## Visual Proof`, and pass each referenced image or video with `--attach`. Attach only files the body references; the CLI appends any unreferenced attachment to the end of the body, publishing it. The CLI uploads the files and rewrites matching local references to GitHub-hosted asset URLs. Preserve existing attachment URLs when editing a body.
2. Save each capture at a unique path, `./proof/<item>/<row>-<theme>.<ext>`. `<item>` is a short slug for the screen or state shown, such as `checkout-empty-cart`; `<row>` is the size, platform, or paper size, such as `mobile`; `<theme>` is `light` or `dark`. Never reuse a path for a different capture.
3. Under `## Visual Proof`, default to this Markdown matrix for screenshots. Keep the size or platform in the left column and Light/Dark on the top axis. Use `N/A` only when that state is unsupported or inapplicable. When proving more than one item, repeat the matrix once per item under a line naming that item. Replace `<item>` and the example rows with the actual local files and attach each one.

   ```markdown
   | Size / platform | Light | Dark |
   | --- | --- | --- |
   | Mobile — 390 px | ![<item> mobile light](./proof/<item>/mobile-light.png) | ![<item> mobile dark](./proof/<item>/mobile-dark.png) |
   | Tablet — 768 px | ![<item> tablet light](./proof/<item>/tablet-light.png) | ![<item> tablet dark](./proof/<item>/tablet-dark.png) |
   | Desktop — 1440 px | ![<item> desktop light](./proof/<item>/desktop-light.png) | ![<item> desktop dark](./proof/<item>/desktop-dark.png) |
   | Print — Letter | ![<item> print](./proof/<item>/print-light.png) | N/A |
   ```

   For example, `gh pr create --repo OWNER/REPO --base main --head FEATURE --title 'Describe the outcome' --body-file body.md --attach './proof/<item>/mobile-light.png'` uploads that file and replaces its matching reference in the body. Repeat `--attach` for each artifact. For an existing PR, use `gh pr edit PR_URL --body-file body.md --attach './proof/<item>/mobile-light.png'` with the complete revised body.
4. Retrieve the published body and head with `gh pr view PR_URL --json body,url,headRefOid`. Verify every intended artifact has a GitHub-hosted URL, no unintended attachment was appended to the body, images render, recordings are playable, no local proof references remain, and the proof describes the validated revision. A local path or review-app URL is not an attachment.
5. Uploads can partially succeed even when the command exits nonzero and the PR was created or updated. Inspect the existing PR, repair only missing uploads, and verify again; do not create a duplicate PR. If `--attach` is unavailable or uploading fails, use only another GitHub upload mechanism that attaches the file to this pull request, so it inherits the PR repository's visibility. Never use gists, other repositories, or external hosts. If no such mechanism is available, report the exact blocker and mark visual proof incomplete; do not commit temporary evidence as a workaround.

Keep `Size / platform` as the left header; use actual widths, device or platform names, and paper size in its rows. For native proof, use rows such as `iPhone 15 — iOS` or `Pixel 9 — Android`. Preserve unrelated PR-body content.

Render the `## ⚠️ High-Risk Items for Human Review` block required by the main skill immediately before `## How to Test`, not next to this `## Visual Proof` section. The risk table links to exact lines in the PR's **Files changed** view; do not reuse screenshot asset URLs or code blob URLs for those locations.

GitHub supports Markdown images and tables in PR bodies: <https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#images> and <https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/organizing-information-with-tables>.
