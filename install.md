# Install Agent Skills

Use the [`skills` CLI](https://github.com/vercel-labs/skills) to install these skills into a supported coding agent. The CLI requires Node.js/npm. Installing the skills does not install or authenticate any product-specific CLI.

## My user-wide setup

Make these skills available in every local project for **OMP, Claude Code, and Codex**. The source of truth for the internal skills is this repository; the external sources are [obra/superpowers](https://github.com/obra/superpowers) and [danyuchn/asd-ste100-skill](https://github.com/danyuchn/asd-ste100-skill). Install complete skill directories, including their references and scripts.

| Internal skill name | Source directory |
| --- | --- |
| `documenting-change-validation` | `documenting-change-validation/` |
| `merging-dependency-updates` | `merging-dependency-updates/` |
| `setup-monorepo-django` | `monorepo-django/` |
| `superpowers-extensions` | `superpowers-extensions/` |
| `unpushed-work` | `git/unpushed-work/` |
| `superpowers-extensions-plans` | `superpowers-extensions/plans/` |
| `superpowers-extensions-plans-specs` | `superpowers-extensions/plans-specs/` |

At user scope, expose each directory as an immediate child of `~/.agents/skills/` under its skill name. Symlinks to this checkout keep the internal skills current when the checkout changes. OMP and Codex scan that user directory. Expose the same skill directories to Claude Code under `~/.claude/skills/`; symlinks to the shared user directory work. Nested source skills need their own top-level links because OMP does not scan nested skill directories recursively. Keep only one maintained copy of each ordinary skill.

Install **Superpowers as a host plugin/package** so its startup integration works: use `omp plugin install git:github.com/obra/superpowers` when OMP's package dependencies are available, or link a persistent checkout with `omp plugin link /path/to/superpowers`; install `superpowers@superpowers-marketplace` from the author's Claude Code marketplace; and install its Codex plugin from a configured marketplace. The [upstream installation guide](https://github.com/obra/superpowers#installation) is authoritative if these commands change. Copying Superpowers' `skills/` directory alone does not provide the host startup integration.

For `asd-ste100-skill`, install the complete directory from the upstream repository into the shared user skill root and expose it to Claude Code. Record a source revision or plugin version whenever either external source changes. On 2026-10-14, the source revisions used on this machine were `obra/superpowers` `bb92a77741419a4ab5f06e711a283343f1ada0c3` (7.0.0) and `danyuchn/asd-ste100-skill` `3944283c183eea13fe5bd902b572da3b4de8bf44`.

After installation or update, check that every directory has `SKILL.md`, the symlinks resolve, and a fresh OMP, Claude Code, and Codex session discovers the intended skills. Check for duplicate or disabled older installs before declaring the setup complete.

## Install the root skills

Run this in the project where you want the skills available:

```bash
npx skills add jturmel/skills
```

The CLI discovers the four skill directories at the repository root and prompts for the agent and installation method. To see the available root skills without installing them:

```bash
npx skills add jturmel/skills --list
```

To target Codex explicitly, add `--agent codex`. Add `--global` (or `-g`) to install for your user instead of the current project. Use `--skill <name>` to choose specific skills, for example:

```bash
npx skills add jturmel/skills --skill merging-dependency-updates
```

## Install `unpushed-work`

`unpushed-work` is stored under `git/unpushed-work/`, not at the repository root. The repository-level install does not discover it, so add it separately:

```bash
npx skills add https://github.com/jturmel/skills/tree/main/git/unpushed-work
```

Add `--list` to inspect it without installing. You can also use `--agent codex` or `--global` with this command.

The same root-level discovery limit applies to `superpowers-extensions/plans/` and `superpowers-extensions/plans-specs/`. Install or link each nested skill explicitly under its frontmatter name.

## Verify and update

List installed skills:

```bash
npx skills list
```

The list should include whichever root skills you selected. If you also installed `unpushed-work`, verify that it appears as well. Restart the agent session if it does not load newly installed skills.

Update installed skills with:

```bash
npx skills update
```

## Manual installation

For an installation flow supported by your agent but not handled by the CLI, follow the agent's skill-directory conventions and copy the desired skill directory, preserving `SKILL.md` and any adjacent references or other required files. The root skills are the four directories linked in [README.md](README.md); `unpushed-work` is at `git/unpushed-work/`.
