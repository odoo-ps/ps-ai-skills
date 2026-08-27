# Python Guidelines

## PEP8 — Allowed Exceptions
- `E501`: line too long (acceptable)
- `E301`: expected 1 blank line, found 0
- `E302`: expected 2 blank lines, found 1

## Import Order
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

## Odoo-Specific Rules

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

**Magic numbers**: Never use bare numeric or string literals where a named constant communicates intent. Use `Command` for O2M/M2M operations:
```python
# ❌
vals['order_line'] = [(4, line.id)]
vals['order_line'] = [(0, 0, {'product_id': product.id})]

# ✅
from odoo.fields import Command
vals['order_line'] = [Command.link(line.id)]
vals['order_line'] = [Command.create({'product_id': product.id})]
```

**Computed fields — `_search` vs `store=True`**: For a computed field that needs to be searchable, implement `_search_<field>` instead of adding `store=True`. `_search` returns a domain over stored fields, avoiding a DB column, recomputation triggers, and potential staleness. Only use `store=True` when:
- field is used in `_order` (non-stored fields cannot be sorted at DB level)
- field is used in group-by (pivot/graph views require a real column)
- field needs a DB index for range queries (`>=`, `<=`) at scale
- the value cannot be derived from other stored fields (e.g. depends on `datetime.now()` or external state), making a correct `_search` domain impossible

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

## Pyright / Type Checker False Positives

Odoo's dynamic nature (dynamic model registry, `__getattr__` on recordsets, decorator kwargs, etc.) causes many Pyright/mypy false positives. **Do NOT refactor working Odoo code to satisfy the type checker.** Instead, silence known false positives with targeted `# type: ignore[<code>]` comments.

### Common False Positives to Ignore

| Error code | Typical trigger | Suppression |
|---|---|---|
| `call-arg` | `website=True`, `readonly=True`, or other Odoo-specific kwargs on `@http.route(...)` | `# type: ignore[call-arg]` on the decorator line |
| `index` | `request.env[model_name]` when Pyright types `request.env` as optional or non-subscriptable | `# type: ignore[index]` |
| `attr-defined` | `super()._prepare_*()` or any method resolved via Odoo's dynamic MRO / `_inherit` chain | `# type: ignore[attr-defined]` |
| `arg-type` | Odoo domain lists passed to `search()` / `search_count()`; `bytes` to `make_response()` | `# type: ignore[arg-type]` |

### Rules
- Always use the **specific error code** in the ignore comment (`# type: ignore[call-arg]`), never bare `# type: ignore`.
- For multi-line decorators, place the comment on the **opening line** (`@http.route(  # type: ignore[call-arg]`).
- Prefer a real code fix over `# type: ignore` when one exists. For example, initialize a dict that will hold recordsets with empty recordsets (`Model.browse()`) instead of `None` to avoid `reportOptionalSubscript` / `reportAttributeAccessIssue`.

---

## Naming Conventions

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

### Loop / Iteration Variable Names

Use **model-specific names** when iterating over a concrete recordset. Generic names (`record`, `records`, `item`, `obj`) are reserved for mixins, abstract models, or truly generic helpers where the model is unknown.

```python
# ❌ too generic — the model is known
for record in records:
    record.action_confirm()

# ✅ name reflects the actual model
for order in orders:
    order.action_confirm()
```

Same rule applies to QWeb `t-foreach` — see xml.md.

### Indicative Names over Acronyms

Prefer full, descriptive names over ambiguous abbreviations.

```python
# ❌ unclear acronyms
def comp_inv_amt(self): ...
uom_id = ...
pricelist_upd = ...

# ✅ self-explanatory
def _compute_invoice_amount(self): ...
unit_of_measure_id = ...
updated_pricelist = ...
```

**Accepted exceptions** — short names are fine when scope makes meaning obvious:

- Inline single-expression lambdas: a single letter matching the first letter of the model is acceptable.
  ```python
  # ✅ — 'l' for line, scope is one expression
  lines.filtered(lambda l: l.product_id)
  lines.sorted(key=lambda l: l.sequence)

  # ❌ — multi-line or named function: use the full name
  def filter_lines(line):
      return line.product_id and line.price_unit > 0
  ```

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

**Do NOT add `string=` when it matches the auto-generated value**:
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

**Prefer Python `string=` over XML `string=`**: set it once on the field definition — not in every view. XML `string=` on a `<field>` tag is only acceptable when a specific view genuinely needs a different label (see xml.md).

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
    # 1. Private attributes (_name, _description, _inherit, _order, _rec_name)
    # 2. default_get / default_<field> methods
    # 3. Field declarations
    # 4. _sql_constraints
    # 5. Compute / inverse / search (same order as fields)
    # 6. Selection methods
    # 7. @api.constrains and @api.onchange
    # 8. CRUD overrides (create, write, unlink)
    # 9. Action methods (action_*)
    # 10. Other business methods
```
