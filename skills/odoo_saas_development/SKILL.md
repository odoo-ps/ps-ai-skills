# Odoo Online (SaaS) / Importable Modules Skill

Use this skill when developing **importable (data) modules** — custom Odoo modules that contain no custom Python
business logic and can be deployed on managed/SaaS instances like Odoo.com.

## Key Constraint

SaaS instances do **not** allow arbitrary Python code. Everything that would normally be a Python class, method, or
field definition must instead be expressed as **XML data records**. JavaScript files are allowed.

## Module Structure

```
my_module
├── actions/      *.xml  — server actions, window actions
├── models/       *.xml  — ir.model + ir.model.fields records
├── security/     ir.model.access.csv, *_security.xml
├── views/        *.xml  — UI views
├── static/src/js *.js   — JavaScript (OWL components, tours, etc.)
├── __init__.py          — empty, required for classic addons-path installs
└── __manifest__.py      — standard manifest; models listed in `data`
```

`__init__.py` is empty. `data` files must be listed **in dependency order** (models before views).

## Deploying

**Via UI**: zip the module → Apps → Import Module (requires developer mode + `base_import_module` installed).

Upload options:
- **Force init**: re-runs `noupdate="1"` records
- **Import demo data**: imports demo records

**Via CLI**:
```bash
odoo-bin deploy <path_to_module> https://<instance> --login <user> --password <pass> [--force]
```

The deploying user needs **Administration/Settings** rights.

> Changing a field's type after first deploy is not supported — easiest fix is a fresh DB or uninstall first.

## Naming Rules

- All **custom models** must be prefixed: `x_my.model`
- All **custom fields** must be prefixed: `x_my_field`

## Defining Models

Models are `ir.model` records:

```xml
<record id="model_estate_property" model="ir.model">
    <field name="name">Real Estate Property</field>
    <field name="model">x_estate.property</field>
</record>
```

Odoo auto-adds: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`.

## Defining Fields

Fields are `ir.model.fields` records:

```xml
<record id="field_estate_property_name" model="ir.model.fields">
    <field name="model_id" ref="my_module.model_estate_property"/>
    <field name="name">x_name</field>
    <field name="field_description">Name</field>
    <field name="ttype">char</field>
    <field name="required">True</field>
</record>
```

Common `ttype` values: `char`, `integer`, `float`, `boolean`, `date`, `datetime`, `html`, `text`,
`many2one`, `many2many`, `one2many`, `selection`, `binary`.

Key optional attributes: `required`, `readonly`, `index`, `copied`, `translate`, `help`.

## Default Values

Use `ir.default` records (static only — no dynamic defaults like "today"):

```xml
<record id="default_estate_selling_price" model="ir.default">
    <field name="field_id" ref="my_module.field_estate_property_selling_price"/>
    <field name="json_value">100000</field>
</record>
```

Optionally scoped with `user_id` or `company_id`.

## Relational Fields

### Many2one
```xml
<record id="field_estate_property_partner_id" model="ir.model.fields">
    <field name="model_id" ref="my_module.model_estate_property"/>
    <field name="name">x_partner_id</field>
    <field name="field_description">Customer</field>
    <field name="ttype">many2one</field>
    <field name="relation">res.partner</field>
    <!-- optional: ondelete, domain -->
</record>
```

### Many2many
Same as many2one but `ttype` = `many2many`. Optional: `relation_table`, `column1`, `column2`
(only needed when multiple m2m exist between the same two models).

### One2many
Same as many2one but `ttype` = `one2many`. Requires `relation_field` (the m2o field on the child model).

## Computed Fields

Stored by default. Use `depends` + `compute` with a CDATA block. Sandbox provides `datetime`, `dateutil`, `time`
plus Python builtins. **No imports**, no OS access, no `print`. Use dict assignment, not dot assignment:

```xml
<record id="field_estate_property_total_area" model="ir.model.fields">
    <field name="model_id" ref="my_module.model_estate_property"/>
    <field name="name">x_total_area</field>
    <field name="field_description">Total Area</field>
    <field name="ttype">float</field>
    <field name="depends">x_living_area,x_garden_area</field>
    <field name="compute"><![CDATA[
for property in self:
    property['x_total_area'] = property.x_living_area + property.x_garden_area
]]></field>
</record>
```

Set `store="False"` to disable storage; `readonly="True"` to prevent editing.

## Related Fields

Mirror a value through a many2one chain using the `related` attribute:

```xml
<record id="field_estate_property_country_id" model="ir.model.fields">
    <field name="model_id" ref="my_module.model_estate_property"/>
    <field name="name">x_country_id</field>
    <field name="field_description">Buyer's Country</field>
    <field name="ttype">many2one</field>
    <field name="relation">res.country</field>
    <field name="related">x_partner_id.country_id</field>
