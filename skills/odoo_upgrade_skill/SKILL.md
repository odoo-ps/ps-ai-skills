---
name: odoo_upgrade_skill
description: "Expert workflow for Odoo module upgrades (Migration, Refactoring, Resolution)."
---

# Odoo Upgrade Workflow Skill

This skill provides a structured, step-by-step process for upgrading Odoo modules between versions.

> [!IMPORTANT]
> **CRITICAL - Feature Reimplementation Rule**:
> If a feature, method, field, model, or functionality was removed or deprecated in the target Odoo version, **do not** simply delete or disable it if it was being used. Instead, you MUST try to **reimplement** it or find an equivalent target Odoo API/mechanism to preserve the business logic and behavior. Do not lose existing functionality during the upgrade.
> The single narrow exception — an upstream concept that no longer exists at all — is defined in Step 3 and carries a hard evidence bar; it is not a general licence to delete.

## 🚀 UPGRADE WORKFLOW (Strict process for EACH module)

Apply this workflow strictly for each module in the list:

### Step 1: Manifest & Preparation
- **Update Version**: Set the version in `__manifest__.py` to `[target_ver].1.0.0`.
- **Atomic Commit**: Commit the version bump immediately:
  - **Message**: `[UPG][[task_id]] <module_name>: bump version to [target_ver].1.0.0`

### Step 2: Automatable Refactoring (XML Views)
- **Convert Views**: Use `odev upgrade-code -V [target_ver] <db> --from [from_ver] --to [target_ver] --glob "<module_name>/**"`. This is the MANDATORY tool for XML migrations.
- **Atomic Commit**: Commit the XML syntax changes:
  - **Message**: `[UPG][[task_id]] <module_name>: adapt XML views to Odoo [target_ver] syntax`

### Step 3: Iterative Resolution (Python, Security & Feature Reimplementation)
- **Fail Fast (Static Validation)**: Before launching Odoo, validate your changes:
  1. Python: `ruff check path/to/file.py --select E999,F821,F822,F405`. DO NOT fix other lint errors.
  2. XML: `xmllint --noout path/to/file.xml`.
- **Run Verification**: Launch `odev run -V [target_ver] <db> -i <module_name> --stop-after-init`.
- **Log Filtering**: If Odoo fails, DO NOT dump the full log. Use `grep -A 10 -B 10 "Traceback" <log_file>` or `tail -n 20` to extract only the relevant error.
- **Data Migration Reflection**: If you modify models (renaming fields, changing types, deleting tables), assess the impact on production data. Use `odoo_upgrade_utils` to create `pre-migrate.py` or `post-migrate.py` if needed.
- **Handling Removed/Deprecated Features**:
  - **REIMPLEMENT**: If a feature, method, field, model, or functionality was removed or deprecated in the target Odoo version, **do not** simply delete or disable it if it was being used. Instead, try to **reimplement** it or find an equivalent target Odoo API/mechanism to preserve the business logic and behavior.
  - **The narrow exception, and its price**: if the *concept* the customisation depends on is gone upstream — not renamed, not moved, and with no equivalent API — reimplementing it is impossible and pretending otherwise ships dead code. Only then may you **delete the customisation outright**, and only if you can do all three: (a) **resolve** the upstream commit that removed the concept and put it on the `Source:` line — `Source: not identified` is **never** acceptable for a deletion; if you cannot resolve it, keep the code and escalate instead; (b) state in the commit body which equivalent APIs you checked and rejected; (c) record the removed behaviour in `UPGRADE.md` under functional-testing scenarios, so a human can confirm the loss is acceptable. Never leave a stub behind (see Override Boundaries below): a reviewer cannot tell an intentional removal from an accidental one without the citation, and will either restore broken code or spend a day re-deriving your conclusion.
- **Override boundaries** (these fail *silently* — the module installs and tests stay green):
  1. **Signature parity**: before editing an override, print the target's signature and match it exactly. `<target_odoo_path>` is a **container** holding one checkout per repository, so always search the checkout: `grep -rn "def <method>" <target_odoo_path>/odoo/addons/<app>/models/ -A3` for a community addon, `<target_odoo_path>/odoo/odoo/addons/base/models/` for core models (`res.partner`, `ir.*`), `<target_odoo_path>/enterprise/<module>/models/` for enterprise (enterprise has no `addons/` level). Never add, drop, or rename a parameter you have not seen upstream: a forwarded keyword that does not exist upstream raises only on the code path that passes it.
  2. **Never reduce an override to a bare `super()` call**, and never leave it as `pass`. Either keep working logic or delete the method entirely (with the citation above). A stub that keeps the signature and the docstring while dropping the body reads as intentional and silently removes a customer feature.
  3. **Compute methods**: every `@api.depends` path must resolve on the model, and a stored compute must declare dependencies — an incomplete or invalid `depends` does not raise, the field just stops recomputing. In a compute loop assign to the loop variable, never to `self`.
