---
name: odoo_upgrade_report
description:
    "Write a module's UPGRADE.md so functional can test from it, and compile every UPGRADE.md of an upgraded
    project into one FUNCTIONAL_HANDOFF.md: decisions, a single test list by business flow, database checks."
---

# Odoo Upgrade Report

Two jobs, one contract between them:

| Job | When | Read |
| --- | --- | --- |
| **Write** a module's `UPGRADE.md` | Step 4 of `odoo_upgrade_skill`, once per module, and again whenever a later fix lands on the module | [format.md](format.md) |
| **Compile** the project's `FUNCTIONAL_HANDOFF.md` | After the last module, before the database goes to functional; again after each batch of ticket fixes | [handoff.md](handoff.md) |

Examples: [examples/UPGRADE.md](examples/UPGRADE.md) (one module, new format) and
[examples/FUNCTIONAL_HANDOFF.md](examples/FUNCTIONAL_HANDOFF.md) (the compiled result).

## Why this exists

An `UPGRADE.md` has two readers. The reviewer needs the technical log: what broke, the upstream commit, what was
verified. The functional consultant needs something else: what users will see differently, what to click to prove it
still works, and which questions only the client can answer. When both are mixed in one free-form file, the functional
part is buried, the "Functional Testing" section ends up listing what the *developer* ran in `odev shell`, and nobody
can merge twelve such files into one test plan without re-reading the code.

The format fixes the headings so the compile step can rely on them. The compile step turns many module reports, plus
the fixes that landed after them, into one list ordered the way a consultant tests: by business flow, most critical
first.

## Rules that apply to both jobs

- **User language in everything functional reads.** Menu paths, screen names, field labels and report names as the
  user sees them. A model or method name appears only in the technical log, or in parentheses after the user term
  when it removes ambiguity.
- **Never invent.** Every scenario, decision and check traces to a sentence in an `UPGRADE.md` or to a commit. A menu
  path you are not sure of is written with `(path?)` so a human fixes it, never guessed silently.
- **No unverified claims.** "Verified", "tested" and "works" only for a check that was actually run, named in the
  "Dev check" column. A scenario derived from a code change rather than from a test is marked as derived.
- **Install and tests green is not "works".** Onchanges, crons, reports, mail templates, controllers and record rules
  run only when a user or a schedule triggers them. Each one touched gets a test scenario.
- **The database is out of reach.** Studio views, automations, edited mail templates and saved filters live in the
  customer database. Name the specific screens and records to check; never write "check Studio" alone.
