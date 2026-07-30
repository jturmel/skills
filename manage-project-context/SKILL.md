---
name: manage-project-context
description: Use with superpowers to keep agent-authored project context in `.context/plans-specs` and `.context/plans`.
---

# Manage Project Context

## STRUCTURE

```text
./.context
├── plans-specs/  # design specs, discovery notes, and durable current-state references
└── plans/        # executable implementation plans
```

- Save design specs, discovery notes, durable current-state references, and concise feature context in `.context/plans-specs/`.
- Save executable implementation plans in `.context/plans/`.
- Do not create another `.context` subdirectory.
