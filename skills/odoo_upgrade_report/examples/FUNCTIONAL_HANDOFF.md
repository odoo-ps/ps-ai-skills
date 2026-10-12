# Functional handoff — acme 17.0 → 19.0

Branch `19.0-upgrade` · compiled 2026-10-12 from 3 module reports and 2 fixes merged after them.
**1 decision** · **7 scenarios** (4 P1) · **2 later fixes to retest** · 3 of 3 modules covered.

How to use it: settle section 1 first, then test section 2 by flow, P1 first. Write OK / KO and the scenario ID in the
Result column; quote the ID on any ticket. *(derived)* means the scenario comes from a code change and nobody has run
it yet.

## 1. Decisions needed before sign-off

| # | Decision | Module | If nobody decides |
| --- | --- | --- | --- |
| 1 | **Tasks no longer have a Documents button.** Odoo 19 stores documents per project only. Photos from subscription orders still land on the task, in its chatter. Accept, or rebuild a task-level view? | acme_sale_project | Task documents are found through the project or the task chatter. |

## 2. Test list

### Sales

| ID | What to test | Steps | Expected | Pri | Dev check | Module | Result |
| --- | --- | --- | --- | --- | --- | --- | --- |
| S1 | Print the quotation | Confirm an order with one line marked visible on reports and one not > Print > Quotation. | The hidden line is absent; the visible line shows its product name. | P1 | Rendered | acme_sale | |
| S2 | Confirm several orders at once | Select 2+ quotations for different customers that each need a new project > Actions > Confirm. | All confirm; each order gets its own project. | P1 | Fix after report (section 3) | acme_sale | |

### Inventory

| ID | What to test | Steps | Expected | Pri | Dev check | Module | Result |
| --- | --- | --- | --- | --- | --- | --- | --- |
| V1 | Delivery template fills the delivery | Inventory > new delivery > pick a delivery template. | One line per template line, with the right product and quantity. | P1 | Shell | acme_delivery_template | |
| V2 | Products allowed on a template line | Inventory > Configuration > delivery templates > add a line. | Only goods can be picked. | P3 | None *(derived)* | acme_delivery_template | |

### Project & Field Service

| ID | What to test | Steps | Expected | Pri | Dev check | Module | Result |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P1 | Order attachment reaches the project | Attach a photo to a Field Service order, confirm it, run the scheduled action "Transfer attachments to project". | The photo is in the project's documents. | P1 | Shell | acme_sale_project | |
| P2 | Same on a subscription order | As P1, on an order with a recurring plan. | The photo is attached to the task. | P2 | Shell | acme_sale_project | |
| P3 | Where task documents are now | Open a regular task and the subscription task from P2. | No Documents button (decision 1); the photo from P2 is in the task chatter. | P2 | None *(derived)* | acme_sale_project | |

## 3. Fixed after the first delivery — retest

| Ticket | What was wrong | Retest | Module | Commit |
| --- | --- | --- | --- | --- |
| 1000001 | Confirming several orders for different customers crashed or mixed up their projects. | S2 | acme_sale | 1a2b3c4 |
| 1000002 | Printing the delivery slip (customised in Studio) crashed. | Inventory > a delivery > Print > Delivery Slip | acme_sale | 5d6e7f8 |

## 4. Check in the customer database

| Screen / record | What to look for | Module |
| --- | --- | --- |
| Sales Order form and lines | Studio columns next to the price-hiding groups | acme_sale |
| Delivery form | Studio changes around the delivery-template field | acme_delivery_template |
| Task views showing the document count | The count is now project-level and no longer on the standard task form | acme_sale_project |

## 5. Not testable on this database

- None.

## 6. Standard Odoo changes users will notice (not bugs)

| Where users meet it | What changed in Odoo 19 |
| --- | --- |
| Products | "Storable" is no longer a product type; goods have a separate inventory-tracking box. |
| Documents | Documents link to a project, no longer to individual tasks. |

## 7. Questions for the developer

- V2 is derived: confirm the allowed product types before the client tests it.

## Appendix: coverage by module

| Module | Report | Risk | Scenarios | Later fixes | Automated tests |
| --- | --- | --- | --- | --- | --- |
| acme_sale | 20 Aug | High | 2 | 2 | Yes |
| acme_sale_project | 20 Aug | High | 3 | — | No |
| acme_delivery_template | 18 Aug | Medium | 2 | — | No |
