# Odoo Coding Guidelines Skill

Use this skill when the user asks to **review, check, or validate Odoo code** against official coding standards.
Source: https://www.odoo.com/documentation/19.0/contributing/development/coding_guidelines.html

---

## General Principles

### Language
- **All code must be in English**: variable names, field names, method names, model names, comments, commit messages, XML IDs, and string literals used as technical identifiers.

### Comments
- Comments must explain **WHY**, never WHAT. The code itself should be readable enough to convey what it does.
- If a comment is needed to explain what a block of code does, **refactor** the code until it is self-explanatory, then remove the comment.
- Acceptable comments: hidden constraints, non-obvious invariants, workarounds for specific bugs, behavior that would surprise a reader.

### Naming
- **Never truncate names** to save keystrokes. Autocomplete makes long names free — short names cost readability.
  ```python
  # ❌
  def comp_tot_amt(self): ...
  inv_dt = fields.Date(...)

  # ✅
  def _compute_total_amount(self): ...
  invoice_date = fields.Date(...)
  ```
- Prefer clarity over brevity at all times.

---

## Module Structure

### Required Directories
```
my_module/
├── models/          # Model definitions (.py files)
├── views/           # Backend views and templates (_views.xml, _templates.xml)
├── data/            # Demo/data XML (_demo.xml, _data.xml)
├── security/        # ir.model.access.csv, <module>_groups.xml, <model>_security.xml
├── controllers/     # HTTP routes (<module_name>.py, NOT main.py)
├── static/          # Web assets: css/, js/, img/, lib/, scss/, xml/
├── wizard/          # TransientModel classes and their views
├── report/          # SQL view models + QWeb report templates
└── tests/           # Python tests
```

### File Naming Rules
- File names: **only `[a-z0-9_]`** (lowercase alphanumeric + underscore)
- File permissions: folder `755`, file `644`

**Models** — one file per main model, named after it:
```
models/plant_nursery.py      # plant.nursery model
models/plant_order.py        # plant.order model
models/res_partner.py        # inherited res.partner (one file per inherited model)
```

**Views** — suffix `_views.xml`, split by model:
```
views/plant_nursery_views.xml
views/plant_order_views.xml
views/<module>_menus.xml     # optional: top-level menus not tied to a specific action
views/<model>_templates.xml  # QWeb portal/website templates
```

**Security**:
```
security/ir.model.access.csv
security/<module>_groups.xml
security/<model>_security.xml
```

**Data**:
```
data/<model>_data.xml
data/<model>_demo.xml
```

**Wizards** (TransientModel):
```
wizard/<transient>.py
wizard/<transient>_views.xml
```

**Reports**:
```
report/<model>_report.py        # SQL view / report model
report/<model>_report_views.xml # QWeb templates
```

**Controllers**: `controllers/<module_name>.py` (not `main.py`). Inherited controller from another module: `controllers/<inherited_module_name>.py`.

---

## XML Files

### Format Rules
- **Double quotes**: Use `"` for all XML attribute values (not single quotes `'`)
- **No XML declaration**: Do NOT add `<?xml version="1.0" encoding="UTF-8"?>` at the top of XML files — Odoo does not require it
- **No bare `<data>` wrapper**: `<data>` is only useful when it carries `noupdate="1"`. A plain `<data>` with no attributes adds nothing and must be removed:
  ```xml
  <!-- ❌ useless wrapper -->
  <odoo>
      <data>
          <record .../>
      </data>
  </odoo>

  <!-- ✅ correct -->
  <odoo>
      <record .../>
  </odoo>
  ```
- **`noupdate` placement**: If the entire file is noupdate, set it on `<odoo>` directly (`<odoo noupdate="1">`). Use `<data noupdate="1">` only when *part* of the file is noupdate and part is not:
  ```xml
  <!-- ✅ mixed: some records protected, some not -->
  <odoo>
      <data noupdate="1">
          <record .../>   <!-- won't be overwritten on upgrade -->
      </data>
      <record .../>       <!-- updated normally -->
  </odoo>
  ```
- Use `<record>` notation for declaring records
- Attribute order in `<record>`: `id` before `model`
- Attribute order in `<field>`: `name` first, then value or `eval`, then others by importance
- Prefer `<menuitem>` shortcut over `<record model="ir.ui.menu">`
- Prefer `<template>` shortcut for QWeb views

