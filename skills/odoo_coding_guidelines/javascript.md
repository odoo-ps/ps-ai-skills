# JavaScript Guidelines

## Static File Structure
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

## JS Rules
- Use `'use strict';` in all JS files
- Use a linter (jshint, eslint)
- **Never** add minified JS libraries
- Use **PascalCase** for class declarations
- **Do NOT add `/** @odoo-module **/`**: this directive is deprecated and must not appear in JS files
