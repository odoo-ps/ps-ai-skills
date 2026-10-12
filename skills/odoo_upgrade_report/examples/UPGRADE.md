# acme_sale_project — Upgrade 17.0 → 19.0

> **Risk:** High · **State:** Installs, no automated tests

Photos and files attached to a Field Service sales order are moved every night to the order's project, or to its task
for a subscription order, so technicians find them there. In 17.0 the task's **Documents** button also showed a
subscription task's own documents.

## For functional

### What changes for users

| Change | Where | Kind | Decision needed |
| --- | --- | --- | --- |
| Odoo 19 stores documents per project only and removed the task's Documents button. Our redirect of that button is gone with it; task documents are found through the project. | Project > a task | Removed | Yes, decision 1 |
| The nightly move of order attachments to the project (or task, for subscriptions) works as before. | Scheduled action "Transfer attachments to project" | Restored | No |

### Test scenarios

| ID | Scenario | Steps | Expected | Priority | Dev check |
| --- | --- | --- | --- | --- | --- |
| acme_sale_project-T1 | Order attachment reaches the project | 1. Field Service > create a project with a task. 2. Sales > create an order with a line linked to that task, attach a photo in the chatter. 3. Confirm the order. 4. Settings > Technical > Scheduled Actions > "Transfer attachments to project" > Run Manually. | The photo is now in the project's documents and no longer flagged as waiting. | P1 | Shell |
| acme_sale_project-T2 | Same flow on a subscription order | As T1, with an order that has a recurring plan. | The photo is attached to the **task**, not the project. | P1 | Shell |
| acme_sale_project-T3 | Where task documents are now | Open a regular task, then the subscription task from T2. | No Documents button on either (decision 1). The project's documents open from the project; the photo from T2 is in the task's chatter. | P2 | None |

### Decisions needed

1. **Is it acceptable that tasks have no Documents button any more?** In 17.0 a subscription task showed its own
   documents there. Odoo 19 removed the link between a document and a task, and the button with it. Photos moved from
   subscription orders still land on the task (T2), in its chatter. Options: accept it; or rebuild a per-task view, for
   example by tagging documents with the task (estimate on request). If nobody decides, users find task documents through the project or the task chatter.

### Manual checks required

- Task form, list and kanban views edited in Studio — any column or button showing the smart document count
  (`smart_document_count`) or the subscription flag (`is_subscription_task`) still shows, but now with project-level
  numbers; the standard task form no longer shows it.
- Automations or reports that read the subscription flag — the flag is still computed, but nothing in the module
  branches on it anymore.

## Technical log

### Changes

- **`ir.cron.numbercall` removed** — `data/ir_cron_data.xml` set `numbercall = -1`; the field and `doall` no longer
  exist and an active cron now runs indefinitely. Dropped the field. `Source: 2700dd3fd4c45b5282d3803182dfffdfc4418ad8`
  (odoo)
- **Per-task documents removed upstream** — `project.task` lost `document_count`, `document_ids` and
  `action_view_documents_project_task()`; `documents_project/models/project_task.py` was deleted. Deleted
  `views/project_views.xml`, whose two xpath anchors targeted the removed stat button, and removed it from the
  manifest. `Source: f74675c4e3ea7ce6d15cb54fa3c56b2bde45bde4` (enterprise)
- **`_compute_smart_document_count` depended on the removed `document_count`** — fails at registry setup, before
  any data loads. Collapsed the compute and `action_view_smart_documents` to the project-level count and action.
  Kept `is_subscription_task` as a related field in case Studio views or reports read it. Same source as above.

### Verification run

- `odev run -V 19.0 acme19 -i acme_sale_project --stop-after-init` — installs cleanly.
- `odev shell acme19`: created an FSM project and task, an order line linked to the task and an image attachment on
  the order. The attachment is flagged on create, keeps the flag through `action_confirm()`, and
  `_cron_transfer_attachments_to_project()` moves it to `res_model='project.project'`, `res_id=<project>`, then
  clears the flag. Repeated with a subscription order (`plan_id` set): the attachment moves to
  `res_model='project.task'`, `res_id=<task>` instead, as the subscription branch of the cron intends.
- `task.smart_document_count` computes without error from `project_id.document_count`.

### Pre-existing issues

- None found.

### Later fixes

| Date | Ticket | Commit | What was fixed |
| --- | --- | --- | --- |
