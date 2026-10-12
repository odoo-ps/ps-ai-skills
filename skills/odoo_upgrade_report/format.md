# UPGRADE.md format (one file per module)

Location: `<module>/UPGRADE.md`. The top half is for functional, the bottom half for the reviewer. Keep the headings
exactly as below, in this order: the compile step finds sections by heading. Leave a section out only when it would be
empty, except **Test scenarios**, which every module has.

```markdown
# <module> — Upgrade <from> → <to>

> **Risk:** Low | Medium | High · **State:** Installs, tests pass | Installs, no automated tests | Blocked: <why>

<One or two sentences: what this module does for users, in their words.>

## For functional

### What changes for users

| Change | Where | Kind | Decision needed |
| --- | --- | --- | --- |

### Test scenarios

| ID | Scenario | Steps | Expected | Priority | Dev check |
| --- | --- | --- | --- | --- | --- |

### Decisions needed

1. **<Question to the client or functional>** — <what changed, options, what happens if nobody decides>

### Manual checks required

- <Screen / report / email / automation> — <what to look at, and why it may differ>

### Not verifiable here

- <What> — <why it cannot be tested on this database> — <what environment it needs>

## Technical log

### Changes

- **<What changed>** — <why, in one or two sentences>. `Source: <40-char sha>` (odoo|enterprise)

### Verification run

- `<command>` — <result>

### Pre-existing issues

- <Issue> — not introduced by the upgrade — <fixed here | left, because …>

### Later fixes

| Date | Ticket | Commit | What was fixed |
| --- | --- | --- | --- |
```

## Section by section

**Risk.** High: a business flow, money, a document sent to customers or a security rule changed behaviour, or a
decision is open. Medium: views, reports or automations were reworked but behave as before. Low: version bump and
mechanical renames only.

**What changes for users.** One row per change a user can notice. `Where` is a menu path or a named screen, report or
email (`Sales > Orders > Print > Quotation`). `Kind` is one of:

| Kind | Meaning | Must also have |
| --- | --- | --- |
| Restored | Broke in the target version, fixed, behaves as before | a test scenario |
| Standard change | Odoo itself changed it; the customisation follows | nothing (it feeds triage of "is this a bug?") |
| Custom behaviour changed | The customisation now behaves differently | a decision or a scenario that shows the new behaviour |
| Removed | A customisation or field is gone | a decision, unless the removal is a standard change |

Mechanical changes that users cannot see (tree → list, `_sql_constraints`, imports) do not get a row.

**Test scenarios.** What a consultant clicks to prove the module still works. ID is `<module>-T<n>`, stable across
edits. Steps are numbered, start from a menu, and use demo-like data the tester can create. Expected is observable on
screen or in a printed or sent document. Priority:

- **P1** a core flow (quote to cash, procure to pay, field intervention), money, a document sent to customers, or
  access rights
- **P2** used weekly, or a screen a team works in every day
- **P3** rare or cosmetic

`Dev check` says what the developer already ran on this flow: `Auto test <name>`, `Shell`, `Rendered` (render gate),
`HTTP call`, or `None`. It never says "OK" without naming the check. Add a scenario for every touched view, report,
mail template, onchange, cron, controller and record rule, and one for each "Restored" or "Custom behaviour changed"
row.

**Decisions needed.** Only questions a developer cannot answer: a lost behaviour, data no longer stored, a security
rule that needs a design choice. Each one says what happens if nobody decides.

**Manual checks required.** Customisations that live in the customer database and are invisible from source. Name the
model and screen and what could break: "Studio fields on the Sales Order form placed next to Taxes: the column was
renamed". This heading is the one `odev upgrade` asks for, keep its exact name. Never put a check you already ran here:
that belongs in **Verification run**.

**Not verifiable here.** External systems (SFTP, payment providers, third-party APIs), production volumes, anything
that needs real customer data.

**Changes.** One bullet per discrete fix, mirroring the commits. `Source:` holds a resolved SHA, or
`Source: not identified` plus the core file compared against.

**Verification run.** The commands actually run and what they showed. This is where the old "Functional Testing"
content goes.

**Later fixes.** Fixes made after the first delivery, usually from functional tickets. Add a row for each, and add or
update the matching test scenario so it is retested. The compile step also reads `git log`, but a row here carries the
user-facing wording.

## Legacy files

Reports written before this format typically have `Technical Summary`, `Functional Testing` and
`Manual checks required`, plus free headings such as "Confirmed gap", "Pre-existing issue" or "Functional regression".
When a module is touched again, rewrite its report into this format. The compile step can read legacy files, but it
has to derive scenarios from the technical text and marks them as derived.
