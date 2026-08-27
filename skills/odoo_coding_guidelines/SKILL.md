---
name: odoo_coding_guidelines
description:
    "Official Odoo coding standards for reviewing, checking, or writing Python, XML, JavaScript, and CSS/SCSS,
    plus module structure and a full-review checklist."
---

# Odoo Coding Guidelines

Use this skill when the user asks to **review, check, write, or validate Odoo code** against official coding standards.
Source: https://www.odoo.com/documentation/19.0/contributing/development/coding_guidelines.html

Load sub-files based on what is being reviewed:
- Python files, models, fields, methods → read [python.md](python.md)
- XML views, data files, templates → read [xml.md](xml.md)
- JavaScript files → read [javascript.md](javascript.md)
- CSS / SCSS files → read [css.md](css.md)
- Full code review or checklist requested → read [checklist.md](checklist.md)
- Module structure, file naming, directory layout → read [structure.md](structure.md)

For mixed reviews (e.g. a whole module), load all relevant files.

---

## General Principles

### Language
- **All code must be in English**: variable names, field names, method names, model names, comments, commit messages, XML IDs, and string literals used as technical identifiers.

### Comments
- Comments must explain **WHY**, never WHAT. The code itself should be readable enough to convey what it does.
- If a comment is needed to explain what a block of code does, **refactor** the code until it is self-explanatory, then remove the comment.
- Acceptable comments: hidden constraints, non-obvious invariants, workarounds for specific bugs, behavior that would surprise a reader.

### Naming
- **Never truncate names** to save keystrokes. Autocomplete makes long names free — short names cost readability.
  ```python
  # ❌
  def comp_tot_amt(self): ...
  inv_dt = fields.Date(...)

  # ✅
  def _compute_total_amount(self): ...
  invoice_date = fields.Date(...)
  ```

