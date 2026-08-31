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

### Scope of a change

These guidelines describe the code **you write**. They are not a licence to rewrite code that is already there.

- **Change only what the task asks for.** The diff of a dev is its requirement and nothing else. A reformatting, a
  rename, a reordered import or a changed quote style in code you did not come to modify is noise in review, breaks
  `git blame`, and buries the change someone actually has to check.
- **Never reformat a file wholesale**, and never "fix up" a module on the way past — not even when it plainly breaks
  the rules below. Existing code is brought up to standard when someone is asked to do it, as its own change, with its
  own review.
- Editing the lines the requirement needs is right. Editing the two hundred around them is not.
- The exception is an explicit instruction: if you are asked to reformat, to clean up, or to apply the guidelines to
  existing code, do it — that is then the task.

### Formatters and pre-commit

- **Run `pre-commit` only if the repository already configures it** — a `.pre-commit-config.yaml` committed at its
  root. Check before you run anything.
- When it is configured, run it **on your own changes only**:
  `pre-commit run --files <the files you changed>`. Never `pre-commit run --all-files`: on a repository that has not
  been formatted before, it rewrites every file and turns a five-line dev into a thousand-line diff.
- **Never add pre-commit to a repository that has none**, and never run `odev pre-commit`, which copies a
  configuration in and installs the hooks. Whether a client repository is formatted, and by what, belongs to the team
  that owns it. It is not a side effect of a dev.
- No configuration means formatting is not enforced there. Match the style of the file you are editing, and leave it
  at that.
- The same goes for any formatter run by hand — `ruff format`, `black`, `prettier`, `eslint --fix`: only over the
  files you changed, and only if the repository already uses it.

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

