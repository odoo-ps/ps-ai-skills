---
name: odoo-coding-guidelines
description:
    "PS rules for writing or reviewing customer Odoo modules, used together with Odoo's official odoo-guidelines and
    odoo-web-guidelines skills: which official rules change on customer projects and older Odoo versions, plus the
    conventions they do not cover. Invoke when writing, reviewing or validating code in a customer module."
---

# Odoo coding guidelines for customizations

Odoo ships official skills in the `skills/` directory of `odoo/odoo`, from `20.0` on: `odoo-guidelines`,
`odoo-web-guidelines`, `odoo-review`, `odoo-security` and `odoo-git`. They are written for developing Odoo itself.
This skill holds only what is different or missing when the code is a customization, so use both:

1. Read the official section for what you are touching.
2. Check it against [Older Odoo versions](#older-odoo-versions) for the version of the project.
3. Apply [On a customer project](#on-a-customer-project). Where it differs from an official rule, it wins.
4. Read what applies here: [python.md](python.md), [xml.md](xml.md), [web.md](web.md).

`odev` installs the official skills of the version the agent runs against, but versions before `20.0` ship none. If
they are not installed, say so rather than working from memory, and read them from the `skills/` directory of a `20.0`
or later checkout of `odoo/odoo`.

## Scope of a change

These guidelines describe the code you write. They are not a licence to rewrite code that is already there.

- Change only what the task asks for. A reformatting, a rename, a reordered import or a changed quote style in code
  you did not come to modify is noise in review, breaks `git blame` and buries the change someone has to check.
- Never reformat a file wholesale or fix up a module on the way past, even when it plainly breaks these rules. Existing
  code is brought up to standard when someone asks for it, as its own change with its own review.
- The exception is an explicit instruction: if asked to reformat, clean up or apply the guidelines to existing code,
  that is then the task.

## Formatters and pre-commit

- Run `pre-commit` only if the repository commits a `.pre-commit-config.yaml` at its root, and only on your own
  changes: `pre-commit run --files <the files you changed>`. Never `--all-files`: on a repository that was not formatted
  before, it rewrites every file and turns a five-line change into a thousand-line diff.
- No configuration means formatting is not enforced there: match the style of the file you are editing.
- The same goes for any formatter run by hand (`ruff format`, `black`, `prettier`, `eslint --fix`): only the files you
  changed, and only if the repository already uses it.

## On a customer project

| Official rule | On a customer project |
|---|---|
| Changes in a stable version (`stable.md`), and `odoo-review` adding it for a stable target | Does not apply. A `18.0-...` branch of a customer repository is not a stable branch of Odoo: adding stored fields, changing signatures and changing XML ids are normal here. Apply what `odoo-review` says for master: the guidelines bind new code, not the code around it. |
| Data model changes | Allowed, but a rename or removal on a module already installed in production needs a migration script, or the data is lost. See `custom-util`. |
| Avoid patching JavaScript code | Inside Odoo only, as the rule itself says. `patch()` is a normal tool in a customer module. |
| `odoo-git`: Branches and pull requests | Does not apply: no target branch policy, forward-port bots, `odoo-dev` remote or CLA. For commit messages, see `odoo-commit-message-guidelines`. |
| Manifest: `version` follows semantic versioning | Use `<odoo version>.<major>.<minor>.<patch>`, such as `18.0.1.0.0`. |
| Manifest: `license` defaults to `LGPL-3` | Follow the other modules of the repository, usually `OEEL-1` or `OPL-1`. |
| `ruff.toml` at the repository root | Use the configuration of the customer repository, when it has one, not the one of `odoo/odoo`. See [Formatters and pre-commit](#formatters-and-pre-commit). |

Never edit standard Odoo code. Extend it from the customer module.

## Older Odoo versions

The official skills describe the current version. Most customer projects run an older one, where some of their rules
do not exist yet:

| Official rule | From | On older versions |
|---|---|---|
| `<list>` views | 18.0 | `<tree>` |
| `self.env._()` | 18.0 | `from odoo import _` |
| No `/** @odoo-module **/` | 18.0 | 16.0 and 17.0 core JavaScript files carry it |
| `models.Constraint`, `models.Index` | 19.0 | `_sql_constraints`, which 19.0 and later no longer support |
| `fields.Domain` | 19.0 | `odoo.osv.expression` |
| `ir.access` rows in `security/ir.access.csv` | 20.0 | `security/ir.model.access.csv` and `ir.rule` records |

This list is not complete. Before writing or flagging an API, confirm it exists in the version of the project, as the
Version traps section of `odoo-review` says.
