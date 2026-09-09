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

#### The logic budget, and when to stop and ask

What a database on this hosting is worth keeping is its simplicity: it stays maintainable from Studio and the
interface, and it upgrades on its own. Logic expressed as records — server actions, automation rules, sandboxed
computes — is what erodes that, so it is budgeted. The prompt states the budget for this run; it is 10 hours
where it says nothing.

Count against the budget the hours of everything that is written **differently, or written at all, only because
no Python is loaded**: server actions and their code, automation rules, computed and constrained fields in the
sandbox, crons, and the `ir.model` / `ir.model.fields` records standing in for a class a module on Odoo.sh
would simply declare.

Do not count what a developer on Odoo.sh would have written the same way: views and QWeb, reports, SCSS,
JavaScript and OWL components, and the data the module ships either way. Those are free here — a heavy interface
on a light database is exactly what this hosting is for.

**Stop and ask** — before writing the rest of the analysis — when either is true:

- the counted logic goes over the budget;
- meeting a requirement takes a workaround rather than an implementation: a chain of automation rules standing in
  for one override, a cron polling for something an override would catch, a standard button rebound to a server
  action, standard behaviour reimplemented because the standard method cannot be reached.

Put the question to the developer with the figures behind it: which requirements push it over, the counted logic
total against the budget, the workarounds SaaS forces and what each one costs in fragility, and what Odoo.sh
would allow instead. Then wait. The hosting is the developer's call and the client's, never yours, and an
analysis written past this point against the wrong one is work thrown away.

If the run cannot take an answer, do not pick silently: write the analysis for SaaS as instructed, and open it
with that same question and the same figures, so the hosting is settled before anything is built.

### Odoo.sh

A custom module is deployable, and the estimate has to account for the branch, the build and the deployment of
that module. Account for that **once**, where the target has a place for it — the setup time of a record that
carries one, a caveat where it does not — and never as a piece of the work itself: a build and a deployment are
not a data file, not a script, and not a `post_init_hook`, which stays scoped to what its own code does rather
than to how the module reaches the client.

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
- On Odoo Online, the logic hours carry a budget of their own on top of the total — see the hosting section
  above, and ask before going over it.

## What the analysis has to contain

- One entry per functional requirement, never one block of prose covering all of them. Where the target
  the prompt names has a structure of its own — the typed lines of a record, the fields of a tracker — write
  the requirements into it; markdown is the shape only where it has none.
- Per requirement: the impacted models and fields, how it is implemented, and an estimate in hours. The
  analysis is read as a build plan — by the developer writing the module, or by an AI generating it — and
  the functional analysis it was written from is always delivered next to it. So write the **how**: the
  models and fields, the method to override and what it does there, the view and the node, the domain, the
  file each thing goes in, the order things happen in. Why the client wants it is already written
  elsewhere: restating the requirement, justifying it or recalling the business context is noise, and an
  entry saying nothing the reader cannot see for themselves is better left empty.
- Whatever the analysis calls data is data files the module ships — records in XML, installed with the
  development that needs them. Configuration made by hand in the interface, or a procedure for somebody
  to follow, is not what is built: it does not belong in the analysis.
- Once, up front: whether this is a new custom module or a change to one that already exists. Read the source at
  hand rather than guessing. A new module is the default; name an existing one only when extending it genuinely
  makes more sense.
- The assumptions you had to make and the requirements you left out of the estimate. A few sentences — it is
  read next to the analysis, not instead of it.

Where the analysis is delivered — the conversation, a tracker, a database record — is told to you by the prompt.
