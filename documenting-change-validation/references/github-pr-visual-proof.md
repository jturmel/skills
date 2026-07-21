# GitHub PR Visual Proof

Use this only for a GitHub pull request with captured screenshots.

1. Copy review-safe screenshots to `.github/tmp/<change-slug>/`, naming each `<row>-<theme>.png`. Stage, commit, and push those files before updating the PR body. Use that pushed artifact commit SHA as `<asset-commit>`; never use `main`. Do not commit credentials, customer data, or other sensitive content.
2. Reference each image inline with GitHub's same-repository relative URL form:

   ```markdown
   ![<alt text>](../blob/<asset-commit>/.github/tmp/<change-slug>/<file>?raw=true)
   ```

3. Under `## Visual Proof`, default to this Markdown matrix. Keep the size or platform in the left column and Light/Dark on the top axis. Use `N/A` only when that state is unsupported or inapplicable.

   ```markdown
   | Size / platform | Light | Dark |
   | --- | --- | --- |
   | Mobile — 390 px | ![mobile light](../blob/<asset-commit>/.github/tmp/<change-slug>/mobile-light.png?raw=true) | ![mobile dark](../blob/<asset-commit>/.github/tmp/<change-slug>/mobile-dark.png?raw=true) |
   | Tablet — 768 px | ![tablet light](../blob/<asset-commit>/.github/tmp/<change-slug>/tablet-light.png?raw=true) | ![tablet dark](../blob/<asset-commit>/.github/tmp/<change-slug>/tablet-dark.png?raw=true) |
   | Desktop — 1440 px | ![desktop light](../blob/<asset-commit>/.github/tmp/<change-slug>/desktop-light.png?raw=true) | ![desktop dark](../blob/<asset-commit>/.github/tmp/<change-slug>/desktop-dark.png?raw=true) |
   | Print — Letter | ![print](../blob/<asset-commit>/.github/tmp/<change-slug>/print-letter.png?raw=true) | N/A |
   ```

Keep `Size / platform` as the left header; use actual widths, device or platform names, and paper size in its rows. For native proof, use rows such as `iPhone 15 — iOS` or `Pixel 9 — Android`. Preserve unrelated PR-body content.

GitHub documents relative repository image links for pull requests and comments, and supports Markdown tables in PR bodies: <https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#images> and <https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/organizing-information-with-tables>.
