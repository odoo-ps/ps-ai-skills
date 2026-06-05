# Module Structure

## Required Directories
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

## File Naming Rules
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

**Controllers**: `controllers/<module_name>.py` (not `main.py`). Inherited controller: `controllers/<inherited_module_name>.py`.