### XML ID Naming Conventions

| Element | Pattern | Example |
|---|---|---|
| Menu | `<model>_menu` | `plant_nursery_menu` |
| Submenu | `<model>_menu_<action>` | `plant_nursery_menu_reports` |
| View | `<model>_view_<type>` | `plant_nursery_view_form`, `plant_order_view_list` |
| Main action | `<model>_action` | `plant_nursery_action` |
| Named action | `<model>_action_<detail>` | `plant_nursery_action_confirm` |
| Window action | `<model>_action_view_<type>` | `plant_nursery_action_view_kanban` |
| Group | `<module>_group_<name>` | `plant_nursery_group_user`, `plant_nursery_group_manager` |
| Rule | `<model>_rule_<group>` | `plant_nursery_rule_user`, `plant_nursery_rule_company` |

- **`name` attribute** of a record = XML ID with dots replacing underscores
- Actions must have real, human-readable `name` values (used as display name)

### Inheritance
- Inheriting views reuse the **same `id`** as the original (module prefix prevents collision)
- The `name` field must be: `<original_view_name>.inherit.<module.name>`, where `<original_view_name>` is the **actual `name` field value of the original view** (look it up — do not derive it), and `<module.name>` is the inheriting module's technical name in dot notation. Example: if the original view has `name="stock.picking.form"` and the module is `my_module`, use `stock.picking.form.inherit.my.module`.
- New primary views (not pure overrides) do NOT need `.inherit` suffix

---

## Python

### PEP8 — Allowed Exceptions
- `E501`: line too long (acceptable)
- `E301`: expected 1 blank line, found 0
- `E302`: expected 2 blank lines, found 1

### Import Order
```python
# 1. Python stdlib (alphabetically sorted)
import logging
import re

# 2. Odoo core
from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError

# 3. Odoo addons (rare, only if necessary)
from odoo.addons.web.controllers.main import Home
```

### Odoo-Specific Rules

**Context**: Never modify context dict directly. Use `with_context()`:
```python
# ✅
records.with_context(mail_notrack=True).write(vals)
# ❌
self.env.context['key'] = value  # context is frozendict
```

**Transactions**: **Never call `cr.commit()` or `cr.rollback()`** unless you explicitly opened the cursor yourself. The framework handles transactions per RPC call.

**Exceptions**: Catch only specific exceptions; limit try/except scope. Avoid broad `except Exception`.

**Translation**: Only use `_()` on static literal strings:
```python
# ✅
raise UserError(self.env._('Invalid state for this operation'))
# ❌
raise UserError(self.env._(variable_string))
```
Use `%` formatting over `.format()` for translated strings; use `%(varname)s` for multiple variables.

**Think extendable**: Prefer many small methods (single responsibility) over large complex ones. Avoid hardcoding business logic.

**Magic numbers**: Never use bare numeric or string literals where a named constant communicates intent. Define them as module-level or class-level constants using `Command` (for O2M/M2M operations) or plain named variables:
```python
# ❌
vals['order_line'] = [(4, line.id)]
vals['order_line'] = [(0, 0, {'product_id': product.id})]

# ✅
from odoo.fields import Command
vals['order_line'] = [Command.link(line.id)]
vals['order_line'] = [Command.create({'product_id': product.id})]
```

**Programming idioms**:
- Prefer readability over conciseness
- Do NOT use `.clone()`
- Use list/dict comprehensions and builtins (`map`, `filter`, `sum`, `any`, `all`)
- Use `if some_collection:` instead of `if len(some_collection):`
- Use `dict.setdefault()` appropriately
- Use `filtered()`, `mapped()`, `sorted()` on recordsets

---

## Testing

**`assertRaises` type annotation**: When using `self.assertRaises` as a context manager, mypy raises a false positive on the `with` block. Suppress it with `# type: ignore[attr-defined]`:
```python
# ✅
with self.assertRaises(UserError) as cm:  # type: ignore[attr-defined]
    record.action_confirm()
```

---

## Symbols and Naming Conventions

