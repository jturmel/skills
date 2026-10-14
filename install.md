# Install Agent Skills

Use the [`skills` CLI](https://github.com/vercel-labs/skills) to install these skills into a supported coding agent. The CLI requires Node.js/npm. Installing the skills does not install or authenticate any product-specific CLI.

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
