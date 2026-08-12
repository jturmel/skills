---
name: superpowers-extensions
description: "Use with superpowers to centralize agent-authored project context in `.context/plans-specs` and `.context/plans`."
---

# Manage Project Context

Use this with superpowers to keep agent-authored project context in its two canonical directories.

## STRUCTURE

```text
./.context
├── plans-specs/  # Design specs, discovery notes, and durable current-state references
└── plans/        # Executable implementation plans
```

## ROUTING

- Save design specs, discovery notes, durable current-state references, and concise feature context in `.context/plans-specs/`.
- Save executable implementation plans in `.context/plans/`.
- Do not create another `.context` subdirectory. Relocate any older artifact outside these two directories and repair its repository references.

## APPROVAL HANDOFF

Preserve Superpowers' native sequence: approved spec → plan creation → approved plan → implementation.

- When a spec is written, present the complete artifact itself in the response and link to its exact `.context/plans-specs/...` file. Ask whether the spec is approved. Do not create the implementation plan until the user approves the spec.
- After spec approval and plan creation, present the complete plan itself and link to its exact `.context/plans/...` file. Ask whether the plan is approved. Do not start implementation or execute the plan until the user approves the plan.
- “Present” means show the artifact, not a summary of it. Preserve the built-in Superpowers handoff and approval questions.
