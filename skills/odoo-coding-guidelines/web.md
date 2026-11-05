# Web: what the official guidelines do not cover

- No `'use strict';`: JavaScript modules are strict already.
- `/** @odoo-module **/` follows the version of the project. See Older Odoo versions in [SKILL.md](SKILL.md).
- Tour tests go in `static/tests/tours/`. Onboarding tours for end users go in `static/src/js/tours/`.
- Prefer Bootstrap utility classes (`text-end`, `fw-bold`, `mb-3`) over inline `style=""` where an equivalent exists.
  Layout values with no equivalent, such as `min-width`, can stay inline.