### Model Names (dot notation, singular)
```python
# ✅
class PlantNursery(models.Model):
    _name = 'plant.nursery'         # singular

class AccountInvoiceMake(models.TransientModel):
    _name = 'account.invoice.make'  # wizard: <base_model>.<action>

class SaleReport(models.Model):
    _name = 'sale.report'           # SQL view: <base_model>.report.<action>
```

### Python Class Names
- Models: **PascalCase** — `PlantNursery`, `SaleOrderLine`

### Variable Names
- Model variables: **PascalCase** — `PlantNursery = self.env['plant.nursery']`
- Common variables: **snake_case** — `order_lines`, `total_amount`
- Variables holding a record ID: suffix `_id` — `partner_id` (ID int, NOT a recordset)
- Variables holding list of IDs: suffix `_ids`

### Field Naming
| Field type | Suffix | Example |
|---|---|---|
| Many2one | `_id` | `partner_id`, `user_id`, `company_id` |
| One2many | `_ids` | `order_line_ids`, `tag_ids` |
| Many2many | `_ids` | `category_ids`, `attribute_value_ids` |

### Field `string` Attribute

Odoo auto-generates the field label from the field name:
- `_id` suffix is stripped, `_ids` suffix is stripped, underscores become spaces, result is title-cased.
- `partner_id` → `"Partner"`, `order_line_ids` → `"Order Line"`, `invoice_date` → `"Invoice Date"`

**Do NOT add `string=` when it matches the auto-generated value** — it is pure noise:
```python
# ❌ redundant
partner_id = fields.Many2one('res.partner', string='Partner')
invoice_date = fields.Date(string='Invoice Date')
order_line_ids = fields.One2many('sale.order.line', 'order_id', string='Order Line')

# ✅ correct
partner_id = fields.Many2one('res.partner')
invoice_date = fields.Date()
order_line_ids = fields.One2many('sale.order.line', 'order_id')
```

Only add `string=` when the desired label genuinely differs from the auto-generated one:
```python
# ✅ justified — auto would be "Currency" but business label is "Company Currency"
currency_id = fields.Many2one('res.currency', string='Company Currency')
```

**Prefer Python `string=` over XML `string=`**: if a label override is needed, set it once on the field definition in Python — not in every view that shows the field. XML `string=` on a `<field>` tag should be extremely rare (only when the same field needs a *different* label in a specific view context, and only as a last resort):
```xml
<!-- ❌ wrong: override belongs in the field definition, not the view -->
<field name="partner_id" string="Customer"/>

<!-- ✅ only acceptable when this view intentionally needs a different label than the model default -->
<field name="partner_id" string="Delivery Address"/>
```

### Method Naming Conventions
| Purpose | Pattern | Example |
|---|---|---|
| Compute | `_compute_<field>` | `_compute_total_amount` |
| Inverse | `_inverse_<field>` | `_inverse_name` |
| Search | `_search_<field>` | `_search_display_name` |
| Default | `_default_<field>` | `_default_currency_id` |
| Selection | `_selection_<field>` | `_selection_state` |
| Onchange | `_onchange_<field>` | `_onchange_partner_id` |
| Constraint | `_check_<constraint>` | `_check_date_range` |
| Action | `action_<name>` | `action_confirm`, `action_draft` |

- Action methods act on single records: add `self.ensure_one()` at the start.

### Model Class Attribute Order
```python
class MyModel(models.Model):
    # 1. Private attributes
    _name = 'my.model'
    _description = '...'
    _inherit = [...]
    _order = '...'
    _rec_name = '...'

    # 2. Default methods and default_get
    @api.model
    def default_get(self, fields_list): ...

    # 3. Field declarations
    name = fields.Char(...)
    partner_id = fields.Many2one(...)

    # 4. SQL constraints and indexes
    _sql_constraints = [...]

    # 5. Compute / inverse / search (same order as field declarations)
    @api.depends('...')
    def _compute_name(self): ...

    # 6. Selection methods (for computed selection values)
    def _selection_state(self): ...

    # 7. Constraints (@api.constrains) and onchange (@api.onchange)
    @api.constrains('date_start', 'date_end')
    def _check_date_range(self): ...

    @api.onchange('partner_id')
    def _onchange_partner_id(self): ...

    # 8. CRUD overrides
    @api.model_create_multi
    def create(self, vals_list): ...
    def write(self, vals): ...
    def unlink(self): ...

    # 9. Action methods
    def action_confirm(self): ...

    # 10. Other business methods
    def _get_applicable_taxes(self): ...
```

