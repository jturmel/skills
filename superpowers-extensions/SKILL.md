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
