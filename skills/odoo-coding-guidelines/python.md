# Python: what the official guidelines do not cover

## Computed fields

**`_search` over `store=True`.** For a computed field that needs to be searchable, implement `_search_<field>`. It
returns a domain over stored fields, avoiding a column, recomputation triggers and staleness. Use `store=True` only
when:

- the field is used in `_order`
- the field is used in group-by (pivot and graph views need a real column)
- the field needs an index for range queries at scale
- the value cannot be derived from other stored fields, so no correct `_search` domain exists

**`compute_sudo` over `sudo()`.** A compute that must read records the current user may not access declares
`compute_sudo=True` on the non-stored field instead of calling `sudo()` inside the method:

```python
# bad
task_count = fields.Integer(compute="_compute_task_count")

def _compute_task_count(self):
    for batch in self:
        batch.task_count = len(batch.sudo().task_ids)

# good
task_count = fields.Integer(compute="_compute_task_count", compute_sudo=True)

def _compute_task_count(self):
    for batch in self:
        batch.task_count = len(batch.task_ids)
```

This does not make any button or action on the field accessible; those still need their own `groups=`.

## Field `string`

Odoo generates the label from the field name: `_id` and `_ids` are stripped, underscores become spaces, and the result
is title-cased. Do not add `string=` when it matches:

```python
# bad
partner_id = fields.Many2one('res.partner', string='Partner')

# good
partner_id = fields.Many2one('res.partner')

# good: the label differs from the generated one
currency_id = fields.Many2one('res.currency', string='Company Currency')
```

Set a label once on the Python field, not in every view. See [xml.md](xml.md).

## Names

Name loop variables after the model. `record`, `records`, `item` and `obj` are for mixins, abstract models and helpers
where the model is unknown:

```python
# bad
for record in records:
    record.action_confirm()

# good
for order in orders:
    order.action_confirm()
```

Prefer full names over ambiguous abbreviations (`_compute_invoice_amount`, not `comp_inv_amt`). The exception is an
inline single-expression lambda, where a single letter matching the model is fine: `lines.filtered(lambda l: l.product_id)`.

## `Command`

Use `Command` for x2many operations, not magic tuples:

```python
# bad
vals['order_line'] = [(4, line.id)]

# good
vals['order_line'] = [Command.link(line.id)]
```

## Type checker false positives

Odoo's dynamic registry and recordsets cause Pyright and mypy false positives. Do not refactor working code to satisfy
the checker. Silence the known cases with the specific code, never a bare `# type: ignore`:

| Code | Typical trigger |
|---|---|
| `call-arg` | `website=True`, `readonly=True` or other Odoo kwargs on `@http.route(...)`; on a multi-line decorator, put the comment on the opening line |
| `index` | `request.env[model_name]` |
| `attr-defined` | `super()._prepare_*()` and other methods resolved through `_inherit`; `with self.assertRaises(...)` |
| `arg-type` | Domain lists passed to `search()` or `search_count()`; `bytes` passed to `make_response()` |

Prefer a real fix when one exists: a dict that will hold recordsets starts with empty recordsets (`Model.browse()`),
not `None`.
