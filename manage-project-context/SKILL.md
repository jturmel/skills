---
name: manage-project-context
description: Use to keep a project's context up-to-date, this includes
  features built, plan specs, and plans for active work as the three areas that
  form the project context.
---

# Manage Project Context

## Overview

Must be used in conjuction with the superpowers plugin to get the full benefit.

## STRUCTURE
```text
./.context
├── features/    # project feature knowledge base, they describe what the project does
├── plan-specs/  # what is being brainstormed, managed by the superpowers/brainstorming skill, overrides default save location
└── plans/       # what is being worked on, managed by the superpowers/writing-plans skill, overrides default save location
```

