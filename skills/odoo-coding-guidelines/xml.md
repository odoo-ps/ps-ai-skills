# XML: what the official guidelines do not cover

## File format

- Double quotes for attribute values.
- No `<?xml version="1.0" encoding="UTF-8"?>` declaration.
- No bare `<data>`: it is only useful with `noupdate="1"`, for a file where only part of the records are noupdate.

## Inheriting view name

The `name` of an inheriting view is `<original_view_name>.inherit.<module.name>`. `<original_view_name>` is the actual
`name` value of the original view, so look it up rather than deriving it. `<module.name>` is the technical name of the
inheriting module in dot notation. For `stock.picking.form` extended from `my_module`:
`stock.picking.form.inherit.my.module`.

## Labels

Avoid `string=` on a `<field>`: set the label on the Python field. Use it only when this view needs a label different
from the field definition.

```xml
<!-- bad: belongs on the field definition -->
<field name="partner_id" string="Customer"/>

<!-- good: this view genuinely means something else -->
<field name="partner_id" string="Delivery Address"/>
```

## QWeb

Name `t-foreach` variables after the record type, like Python loop variables:

```xml
<!-- bad -->
<t t-foreach="records" t-as="record">

<!-- good -->
<t t-foreach="orders" t-as="order">
```

## Portal templates

- Reusable portal sub-templates are standalone `<template>` tags, without `inherit_id`, called with `t-call`.
- Portal templates do not use `website=True`, which belongs to the `website` module.
- For a modal, keep the wrapper in the parent template and extract only the content:

```xml
<!-- parent -->
<div role="dialog" class="modal fade" id="my_modal" tabindex="-1">
    <div class="modal-dialog">
        <t t-call="my_module.my_modal_content"/>
    </div>
</div>

<!-- sub-template -->
<template id="my_modal_content" name="My Modal Content">
    <div class="modal-content">
        <div class="modal-header">...</div>
        <div class="modal-body">...</div>
        <div class="modal-footer">...</div>
    </div>
</template>
```
