```prompt
---
agent: nist-evidence.assess
---

Before running, enforce this first-run bootstrap checklist:

1. Treat missing artifacts as normal on first run.
2. Ensure `.nist-evidence/` exists.
3. If tasks file is missing, run plan first to create `./.nist-evidence/nist-evidence-tasks.md`.
4. If report JSON is missing, initialize a fresh report and continue.
5. Always write outputs to `./.nist-evidence/` by default:
   - `nist-evidence-report.json`
   - `nist-evidence-summary.md`
6. Never fail only because prior artifacts are absent.

Then execute normal assessment behavior.
```
