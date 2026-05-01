---
name: test_skill
description: "AI Test Failure Resolution Rules for Odoo development."
---

# Odoo Test Failure Resolution Rules

This skill provides mandatory rules for handling test failures during AI-assisted testing.

## 🧪 AI TEST FAILURE RESOLUTION RULES

When running Odoo tests, analyze any failures and apply exactly one of the three rules below — nothing else:

1. **RULE 1 — Custom test fails**:
   The custom module code is broken (e.g. upgrade logic did not preserve the expected workflow).
   → **Fix the custom module CODE** so the workflow is correct again. Never weaken or skip the test.

2. **RULE 2 — Standard Odoo test fails because of our custom code**:
   Our changes altered a workflow or model that a standard test relied on.
   → **Monkey-patch the standard test** from within the custom module so it aligns with the new workflow.
   Never modify standard Odoo files. Never make the test trivially pass by removing assertions.

3. **RULE 3 — Standard Odoo test fails for an unrelated reason**:
   → **Do nothing**. Skip it and move on.

After applying a fix, re-run the tests and repeat until all Rule-1 and Rule-2 failures are resolved.
Finish with a summary of every change made and which rule justified it.
