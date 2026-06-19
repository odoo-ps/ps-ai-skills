---
name: playwright-odoo
description:
    "Drive a real Chrome browser through Playwright MCP to visually test, inspect, and interact with a local or test
    Odoo instance. Use for UI flows, views, wizards, reports, and portal pages — never against production."
---

# Playwright Odoo Browser Skill

Use this skill to validate Odoo behavior through the web UI with the Playwright MCP server.

## Use when

- Testing or reproducing an Odoo UI flow (form, list, wizard, report, portal, website).
- Verifying that a button, menu, view, or page renders and behaves correctly.
- Checking a visual rendering, or comparing the UI before/after a change.
- Inspecting browser-visible errors, missing elements, or unexpected behavior.

## Do not use when

- The task only needs reading, editing, or refactoring code.
- The answer is available from logs, the shell, or static analysis.
- The target could be a production database (data changes are not reversible).

## 🛑 Safety rules

1. **Local or test databases only.** Never automate against production.
2. **No destructive actions without explicit user confirmation** — do not delete
   records, post journal entries, validate payments, confirm orders, send emails,
   or run irreversible wizards unless clearly approved.
3. **Stop and ask** when the database, environment, or impact is unclear.
4. Use a **dedicated browser profile**, never a personal one.

## Setup (run once per machine)

The MCP connects to Chrome over the DevTools protocol (CDP). Before a browser session:

1. **Start Chrome with remote debugging and a dedicated profile** (adjust path per OS):

   ```bash
   # macOS
   open -na "Google Chrome" --args \
     --remote-debugging-port=9222 \
     --user-data-dir="$HOME/.chrome-playwright-odoo"

   # Linux
   google-chrome \
     --remote-debugging-port=9222 \
     --user-data-dir="$HOME/.chrome-playwright-odoo" &
   ```

   If Chrome is already listening on the port, do not relaunch it.

2. **Verify the debug endpoint responds:**

   ```bash
   curl -s http://localhost:9222/json/version
   ```

   Expect a JSON object with `Browser` and `webSocketDebuggerUrl`.

3. **Confirm the Playwright MCP is registered** (`claude mcp list` shows `playwright`).
   If missing, ask user to run:

   ```bash
   claude mcp add --scope user playwright -- \
     npx @playwright/mcp@latest --cdp-endpoint http://localhost:9222
   ```

If any step fails, report exactly what is missing and the command to fix it before continuing.

## Working method

1. Identify the goal, target URL, database, login state, and target menu/module.
2. Navigate with `browser_navigate` and describe key actions as you perform them.
3. Use `browser_snapshot` (structure) and `browser_take_screenshot` (visual) to confirm state.
4. `browser_wait_for` before reading results — let pages, dialogs, and async UI settle.
5. Report what you observed and whether the flow passed.

## Key MCP tools

- `browser_navigate` — open a URL
- `browser_click` — click buttons, links, menu items
- `browser_type` / `browser_fill_form` — fill fields
- `browser_wait_for` — wait for text, elements, or state changes
- `browser_snapshot` — read the accessibility tree / page structure
- `browser_take_screenshot` — capture the visual state
- `browser_console_messages` — read browser/JS errors

## Response format

- **Opened**: URL and menu path
- **Tested**: key actions performed
- **Observed**: result, errors, useful snapshot/screenshot
- **Conclusion**: pass/fail and notes
- **Next step**: fix or follow-up, if any

## Cleanup

Screenshots and snapshots are saved to disk during a session. At the end, delete any
artifacts written into the project to avoid polluting the repo. Prefer writing them to a temp dir in the first place:

```bash
# remove generated artifacts, e.g.
rm -rf /tmp/playwright-mcp
git status   # confirm no stray .png/.jpeg files were left in the working tree
```

Keep an artifact only if the user explicitly asks to.