- **If an error occurs (Python, Security, etc.)**:
  1. **SEARCH origin**: Use `git log -S "..."`, `git log -L`, or `grep` in Odoo target source code to find the exact Commit Hash that introduced the Breaking Change.
  2. **FIX**: Correct the code in our module.
  3. **ATOMIC COMMIT**: Make a commit ONLY for this specific fix.
     - **Format**: `[UPG][[task_id]] <module_name>: description`
     - **Body**: You MUST include the Odoo Hash found (made clickable) and a brief explanation of the breaking change. This is MANDATORY as it will be used in the final synchronization step.
- **Repeat** Step 3 until the module installs and runs without errors.

### Step 4: Module Closure
- **Check Progress**: Mark the module as `[x]` in `TASKS.md`.
- **Upgrade Log**: Update `UPGRADE.md` with a technical summary of changes, Odoo SHAs/PRs found, and functional testing scenarios.

### Step 5: Final Validation (Clean Install & Tests) - MANDATORY
A migration is NEVER considered finished by testing only on the current working database. You MUST prove that the module installs "out of the box" on a fresh environment:
1. **Create a fresh DB**: `odev create -f -V [target_ver] <test_db> -i base`.
2. **Install & Test**: Run `odev test -V [target_ver] <test_db> -i <module_name>`.
3. **Handle Failures**: If this command fails, analyze the logs, fix the code, and **RESTART** the verification on a NEW clean database.

### Step 5a: QWeb Report & Mail Template Render Gate - MANDATORY for any module shipping or extending reports / mail templates

**WHY**: install, `--stop-after-init` and `odev test` never render a report or a mail body. Field and method references inside QWeb (`t-field`/`t-out`/`t-esc`/`t-foreach`) and inside `mail.template` bodies resolve **only at render time**, so a reference to something removed in the target version installs clean, tests green, and then raises `AttributeError` the first time a user prints or emails it in production. Step 5 does not cover this. Run against the already-populated upgrade `<db>` — no fresh database needed.

**1. DISCOVERY** — enumerate every render-time artifact the module owns or extends:
- Report templates: `grep -rnE '<template[^>]+id=' <module> --include="*.xml"`.
- Inherited standard reports: `grep -rnE 'inherit_id="(sale|account|stock|purchase|mrp|hr_expense|point_of_sale|l10n_)' <module> --include="*.xml"`.
- Report actions: `grep -rnE '<record[^>]+model="ir.actions.report"' <module> --include="*.xml"`.
- Mail templates: `grep -rnE '<record[^>]+model="mail.template"' <module> --include="*.xml"` (audit `body_html` **and** `subject`).

**2. STATIC FIELD-AUDIT against the TARGET source** — extract every field path used in those expressions (plus `o.`/`doc.`/`line.`/`object.` chains) and prove each still exists in the target source. `<target_odoo_path>` is a **container** holding one checkout per repository, so search the checkout, never the container:
- community addon: `grep -rn "<field>" <target_odoo_path>/odoo/addons/<app>/models/`
- core models (`res.partner`, `ir.*`): `grep -rn "<field>" <target_odoo_path>/odoo/odoo/addons/base/models/`
- enterprise: `grep -rn "<field>" <target_odoo_path>/enterprise/<module>/models/` (enterprise has no `addons/` level)

No hit ⇒ the field is gone. **Adapt, never re-add a removed field.**

**3. RENDER GATE** (no wkhtmltopdf needed, transaction rolled back). `ir.actions.report` and `mail.template` have no `module` column, so scope through `ir.model.data`. Pipe the script into the shell — `odoo-bin shell` execs stdin when it is not a TTY, which works on every version (`--shell-file` exists only from 19.0):

```bash
odev shell -V [target_ver] <db> --log-level=warn < /tmp/render_gate.py
```

