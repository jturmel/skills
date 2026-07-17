# Documenting Change Validation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create and globally install a platform-neutral Codex skill that maintains required `How to Test`, `Automated`, and `Manual QA` content in pull/merge request bodies.

**Architecture:** A standalone skill directory at `documenting-change-validation/` will contain concise trigger metadata, imperative workflow guidance, and generated UI metadata. It will not contain platform adapters, scripts, or project-specific commands.

**Tech Stack:** Markdown, YAML front matter, `skill-creator` initialization and validation scripts, Git.

## Global Constraints

- Keep the skill separate from `superpowers-extensions`.
- Keep front matter concise and limited to `name` and `description`.
- Remain agnostic to Git hosting platforms and CLIs.
- Preserve request-body content owned by other skills.
- Require `## How to Test` with `### Automated` and `### Manual QA`.
- Never invent repository test or verification commands.

---

### Task 1: Initialize the skill scaffold

**Files:**
- Create: `documenting-change-validation/SKILL.md`
- Create: `documenting-change-validation/agents/openai.yaml`

- [x] **Step 1: Run the standard initializer**

Run:

```bash
python /home/jt/.codex/skills/.system/skill-creator/scripts/init_skill.py documenting-change-validation --path /home/jt/dev/jturmel/skills --interface display_name="Documenting Change Validation" --interface short_description="Keep pull/merge request testing guidance accurate" --interface default_prompt="Maintain the How to Test section for this pull/merge request."
```

Expected: the two skill files are created under `documenting-change-validation/`.

### Task 2: Write the skill instructions

**Files:**
- Modify: `documenting-change-validation/SKILL.md`

- [x] **Step 1: Replace the initializer body with the approved workflow**

Include concise front matter and imperative instructions covering trigger, repository inspection, shared-body preservation, exact required headings, automated evidence, manual QA, update behavior, ambiguity handling, and platform neutrality.

- [x] **Step 2: Check the skill for scope and wording issues**

Run:

```bash
wc -w documenting-change-validation/SKILL.md
rg -n "TODO|TBD|FIXME|GitHub-only|invent" documenting-change-validation/SKILL.md
```

Expected: no placeholders, no platform lock-in, and a compact body.

### Task 3: Validate and install

**Files:**
- Verify: `documenting-change-validation/SKILL.md`
- Verify: `documenting-change-validation/agents/openai.yaml`
- Install: `/home/jt/.codex/skills/documenting-change-validation/`

- [x] **Step 1: Run skill validation**

Run:

```bash
python /home/jt/.codex/skills/.system/skill-creator/scripts/quick_validate.py documenting-change-validation
```

Expected: validation succeeds. The bundled validator could not run because `PyYAML` is not installed; equivalent Ruby YAML and structural checks passed.

- [x] **Step 2: Compare local and global copies**

Install the complete skill directory into `/home/jt/.codex/skills/documenting-change-validation/`, then run:

```bash
diff -ru documenting-change-validation /home/jt/.codex/skills/documenting-change-validation
```

Expected: no differences.

- [x] **Step 3: Commit the skill**

```bash
git add documenting-change-validation
git commit -m "Add change validation documentation skill"
```
