# XML Guidelines

## Format Rules
- **Double quotes**: Use `"` for all XML attribute values (not single quotes `'`)
- **No XML declaration**: Do NOT add `<?xml version="1.0" encoding="UTF-8"?>` at the top of XML files
- **No bare `<data>` wrapper**: `<data>` is only useful when it carries `noupdate="1"`. A plain `<data>` with no attributes must be removed:
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

---

## XML ID Naming Conventions

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

---

## Inheritance

- Inheriting views reuse the **same `id`** as the original (module prefix prevents collision)
- The `name` field must be: `<original_view_name>.inherit.<module.name>`, where `<original_view_name>` is the **actual `name` field value of the original view** (look it up — do not derive it), and `<module.name>` is the inheriting module's technical name in dot notation. Example: if the original view has `name="stock.picking.form"` and the module is `my_module`, use `stock.picking.form.inherit.my.module`.
- New primary views (not pure overrides) do NOT need `.inherit` suffix

---

## Portal Templates

- Reusable portal sub-templates are **standalone `<template>` tags** (no `inherit_id`), called via `t-call`.
- Portal templates do **not** use the `website=True` attribute — that is exclusive to the `website` module.
- **Modal dialogs**: keep the modal wrapper (`<div class="modal fade">`, `<div class="modal-dialog">`) in the **parent** template. Extract only the `<div class="modal-content">` and its children into the sub-template:
  ```xml
  <!-- ✅ Parent template keeps the wrapper -->
  <div role="dialog" class="modal fade" id="my_modal" tabindex="-1">
      <div class="modal-dialog">
          <t t-call="my_module.my_modal_content"/>
      </div>
  </div>

  <!-- ✅ Sub-template has only modal-content -->
  <template id="my_modal_content" name="My Modal Content">
      <div class="modal-content">
          <div class="modal-header">...</div>
          <div class="modal-body">...</div>
          <div class="modal-footer">...</div>
      </div>
  </template>
  ```

---

## QWeb Naming

**`t-foreach` variables** must reflect the actual record type — same rule as Python loop variables:

```xml
<!-- ❌ too generic -->
<t t-foreach="records" t-as="record">
    <span t-out="record.name"/>
</t>

<!-- ✅ name reflects the model -->
<t t-foreach="orders" t-as="order">
    <span t-out="order.name"/>
</t>
```

**`string=` on `<field>` tags**: avoid. If a label override is needed, set it on the Python field definition. Only use XML `string=` when this specific view intentionally needs a label different from the field definition:

```xml
<!-- ❌ override belongs in the field definition -->
<field name="partner_id" string="Customer"/>

<!-- ✅ only when this view genuinely needs a different label -->
<field name="partner_id" string="Delivery Address"/>
```
