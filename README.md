# Agent Skills

Reusable agent skills maintained in this repository. Skills follow the [Agent Skills](https://agentskills.io) format and install with the [`skills` CLI](https://github.com/vercel-labs/skills).

```bash
npx skills add jturmel/skills
```

That command discovers the four skill directories at the repository root. The `unpushed-work` skill is nested under `git/`, so install it separately; see [install.md](install.md) for the exact command and options.

## Skills

| Skill | Description |
|-------|-------------|
| [documenting-change-validation](documenting-change-validation/SKILL.md) | Add accurate automated, engineering, product-owner, and visual validation guidance to pull/merge requests. |
| [merging-dependency-updates](merging-dependency-updates/SKILL.md) | Process dependency-update pull request queues safely, with fresh state checks and controlled merges. |
| [setup-monorepo-django](monorepo-django/SKILL.md) | Set up a new Django project in a monorepo. |
| [superpowers-extensions](superpowers-extensions/SKILL.md) | Keep agent-authored project specs and implementation plans in the repository's canonical `.context/` directories. |
| [unpushed-work](git/unpushed-work/SKILL.md) | Find local branches and worktrees whose work is not safely pushed or merged. **Nested skill; install separately.** |

## Install and update

See [install.md](install.md) for project/global installation, installing the nested skill, verification, and updates.

## Contributing

Skills and their supporting references are authored and maintained here. Edit the relevant skill directory and keep its `SKILL.md`, references, and README description consistent. Issues and pull requests about this repository belong here.
