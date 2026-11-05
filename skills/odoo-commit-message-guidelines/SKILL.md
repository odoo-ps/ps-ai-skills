---
name: odoo-commit-message-guidelines
description:
    "PS additions to the commit message format of Odoo's official odoo-git skill: the [TAG][ID] header, the [UPG] tag
    and how customer repositories name several modules. Invoke when writing, rewriting or validating a commit message
    in a customer repository."
---

# Commit messages on customer repositories

Follow the Commit message section of the official `odoo-git` skill: header, why-first body, wrapping, tags, and never
inventing a reference. Its Branches and pull requests section does not apply to customer repositories. This skill adds
what is specific to them. If `odoo-git` is not installed, see `odoo-coding-guidelines`.

## Header

```text
[TAG][ID] module: imperative summary
```

- `[ID]` is the task number, right after the tag. Leave it out when there is no task.
- With the ID in the header, no `task-123` trailer is needed. Other references, such as `opw-123`, still go last.
- Several modules: `module_a, module_b`, or `prefix_*` for a shared prefix, or `prefix_(a,b)` for known variants, or
  the main module then `*` when unrelated.

## Tags

The official tags apply, plus:

- `[UPG]`: a change made during a version upgrade, such as 16.0 to 17.0.

## Example

```text
[FIX][1234567] bookshop_custom: keep open orders after archiving a book

Archiving a book removed its open orders from the customer's portal
page, so the customer lost track of what was still to be delivered.

Search the orders directly, with active_test disabled on the book.
```

## Writing the message

- Return the raw message, without code fences, unless asked to explain it.
- Ask only for what is missing: the task number and why the change is needed.
- Do not run the commit without explicit confirmation. Write the message to a temporary file and use
  `git commit --file=<tmpfile>`, never `-m` with interpolated text.
