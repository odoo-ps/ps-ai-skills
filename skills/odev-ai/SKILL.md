---
name: odev-ai
description:
    "Maps a technical analysis or functional specification onto the presales.analysis Odoo data structure via MCP
    tools, populating the parent record and its typed child specification lines."
---

# SKILL: Ps-Tools Analysis Architecture Mapping & Structuring

## Main Objective
Translate a technical analysis or functional specification into a precise and qualified Odoo data structure. The goal is to populate the main parent record `presales.analysis` and all its child technical specification lines via MCP tools. All child lines must explicitly populate the `analysis_id` field with the ID of the parent record.

---

## Target Odoo Database Architecture Catalog

For each component discovered during the analysis, the data must be mapped according to the strict models and fields defined below:

### 1. Central Workspace (`presales.analysis`)
* **Usage:** Main parent record. Initialize this record first to capture the main analysis workspace ID.
* **Key Fields:** * `is_new_module`: (Boolean) Default assumption: the requirement is built as a new custom module. Set to `false` only when it genuinely extends a module that already exists and it makes sense to keep the change there rather than add a new module.
    * `existing_module_name`: (String - Technical name) Required when `is_new_module` is `false`, naming the module being extended.

### 2. Business Logic & Python Workflows (`presales.business_flow_line`)
* **Usage:** Used for custom methods, computation overrides, or major business workflows.
* **Key Fields:** * `action`: `'add'` | `'override'`
    * `model`: (String - e.g., `'sale.order'`)
    * `name`: (String - Method/Field technical name)
    * `action_name`: (String - Label)
    * `depends_fields`: (String)
    * `estimated_time`: (Float)
    * `description`: (Text)

### 3. HTTP & Routing Controllers (`presales.controller_line`)
* **Usage:** Used for web controllers, routes, or custom endpoints.
* **Key Fields:** * `action`: `'add'` | `'override'`
    * `action_name`: (String - Route name)
    * `estimated_time`: (Float)
    * `description`: (Text)

### 4. Initial / Demo Data (`presales.data_line`)
* **Usage:** Used for required data files, CSV loads, or XML base records installation.
* **Key Fields:** * `model`: (String - Target model string)
    * `estimated_time`: (Float)
    * `description`: (Text)

### 5. Database Fields Modifications (`presales.field_line`)
* **Usage:** Used whenever custom fields are added or existing fields are extended.
* **Key Fields:** * `model_type`: `'existing'` | `'new'`
    * `action`: `'add'` | `'override'` | `'studio'`
    * `field_name`: (String - Technical name)
    * `model`: (String - Target model)
    * `type`: (Many2one link ID to data type)
    * `comodel_name`: (String)
    * `default_value`: (String)
    * `is_compute` | `is_stored` | `is_required` | `is_readonly` | `is_tracked` | `index` | `inherit_mail_thread` | `inherit_mail_activity_mixin`: (Boolean)
    * `domain`: (String)
    * `relation`: (String - M2M relation name)
    * `estimated_time`: (Float)
    * `description`: (Text)

### 6. External Connectors & APIs (`presales.integration_line`)
* **Usage:** Used for file synchronization, APIs, cron data syncs, or webhook pipelines.
* **Key Fields:** * `model`: (String)
    * `name`: (String - Integration label)
    * `flow`: `'in'` | `'out'` | `'out_real'`
    * `type`: `'ftp'` | `'sftp'` | `'api'`
    * `format`: `'csv'` | `'json'` | `'text'` | `'pdf'` | `'sql'` | `'xml'`
    * `interval_number`: (Integer)
    * `interval_type`: `'minutes'` | `'hours'` | `'days'` | `'weeks'` | `'months'`
    * `filter`: (String)
    * `process` | `post_process`: (Text specification)
    * `estimated_time`: (Float)

### 7. Frontend Web Client & Assets (`presales.js_line`)
* **Usage:** Used for custom widgets, Owl components, or custom style sheets styling.
* **Key Fields:** * `action`: `'add'` | `'override'` | `'new'`
    * `type`: `'js'` | `'css'`
    * `assets`: `'backend'` | `'frontend'` | `'common'`
    * `file_path`: (String)
    * `object_name`: (String)
    * `name`: (String - Method string)
    * `action_name`: (String)
    * `estimated_time`: (Float)
    * `description`: (Text)

### 8. Documents & PDF Layouts (`presales.report_line`)
* **Usage:** Used for custom QWeb actions, printed actions documents, or HTML report templates.
* **Key Fields:** * `action`: `'add'` | `'override'`
    * `model`: (String)
    * `view`: (String - View XML external ID)
    * `estimated_time`: (Float)
    * `description`: (Text)

### 9. Database Installation & Migration Hooks (`presales.script_line`)
* **Usage:** Used for pre/post installation hooks, data migration paths, or optimization raw SQL scripts that ship *inside the custom module itself*. Scope each line to what the hook's own code does (e.g. backfilling a field, scaffolding module-level tests) — never to hosting or deployment workflow (branch creation, builds, staging/production deployment). Those belong on the analysis as a whole, not on a script line, and only when the hosting rules call for stating them.
* **Key Fields:** * `action`: `'pre_init_hook'` | `'post_init_hook'` | `'uninstall'` | `'post_hook'` | `'pre_migrate'` | `'post_migrate'` | `'end_migrate'` | `'sql'`
    * `model` | `field_name`: (String)
    * `estimated_time`: (Float)
    * `description`: (Text)

### 10. Security Groups & Access Control (`presales.security_line`)
* **Usage:** Used for security matrix layout, explicit access right modifications, or structural record rules.
* **Key Fields:** * `action`: `'add'` | `'override'`
    * `type`: `'acl'` | `'record_rules'` | `'new_group'`
    * `name`: (String)
    * `model`: (String - Technical target string)
    * `groups`: (String - Names comma-separated)
    * `domain`: (String - Evaluation filters)
    * `as_read_access` | `as_write_access` | `as_create_access` | `as_delete_access`: (Boolean)
    * `estimated_time`: (Float)
    * `description`: (Text)

### 11. Frontend Portal & E-commerce Templates (`presales.website_template_line`)
* **Usage:** Used for QWeb portal pages adjustments or specific website template customization block extensions.
* **Key Fields:** * `action`: `'add'` | `'override'` | `'disable'`
    * `view`: (String - View External ID)
    * `field`: (String - Target node string)
    * `estimated_time`: (Float)
    * `description`: (Text)