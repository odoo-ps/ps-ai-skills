# Compiling FUNCTIONAL_HANDOFF.md

Input: an upgraded repository (the upgrade branch checked out). Output: one `FUNCTIONAL_HANDOFF.md` that a functional
consultant can test from without opening the repository. Default location is the repository root, **not committed**
unless asked: the file is meant to be posted on the Odoo task or shared, not reviewed as code.

## 1. Inventory

```bash
find . -name UPGRADE.md -not -path './.git/*' | sort
git log -1 --format='%cs %h' -- <module>/UPGRADE.md          # when each report was last written
git log --since=<that date> --format='%h %cs %s' -- <module>  # what landed on the module after it
```

Build the list of upgraded modules from `TASKS.md` if present, else from every directory with a `__manifest__.py`.
A module without an `UPGRADE.md` is a gap: list it, never skip it silently. A directory without a manifest (import
scripts, tooling) is not a module.

For each report note whether it follows [format.md](format.md) (it has `## For functional`) or is legacy.

## 2. Extract, per module

**New format:** take the tables as they are.

**Legacy format:** map by heading, then derive.

| Legacy heading | Goes to |
| --- | --- |
| Technical Summary | Changes; derive scenarios and standard changes from it |
| Functional Testing | Dev check of the matching scenarios (what was already run), never a scenario by itself |
| Manual checks required | Database checks; any item that asks "acceptable?" or "needs a decision" becomes a decision |
| "gap", "regression", "not preserved", "needs a decision" | Decisions |
| "pre-existing", "test fix", "mismatch" | Pre-existing findings |
| A line marked ✅ inside Manual checks | Dev check, not a database check |

Derive one scenario per user-reachable change: a view or report re-anchored, a mail template, an onchange, a cron, a
controller or API endpoint, a record rule, a removed or collapsed feature. Mark derived scenarios `(derived)` in the
Dev check column so the developer can confirm them. A change that only renamed a tag or an import produces nothing.

## 3. Later fixes

Every `[UPG]`, `[FIX]` or `[IMP]` commit on a module after its report's date is a fix the report does not describe,
usually raised by functional on a ticket. Read each commit body (`git log -1 --format=%B <sha>`), restate it in user
words, and list it under **Fixed after the first delivery** with its ticket id from the `[UPG][<id>]` header. Each one
gets a retest line. Skip commits that only trigger CI or bump requirements.

## 4. Merge into one list

Group by business flow, not by module. Use these groups and drop the empty ones:

Sales · CRM · Purchase · Inventory · Invoicing & EDI · Project & Field Service · Documents · Integrations & APIs ·
Users & access · Reports & emails (only for items that belong to no flow above)

Within a group, order by the flow's own sequence (lead → quote → order → delivery → invoice → payment follow-up),
then by priority. When two modules touch the same screen or flow, merge into one row and list both modules.
Database checks are merged per screen: one line for the Sales Order form, whatever number of modules asked for it.

IDs in the handoff are `<group letter><n>` (S1, C3, I2…), with the source module in its own column, so the consultant
can report "S4 fails" on a ticket and the developer finds the module.

## 5. Write

```markdown
# Functional handoff — <project> <from> → <to>

<Branch, date, N modules, N scenarios (N P1), N decisions, N later fixes to retest. One line on how to use it.>

## 1. Decisions needed before sign-off
| # | Decision | Module | If nobody decides |

## 2. Test list
### <Group>
| ID | What to test | Steps | Expected | Pri | Already checked by dev | Module | Result |

## 3. Fixed after the first delivery — retest
| Ticket | What was wrong | Retest | Module | Commit |

## 4. Check in the customer database
| Screen / record | What to look for | Module |

## 5. Not testable on this database
## 6. Standard Odoo changes users will notice (not bugs)
## 7. Questions for the developer
## Appendix: coverage by module
| Module | Report | Risk | Scenarios | Later fixes | Automated tests |
```

The `Result` column is left empty for the tester. Section 6 is what turns "this looks different" tickets into a
quick answer instead of a dev investigation: list each standard change once, with the screen where users meet it.

## 6. Check before handing over

- Every decision and every "Manual checks required" item from every report appears in sections 1 or 4.
- Every later fix from step 3 appears in section 3.
- Every module appears in the appendix, with its scenario count.
- Contradictions between sections of one report (the test says one thing, the manual check another), and scenarios
  whose menu path ends in `(path?)`, are listed in section 7 rather than resolved by guessing.
- No customer data copied beyond what the reports already contain (no partner names, amounts, credentials).
