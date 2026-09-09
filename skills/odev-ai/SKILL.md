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

## Rules That Apply To Every Line

* **A `description` says how the line is built, not why it is wanted.** It is the build instruction of
  whoever implements it — a developer, or an AI generating the module from this analysis — and the
  functional analysis is always delivered next to it. So write the technical *how*: the model and field it
  lands on, the method overridden and what it does there, the view and the xpath, the domain, the file it
  goes in, the order things happen in. Restating the requirement, justifying it or recalling the business
  context is noise; so is repeating what the other fields of the line already carry. Nothing useful to add
  means leaving it empty. Never quote the task or the board back at the reader ("gives the follow-up asked
  for", "answers 'remove the SO information'", "as per the diagram") and never argue the design choice
  ("dropped rather than transferred because..."): say what is built, and where a choice needs confirming
  put it in the tab description instead.
* **Nothing about the module itself is a line.** The scaffolding, the manifest, the development branch, the
  build and the deployment are not lines of any tab — not a `presales.data_line` on `ir.module.module`, not
  a script line. Ps-Tools derives `setup_time`, and `migration_time` when the analysis needs SH, from the
  weight of the database; a line raised for any of it counts the same hours twice.
* **Never write a line count in a `description`.** No "~40 LOC", no "about 30 lines of Python", no "≈ 3 files
  of 20 lines". The line count is *derived* from `estimated_time`, never the reverse — restating it in the text
  duplicates that computation and contradicts it the moment the estimate is adjusted. `estimated_time` alone
  carries the size.

---

## Target Odoo Database Architecture Catalog

For each component discovered during the analysis, the data must be mapped according to the strict models
and fields defined below. `mcp__ps_tools__get_analysis_line_schema` reads the accepted fields and values off
the live Ps-Tools models, so **it wins wherever it disagrees with this catalog**: what the catalog carries
and the schema cannot is what each kind of line *means*, and what does not belong on it.

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
* **Usage:** One data **file the module ships**, per target model: the records the development needs to work
  — a sequence, a mail template, a server action, an automation rule, a system parameter, a default
  configuration record — written as `<record>` in a file of the module's `data/` (or a `.csv` loaded by the
  manifest) and installed with the code.
* **Never a manual configuration or a process.** Not a setting somebody clicks in the interface, not a
  procedure for a consultant or the client to follow, not a step to run by hand on the database. What
  cannot be a data file the module installs is not a data line, and is not what the analysis delivers.
* The `description` names the records the file creates and the values that matter (xml_id, the fields set,
  the domain or the trigger of an automation), so the file can be written from it.
* **Key Fields:** * `model`: (String - Target model string)
    * `estimated_time`: (Float)
    * `description`: (Text)

### 5. Database Fields Modifications (`presales.field_line`)
* **Usage:** Used whenever custom fields are added or existing fields are extended.
* **`action: 'studio'` depends on the hosting, and the prompt states which one applies:**
    * **Odoo Online (SaaS):** allowed — a Studio field is a legitimate implementation there.
    * **Odoo.sh / On-Premise:** not allowed. A Studio customisation that already exists in the database has to
      be **migrated into code**: raise the field as `'add'` (or `'override'`) on the custom module, and estimate
      the migration of the existing Studio field *and of the data it already holds* (a `presales.script_line`
      with `'pre_migrate'` / `'post_migrate'`). Never leave a `'studio'` line on an Odoo.sh analysis.
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
* **Prefer a migration script over an init hook.** `pre_init_hook` and `post_init_hook` are deprecated and are
  the wrong default: use `'pre_migrate'` / `'post_migrate'` (or `'end_migrate'`) for anything touching existing
  data — backfilling a field, renaming a model, moving Studio customisations into code. Pick an init hook only
  where the work genuinely can run nowhere but the module's first install, and say why in the `description`.
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