</record>
```

## Server Actions (Business Logic)

Code-type server actions run in a sandbox. Available globals: `env`, `model`, `user`, `uid`, `record`,
`records`, `datetime`, `dateutil`, `timezone`, `time`, `float_compare`, `b64encode`, `b64decode`, `Command`.

To return a client action assign to `action`; for website assign to `response`.

```xml
<record id="action_estate_refuse_offers" model="ir.actions.server">
    <field name="name">Refuse all offers</field>
    <field name="model_id" ref="my_module.model_estate_property"/>
    <field name="state">code</field>
    <field name="code"><![CDATA[
for property in records:
    property.x_offer_ids.write({'x_status': 'refused'})
]]></field>
</record>
```

### Binding to a button in a view

```xml
<button name="my_module.action_estate_refuse_offers" type="action" string="Refuse All Offers"/>
```

### Binding to the gear icon (⚙)

Set `binding_model_id` and `binding_view_types` (`tree`, `form`, or both) on the server action record.

## Overriding Standard Modules

### Via UI replacement

Replace a button's `type="object"` call with a `type="action"` that calls your server action, which then
calls the original method before/after your custom logic:

```xml
<record id="view_sale_order_form_estate" model="ir.ui.view">
    <field name="inherit_id" ref="sale.view_order_form"/>
    <field name="model">sale.order</field>
    <field name="arch" type="xml">
        <xpath expr="//button[@name='action_confirm'][@type='object']" position="attributes">
            <attribute name="type">action</attribute>
            <attribute name="name">my_module.action_create_property_from_sale</attribute>
        </xpath>
    </field>
</record>
```

### Via Automation Rules (preferred for lifecycle events)

More robust than UI replacement — fires even when the event is triggered by portal, API, etc.
Requires `base_automation` in module dependencies.

```xml
<record id="automation_create_property_on_sale_confirm" model="base.automation">
    <field name="name">Create property on sale confirm</field>
    <field name="model_id" ref="sale.model_sale_order"/>
    <field name="trigger">on_state_set</field>
    <field name="trg_selection_field_id" ref="sale.selection__sale_order__state__sale"/>
    <field name="trigger_field_ids" eval="[(4, ref('sale.field_sale_order__state'))]"/>
    <field name="action_server_ids" eval="[(4, ref('my_module.action_create_property_from_sale'))]"/>
</record>
```

## Website Controllers

When `website` is installed, server actions can act as HTTP endpoints:

```xml
<record id="server_action_estate_list" model="ir.actions.server">
    <field name="name">Estate List Controller</field>
    <field name="model_id" ref="my_module.model_estate_property"/>
    <field name="website_published">True</field>
    <field name="website_path">estate</field>   <!-- full path: /website/action/estate -->
    <field name="state">code</field>
    <field name="code"><![CDATA[
properties = request.env['x_estate.property'].search([])
response = request.make_json_response([{'name': p.x_name} for p in properties])
]]></field>
</record>
```

Useful `request` methods: `get_http_params()`, `get_json_data()`, `render(xmlid, qcontext)`,
`redirect(url)`, `not_found()`, `make_response(data)`, `make_json_response(data)`.

> The server action's `model_id` must be readable by the public user, or the action returns 403.
> Link to `ir.filters` as a workaround if needed.

## JavaScript / OWL Components

JS files are fully supported. No glob patterns — list every file explicitly in the manifest:

```python
"assets": {
    "web.assets_backend": [
        "my_module/static/src/js/my_component.js",
    ],
},
```

### Onboarding Tours

`static/src/js/tour.js`:
```javascript
import { registry } from "@web/core/registry";
registry.category("web_tour.tours").add('my_tour', {
  url: "/web",
  steps: () => [{
    trigger: '.o_app[data-menu-xmlid="my_module.menu_root"]',
    content: 'Welcome to the app!',
  }],
});
```

Register the tour as an XML record:
```xml
<record id="my_tour" model="web_tour.tour">
    <field name="name">my_tour</field>
    <field name="sequence">2</field>
    <field name="rainbow_man_message">You made it!</field>
</record>
```

## Known Limitations

| Feature | Status |
|---|---|
| Custom Python classes/methods | Not allowed |
| Importing Python libraries | Not allowed |
| Dynamic default values (e.g., "today") | Not allowed |
| Changing a field type after deploy | Not supported |
| Glob patterns in assets | Not supported |
| Arbitrary OS/file access in sandbox | Not allowed |
| `print()` in sandbox | Not allowed |
| Dot assignment in compute (`self.x_field = v`) | Not allowed — use `self['x_field'] = v` |
