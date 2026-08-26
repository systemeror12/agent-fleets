---
name: playwrite-odoo-testing
description: Use when adding, changing, or running browser tests in an Odoo repository.
---

# Playwright Odoo testing

The work has two parts: a user-visible Odoo journey and a report another developer can verify.

## Runbook

1. Find the local contract. Read the repository instructions, package scripts, Playwright config, existing helpers, and the target addon views, actions, and security. Resolve the base URL, database, authentication setup, test directory, project, and artifact paths from local config and environment variables. Keep credentials out of code and output.

   Completion: the test command and every required precondition are known.

2. Model the journey. Write the path a person follows: login, app or menu, view, form or list, data entry, action, result. Prefer role, label, and visible-text locators for controls. Use stable Odoo anchors for structure, including `[data-menu-xmlid]`, `.o_field_widget[name="..."]`, `.o_form_view`, `.o_list_view`, `.o_data_row`, `.o_dialog`, `.o_statusbar_status`, and `.o_loading_indicator`. Assert each transition.

   Completion: every acceptance step has one user action and one visible assertion.

3. Keep Odoo mechanics in helpers. Reuse or extend the local `helpers.ts`. Useful seams include `openMenu`, `fieldWidget`, `fillField`, `selectField`, `selectMany2One`, inline-row helpers, `saveForm`, `clickAction`, `openNewForm`, and `expectView`. Add a helper when it hides repeated Odoo DOM behavior. Keep XML IDs and field names at the spec call site. Keep CSS details in the helper.

   Completion: the spec reads as a business workflow, and repeated Odoo DOM work has one implementation.

4. Add evidence while writing. Use run-scoped names for created records. Wait on visible Odoo state or the loading indicator. Use `test.step` or the repository's child-step helper. Give each meaningful step a stable ID and a short passed or failed message. Preserve the configured trace, video, screenshot, and HTML report settings.

   Completion: a failure identifies the first broken step, its expected result, and any available artifact.

5. Run visibly. Find the repository's configured UI command in its local instructions, package scripts, and Playwright config. Prefer the package script because it may load environment and authentication setup. If no UI script exists, use Playwright's UI mode directly:

   ```sh
   npm run <ui-test-script> -- <affected-spec>.spec.ts
   npx playwright test --ui <affected-spec>.spec.ts
   ```

   A headless-only run is a CI check. It is not the developer validation reported by this skill. If the visible run cannot start because of a missing server, credential, database, or browser, report `BLOCKED` or `UNRUN` and name the missing precondition.

   Completion: the developer can inspect the visible run, or the report clearly states why it did not run.

6. Hand off the result in this format:

   ```text
   Playwright validation: PASS | FAIL | BLOCKED | UNRUN
   Command: <exact command>
   Scope: <spec/test and browser project>
   Observed: <visible result and environment label, without secrets>
   Steps: <step IDs and actual results; summarize passing steps>
   Artifacts: <trace, video, screenshot, HTML report, or none>
   Cause/next action: <first failure, missing precondition, or follow-up>
   ```

   `PASS` requires the visible command and scope. `FAIL` names the first broken step and artifact. `BLOCKED` and `UNRUN` name the missing precondition.

## Good example

The spec describes the workflow. The helper owns Odoo selectors. Child steps produce messages a developer can read.

```ts
import { expect, test } from "@playwright/test";
import {
  expectView,
  fieldWidget,
  fillField,
  openMenu,
  runChildSteps,
  saveForm,
} from "./helpers";

const app = "your_module.menu_root";

test("ODOO-001 creates a job position", async ({ page }) => {
  const name = `Playwright Job ${process.env.TEST_RUN_ID ?? Date.now()}`;

  await runChildSteps([
    {
      id: "ODOO-001-01",
      passed: "Job Positions opened through the configured menu.",
      action: async () => {
        await openMenu(page, app, [
          "your_module.menu_configuration",
          "your_module.menu_job_positions",
        ]);
        await expectView(page, "Job Positions");
      },
    },
    {
      id: "ODOO-001-02",
      passed: "The new form accepted and saved the job position.",
      action: async () => {
        await page.getByRole("button", { name: "New", exact: true }).click();
        await expect(fieldWidget(page, "name")).toBeVisible();
        await fillField(page, "name", name);
        await saveForm(page);
        await expect(page.getByText(name, { exact: true }).first()).toBeVisible();
      },
    },
  ]);
});
```

Run it with the visible UI command and report the step messages plus any trace, video, screenshot, or HTML report.

## Bad example

This uses incidental DOM order, skips a visible assertion, and runs headless by default:

```ts
test("creates a job position", async ({ page }) => {
  await page.goto("/web");
  await page.locator("input").nth(3).fill("Job");
  await page.locator("button").nth(8).click();
});
```

```sh
npx playwright test playwright/job-position.spec.ts
```

It gives the developer no stable Odoo boundary, step result, or inspectable browser run.
