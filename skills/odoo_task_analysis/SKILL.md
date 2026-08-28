---
name: odoo_task_analysis
description:
    "Method for analysing an Odoo task and estimating it: what to read before concluding, what each hosting
    allows, how to ground an estimate in a line count, and what the written analysis has to contain."
---

# Odoo Task Analysis

Use this skill when analysing an Odoo task to say what has to be built and what it costs — `odev analyze`
loads it for you.

The prompt carries the **facts** of the run: the task and its description, the client, the target version, the
hosting, where the diagrams and the standard Odoo source were mounted, the throughputs to estimate with, and
where the finished analysis goes. This skill carries the **method**, which is the same from one run to the next.

## Before concluding

- Read the source before drawing a conclusion. Guessing what Odoo already does is the single most expensive
  mistake an analysis can make: it turns standard behaviour into a line of estimated work.
- Where the standard source of the target version is mounted read-only, read it to tell what Odoo already does
  from what has to be built. **Only the second is estimated.** Never modify it.
- Identify the impacted models, views and modules. Flag a requirement the task leaves missing, contradictory or
  ambiguous rather than inventing an answer for it.
- Diagrams and embedded images are exported to files the prompt names. Read them: the description refers to each
  by that same file name, where the picture belonged. The models and views you propose must match the structure
  a diagram shows.

## What the hosting allows

The prompt states which one applies. Read that entry before proposing any implementation.

### Odoo Online (SaaS)

No Python is **loaded** — a module can be imported, but none of its `.py` files is ever executed. So no new
class, no override of `create`, `write`, `unlink` or any other method, no compute written in code, no controller
or route, no `post_init_hook`, no third-party Python library.

Everything a module carries as *data* still works. An imported module ships `.xml`, `.csv` and `.sql` files plus
its `static/` assets, and data can create records of any model: `ir.model` and `ir.model.fields` for new models
and fields, views, actions, `ir.model.access` and `ir.rule`, automation rules, crons, server actions and
computed fields whose Python is `safe_eval`'d rather than loaded, reports, mail templates.

The choice is therefore between Studio and a **data-only importable module** — never between Studio and nothing.
Load the `odoo_saas_development` skill for the record shapes and the real limits before you estimate anything
here. Say so explicitly only for a requirement that genuinely needs loaded Python, which is the one thing that
cannot be met.

### Odoo.sh

A custom module is deployable, and the estimate has to account for the branch, the build and the deployment of
that module. Account for that **once** — its own line, or a caveat in the analysis — never folded into the
description or estimate of an unrelated one: a `post_init_hook` stays scoped to what its code does, not to how
the module reaches the client.

### On-Premise

A custom module is deployable, but nothing about the hosting, the deployment or the third-party modules already
installed can be assumed. Say which assumption you had to make.

## Estimating

- Ground every estimate in the lines of code the requirement takes to write, not a round guess. Count the lines,
  divide by the throughput the prompt gives for that language, and round to the nearest quarter hour.
- The total must not fall below the floor the prompt states: even a small requirement carries setup, testing and
  review overhead that per-line estimates alone tend to undercut. Raise the smallest item rather than inflating
  every one.
- Estimate what has to be **built**. What Odoo already does is not work.

## What the analysis has to contain

- Markdown, structured per functional requirement.
- Per requirement: the impacted models and fields, the proposed implementation, and an estimate in hours.
- Once, up front: whether this is a new custom module or a change to one that already exists. Read the source at
  hand rather than guessing. A new module is the default; name an existing one only when extending it genuinely
  makes more sense.
- The assumptions you had to make and the requirements you left out of the estimate. A few sentences — it is
  read next to the analysis, not instead of it.

Where the analysis is delivered — the conversation, a tracker, a database record — is told to you by the prompt.