```python
imd, failures, rendered = env['ir.model.data'], [], 0

# ir_ui_view.arch_db is a per-language jsonb column, so a stale translated arch
# renders wrong while en_US looks correct. Standard customer documents force the
# partner's language INSIDE the template (sale: t-lang="doc.partner_id.lang"), so
# the caller's context governs only the layout and any template that does not
# force a lang. Rendering under each installed language exercises those; to reach
# a forced one, render against a record whose partner has a non-default language.
langs = env['res.lang'].search([]).mapped('code') or ['en_US']

rids = imd.search([('module', '=', '<module>'), ('model', '=', 'ir.actions.report')]).mapped('res_id')
for report in env['ir.actions.report'].browse(rids).exists():
    # report_type is stored HYPHENATED: 'qweb-html' / 'qweb-pdf' / 'qweb-text'.
    if (report.report_type or '').replace('-', '_') not in ('qweb_html', 'qweb_pdf'):
        continue
    try:
        recs = env[report.model].search([], limit=1)  # model itself may be gone in the target
    except KeyError as e:
        failures.append(('report:%s' % report.report_name, 'model missing: %r' % e)); continue
    if not recs:
        print('SKIP report %s: no %s record' % (report.report_name, report.model)); continue
    for lang in langs:
        # A savepoint per render: a dead COLUMN raises a SQL error that aborts the
        # transaction, and every later render would then fail with "transaction is
        # aborted" - turning one real failure into a uselessly noisy report.
        try:
            with env.cr.savepoint():
                # signature is (report_ref, docids, data=None); report_name is a valid report_ref
                env['ir.actions.report'].with_context(lang=lang)._render_qweb_html(report.report_name, recs.ids)
            rendered += 1
        except Exception as e:  # AttributeError / KeyError => dead reference in the template
            failures.append(('report:%s[%s]' % (report.report_name, lang), repr(e)))

tids = imd.search([('module', '=', '<module>'), ('model', '=', 'mail.template')]).mapped('res_id')
for tmpl in env['mail.template'].browse(tids).exists():
    try:
        recs = env[tmpl.model].search([], limit=1) if tmpl.model else None
    except KeyError as e:
        failures.append(('mail:%s' % tmpl.name, 'model missing: %r' % e)); continue
    if not recs:
        print('SKIP mail.template %s' % tmpl.name); continue
    try:
        with env.cr.savepoint():
            tmpl._render_field('body_html', recs.ids)
        rendered += 1
    except Exception as e:
        failures.append(('mail:%s' % tmpl.name, repr(e)))

env.cr.rollback()  # leave NO trace in <db>
if failures:
    raise SystemExit('RENDER GATE FAILED: %s' % failures)
print('RENDER GATE PASSED (%s renders, langs: %s)' % (rendered, ','.join(langs)))
```

- **A gate that tested nothing has not passed.** Always print the count. If the module **owns** an `ir.actions.report` or a `mail.template` and `rendered` is 0, the `ir.model.data` scoping is wrong — investigate instead of accepting the pass. A module that only *inherits* standard report templates legitimately owns nothing: `ir.model.data` scoping finds only owned records, so for those resolve the inherited `ir.ui.view` to the standard `ir.actions.report` that renders it and gate that report instead. Never let "0 rendered" stand as a pass for a module that ships report XML.
- Distinguish a genuine dead-reference `AttributeError` (real bug → adapt the template) from a record-specific or missing-context error (render against a fuller record).
- **Commit** one atomic fix per discrete problem, with the originating Odoo SHA on the `Source:` line, and re-run until the gate prints PASSED.

### Step 6: Knowledge Base Synchronization (Final Phase)
Once all modules from the list have passed Step 5 **and Step 5a**:
1. **Review History**: Run `git log --grep="\[UPG\]" --reverse` to review all changes and their sources across all upgraded modules.
2. **Update Centralized Knowledge**: For each unique breaking change found in the history:
   - Add/Update the entry in the knowledge table at `[k_path]/<module_name>/[from_ver]_to_[target_ver].md`.
   - Format: `| Title | Explanation | Odoo Commit Hash |`.
3. **Consistency**: Ensure the information in the centralized KB matches your final `UPGRADE.md` findings.

---

## 📝 COMMIT GUIDELINES RECAP
- One commit per discrete fix. **NO MASSIVE COMMITS**.
- **Format**: `[UPG][[task_id]] module_name: Concise description`
- **Body**: Detailed description + **MANDATORY** Source (Odoo Commit SHA made clickable, PR, or Core path).
- **The `Source:` SHA must be one you actually resolved**, never one you recall:
  `git -C <target_odoo_path>/odoo cat-file -e <sha>^{commit}` (or `<target_odoo_path>/enterprise`). The
  version worktree directory is a container, not a repository — `git -C` on it fails with "not a git
  repository". Quote the full 40-character
  hash. If you cannot find the originating commit, write `Source: not identified` and name the core
  file or symbol you compared against instead. **Never write a SHA you have not resolved**: an
  unverifiable citation is worse than none, because the reviewer cannot tell a correct change from a
  guess and will redo the work. `odev upgrade` rejects unresolvable citations in a post-flight gate.
- **No unverified claims**: write "verified" / "tested" / "confirmed" only for a check you actually
  ran, and name it. A manifest bump is a manifest bump.
- **Ruff**: Run `ruff check <file> --select E999,F821,F822,F405` after Python edits. DO NOT fix unrelated style issues.
