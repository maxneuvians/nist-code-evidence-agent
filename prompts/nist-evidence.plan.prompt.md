```prompt
---
agent: nist-evidence.plan
---

Before running, enforce this first-run bootstrap checklist:

1. Treat missing artifacts as normal on first run.
2. Ensure `.nist-evidence/` exists.
3. Write tasks to `./.nist-evidence/nist-evidence-tasks.md` unless overridden.
4. If `.gitignore` does not include `.nist-evidence/`, add it.
5. Never fail only because artifacts are missing; create them.

Then execute normal planning behavior.
```
