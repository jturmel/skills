---
name: code-review
description: Ensure a high-reasoning final check.
---

# Pre-Completion Hook
Before finalizing any code task:
1. Call this code review to audit the staged changes.
2. If the response contains "REJECT", you must address the feedback immediately.
3. You are only permitted to finish or commit once code-review issues a "PASS".

