# Agent Skills Repository

This repository contains reusable agent skills maintained directly in this repo. It is not a generated mirror of another project. For user-facing installation instructions, see [install.md](install.md); for the skill catalog, see [README.md](README.md).

## Source of truth

- Each skill lives with its `SKILL.md` and any skill-specific `references/` or `agents/` files.
- Edit the owning skill here. Keep its frontmatter `name` and `description` accurate; the name is the installed skill identifier.
- Keep supporting material close to the skill that uses it. Update links when files move.
- There is no release sync that republishes skills from another repository.

## Discovery and installation

The `skills` CLI discovers the four skill directories at the repository root when installing this repository. `git/unpushed-work` is nested and is not included by that discovery; install it separately using the direct skill URL documented in [install.md](install.md). Keep that distinction accurate in the README and installation guide.

## Repository conventions

- Keep one skill per directory with a `SKILL.md` at that directory's root.
- Use references for long or conditional workflows; direct the skill to read them only when relevant.
- `.context/plans-specs/` is for durable specs and project context; `.context/plans/` is for executable implementation plans. Follow the approval handoff in `superpowers-extensions/SKILL.md` when that workflow applies.
- Avoid introducing a repo-wide build or test framework for documentation-only changes.

## Validation

For skill changes, verify the skill's frontmatter and any referenced paths. Run `npx skills add . --list` to confirm root-level discovery; this lists four skills. Check nested skills with their direct repository path, for example:

```bash
npx skills add https://github.com/jturmel/skills/tree/main/git/unpushed-work --list
```

For documentation edits, verify installation commands and links against the current skill layout. There is no repository-wide build or test command.
