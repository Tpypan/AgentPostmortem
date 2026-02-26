---
description: Deterministically analyze a stored failure and generate typed hypotheses + prevention rules
subtask: true
---

/why-failed

Call the `postmortem_why_failed` plugin tool with flags mapped from `$ARGUMENTS`. Example:

tool: postmortem_why_failed --latest --json
