---
name: superpowers-extensions-plans
description: Must be used with superpowers/writing-plans. Overrides plan output to `.context/plans`.
---

# Writing Plans Addendum

## Use with superpowers/writing-plans

When generating implementation plans, save them to:

`.context/plans/YYYY-MM-DD-<feature-name>.md`

The rest of the superpowers `writing-plans` workflow applies; only the save location is changed.

After creating the plan, present the complete artifact in the response and link to the exact file. Ask the user to approve the plan or request changes. Do not execute the plan or begin implementation until the user approves it; plan approval is the handoff into implementation.
