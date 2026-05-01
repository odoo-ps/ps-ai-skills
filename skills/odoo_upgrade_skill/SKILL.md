---
name: odoo_upgrade_skill
description: "Expert workflow for Odoo module upgrades (Migration, Refactoring, Resolution)."
---

# Odoo Upgrade Workflow Skill

This skill provides a structured, step-by-step process for upgrading Odoo modules between versions.

## 🚀 UPGRADE WORKFLOW (Strict process for EACH module)

### Step 1: Manifest & Preparation
- **Update Version**: Set the version in `__manifest__.py` to `[target_ver].1.0.0`.
- **Atomic Commit**: `[UPG] <module_name>: bump version to [target_ver].1.0.0`

### Step 2: Automatable Refactoring (XML Views)
- **Convert Views**: Use `odev upgrade-code -V [target_ver] <db> --from [from_ver] --to [target_ver] --glob "<module>/**"`. 
- This is the MANDATORY tool for XML migrations (handling renamed views, modified architectures, etc.).
- **Atomic Commit**: `[UPG] <module_name>: adapt XML views to Odoo [target_ver] syntax`

### Step 3: Iterative Resolution (Python & Security)
- **Fail Fast (Static Validation)**: Before launching Odoo, validate your changes:
  1. Python: `ruff check path/to/file.py --select E999,F821,F822,F405`.
  2. XML: `xmllint --noout path/to/file.xml`.
- **Run Verification**: Launch `odev run -V [target_ver] <db> -i <module> --stop-after-init`.
- **Log Filtering**: If Odoo fails, use `grep -A 10 -B 10 "Traceback"` to extract only the relevant error.
- **Data Migration**: If you modify models (renaming fields, changing types, deleting tables), assessment of impact on production data is MANDATORY. Load `odoo_upgrade_utils` for helpers.
- **Functional Impact Analysis**: For every fix, determine the functional consequence (what does the user see? what flow is affected?).
- **Iterative Fix**:
  1. **SEARCH origin**: Use `grep` in Odoo source to find the Breaking Change origin (Commit Hash).
  2. **FIX**: Correct the code.
  3. **ATOMIC COMMIT**: `[UPG][task-id] module: description`. Include the Odoo Hash and explanation.
- **Repeat** until the module installs and runs without errors.

### Step 4: Module Closure
- **Upgrade Bible**: Update `UPGRADE.md` with Technical Summary and Functional Testing scenarios. Document all technical changes and their functional impacts.

---

## 🧪 FINAL PHASE: Validation
Once ALL modules have been upgraded:
1. **Create a fresh DB**: `odev create -f -V [target_ver] <test_db> -i base`.
2. **Install & Test**: Run `odev test --ai -V [target_ver] <test_db> -i [module_list]`.
3. **Handle Failures**: If tests fail, resolve them following the `test_skill` rules.

## 📝 COMMIT GUIDELINES
- One commit per discrete fix. **NO MASSIVE COMMITS**.
- **Format**: `[UPG][task-id] module_name: Concise description`
- **Body**: Detailed description + **MANDATORY** Source (Odoo Commit SHA, PR, or Core path).
