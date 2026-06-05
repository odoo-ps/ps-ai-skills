# Odoo Code Review Checklist

**General**
- [ ] All identifiers, comments, and string literals are in English
- [ ] Comments explain WHY, not WHAT — no comments describing what the code does
- [ ] Code is readable without comments; blocks that need explanation are refactored instead
- [ ] No truncated variable or method names

**Structure**
- [ ] Module directories follow the convention
- [ ] File names use only `[a-z0-9_]`
- [ ] Files are in the correct directories

**XML**
- [ ] No `<?xml version="1.0" encoding="UTF-8"?>` declaration at the top
- [ ] No bare `<data>` wrapper without `noupdate="1"`
- [ ] `noupdate` on `<odoo>` tag when entire file is noupdate; `<data noupdate="1">` only for mixed files
- [ ] XML IDs follow `<model>_view_<type>`, `<model>_action`, etc. patterns
- [ ] `<record>` attribute order: `id` before `model`, `name` first in `<field>`
- [ ] Inherited view IDs match original + name has `.inherit.<module.name>` suffix (module technical name in dot notation)
- [ ] `<menuitem>` / `<template>` shortcuts used where appropriate

**Python**
- [ ] Imports in correct order (stdlib → odoo → addons)
- [ ] No `cr.commit()` / `cr.rollback()` without explicit cursor
- [ ] `_()` used only on literal strings
- [ ] Context modified via `with_context()`, not direct dict mutation
- [ ] Exception handling is specific (no bare `except:`)
- [ ] No magic numbers — O2M/M2M commands use `Command.link`, `Command.create`, etc. instead of bare tuples like `(4, id)` or `(0, 0, vals)`
- [ ] Computed fields that need search use `_search_<field>` instead of `store=True`, unless the field is needed for ordering, group-by, range queries at scale, or cannot be expressed as a domain
- [ ] Class attributes in correct order (private attrs → fields → compute → constraints → CRUD → actions → business)

**Testing**
- [ ] `self.assertRaises(...)` as context manager has `# type: ignore[attr-defined]` to suppress mypy false positive

**Type Checker**
- [ ] Known Pyright/mypy false positives (Odoo route kwargs, env subscript, dynamic MRO, domain types) are silenced with specific `# type: ignore[<code>]` — never bare `# type: ignore`
- [ ] Dicts that will hold recordsets are initialized with `Model.browse()` (empty recordset) instead of `None`

**Naming**
- [ ] Model names are singular dot-notation
- [ ] Wizard: `<base_model>.<action>`, Report: `<base_model>.report.<action>`
- [ ] Many2one fields end in `_id`, O2M/M2M fields end in `_ids`
- [ ] Compute methods: `_compute_<field>`, onchange: `_onchange_<field>`, etc.
- [ ] Action methods start with `action_` and call `self.ensure_one()`
- [ ] Python classes use PascalCase
- [ ] No redundant `string=` on field definitions — only present when label differs from auto-generated
- [ ] No `string=` on XML `<field>` tags unless the view intentionally needs a label different from the Python field definition
- [ ] Loop variables named after the model (`order`/`orders`, `line`/`lines`), not generics (`record`/`records`) — unless the method is truly model-agnostic
- [ ] QWeb `t-foreach` variables follow the same rule — no `record`/`item` when the model is known
- [ ] No ambiguous acronyms — full descriptive names preferred; exception: single-letter variable in a single-expression inline lambda

**JavaScript**
- [ ] No `/** @odoo-module **/` directive (deprecated)

**CSS/SCSS**
- [ ] Classes prefixed with `o_<module_name>_`
- [ ] No id selectors
- [ ] SCSS variables follow `$o-[root]-[element]-[property]` pattern
- [ ] No `:root` CSS variable declarations (unless cross-bundle templates)
- [ ] Inline `style=""` replaced with Bootstrap utility classes where an equivalent exists; unavoidable layout values are acceptable inline
