# Agent Skills

Reusable agent skills maintained in this repository. Skills follow the [Agent Skills](https://agentskills.io) format and install with the [`skills` CLI](https://github.com/vercel-labs/skills). My personal setup also includes the external sources and agents listed in [install.md](install.md).

```bash
npx skills add jturmel/skills
```

That command discovers the four skill directories at the repository root. Three nested skills need separate installation; see [install.md](install.md) for the complete user-wide setup.

## Skills

| Skill | Description |
|-------|-------------|
| [documenting-change-validation](documenting-change-validation/SKILL.md) | Add accurate automated, engineering, product-owner, and visual validation guidance to pull/merge requests. |
| [merging-dependency-updates](merging-dependency-updates/SKILL.md) | Process dependency-update pull request queues safely, with fresh state checks and controlled merges. |
| [setup-monorepo-django](monorepo-django/SKILL.md) | Set up a new Django project in a monorepo. |
| [superpowers-extensions](superpowers-extensions/SKILL.md) | Keep agent-authored project specs and implementation plans in the repository's canonical `.context/` directories. |
| [unpushed-work](git/unpushed-work/SKILL.md) | Find local branches and worktrees whose work is not safely pushed or merged. **Nested skill; install separately.** |
| [superpowers-extensions-plans](superpowers-extensions/plans/SKILL.md) | Route executable plans to `.context/plans/`. **Nested skill; install separately.** |
| [superpowers-extensions-plans-specs](superpowers-extensions/plans-specs/SKILL.md) | Route design specs to `.context/plans-specs/`. **Nested skill; install separately.** |

## Install and update

See [install.md](install.md) for project and user-wide installation, the two external sources, nested skills, verification, and updates.

## Contributing

Skills and their supporting references are authored and maintained here. Edit the relevant skill directory and keep its `SKILL.md`, references, and README description consistent. Issues and pull requests about this repository belong here.