---

## JavaScript

### Static File Structure
```
static/
├── lib/             # Third-party JS libraries (e.g., static/lib/jquery/)
├── src/
│   ├── css/
│   ├── fonts/
│   ├── img/
│   ├── js/
│   │   └── tours/   # End-user onboarding tours (not tests)
│   ├── scss/
│   └── xml/         # QWeb templates rendered in JS
└── tests/
    └── tours/       # Tour test files (not tutorials)
```

### JS Rules
- Use `'use strict';` in all JS files
- Use a linter (jshint, eslint)
- **Never** add minified JS libraries
- Use **PascalCase** for class declarations
- **Do NOT add `/** @odoo-module **/`**: this directive is deprecated and must not appear in JS files

---

## CSS / SCSS

### Formatting
- 4-space indents, no tabs
- Max 80 columns wide
- Opening `{`: space after last selector
- Closing `}`: on its own line
- One declaration per line

### Properties Order
Order from "outside in": position → box model → typography → decorative (`font`, `filter`, etc.).
Scoped SCSS variables and CSS variables go at the **very top**, followed by an empty line.

### Class Naming
- Prefix all classes: `o_<module_name>_...` (e.g., `o_sale_order_form`, `o_forum_post`)
- Exception: webclient uses just `o_` prefix
- Avoid id selectors
- Use "Grandchild" approach — avoid hyper-specific nested class names

### SCSS Variables
Standard convention: `$o-[root]-[element]-[property]-[modifier]`
```scss
$o-sale-total-color: #000;
$o-sale-header-bg-color-active: $primary;
```

### Scoped SCSS Variables (within blocks)
Convention: `$-[variable-name]`
```scss
.o_my_component {
    $-spacing: 8px;
    padding: $-spacing;
}
```

### SCSS Mixins and Functions
Convention: `o-[name]`, use imperative verbs for functions (`o-get-color`, `o-make-shadow`).
Optional arguments use scoped variable form: `$-argument`.

### CSS Variables
Convention: BEM — `--[root]__[element]-[property]--[modifier]`
```css
--sale__header-color--active: var(--primary);
```
- CSS variables are DOM-context-scoped (not global design system)
- Defined inside component's main block with default fallbacks
- Avoid defining on `:root` pseudo-class (use SCSS variables for global design system instead)

---

---

## Checklist for Code Review

When reviewing Odoo code, check:

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
- [ ] Class attributes in correct order (private attrs → fields → compute → constraints → CRUD → actions → business)

**Testing**
- [ ] `self.assertRaises(...)` as context manager has `# type: ignore[attr-defined]` to suppress mypy false positive

**Naming**
- [ ] Model names are singular dot-notation
- [ ] Wizard: `<base_model>.<action>`, Report: `<base_model>.report.<action>`
- [ ] Many2one fields end in `_id`, O2M/M2M fields end in `_ids`
- [ ] Compute methods: `_compute_<field>`, onchange: `_onchange_<field>`, etc.
- [ ] Action methods start with `action_` and call `self.ensure_one()`
- [ ] Python classes use PascalCase
- [ ] No redundant `string=` on field definitions — only present when label differs from auto-generated (strip `_id`/`_ids`, replace `_` with space, title-case)
- [ ] No `string=` on XML `<field>` tags unless the view intentionally needs a label different from the Python field definition

**JavaScript**
- [ ] No `/** @odoo-module **/` directive (deprecated)

**CSS/SCSS**
- [ ] Classes prefixed with `o_<module_name>_`
- [ ] No id selectors
- [ ] SCSS variables follow `$o-[root]-[element]-[property]` pattern
- [ ] No `:root` CSS variable declarations (unless cross-bundle templates)
- [ ] Inline `style=""` replaced with Bootstrap utility classes where an equivalent exists (e.g. `text-end`, `fw-bold`, `mb-3`, `ms-3`); unavoidable layout values (e.g. `min-width`, `width: auto`) are acceptable inline

