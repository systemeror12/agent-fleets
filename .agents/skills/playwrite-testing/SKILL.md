---
name: playwrite-testing
description: Use for custom app browser tests rather than Odoo-specific web-client tests.
---

# Playwright testing

The work has two parts: a user workflow that passes through the UI and evidence another developer can inspect.

## Runbook

1. Find the local test contract. Read repository instructions, package scripts, Playwright config, global setup and teardown, existing helpers, and the target screen code. Resolve the working directory, base URL, browser projects, setup services, required environment, and artifact paths. Use the repository's configured commands instead of inventing new ones.

   Completion: the test command and every required precondition are known.

2. Model the user journey. Start from the acceptance path: navigate, inspect the initial state, interact, observe loading or validation, submit, and verify the resulting screen. Prefer `getByRole`, `getByLabel`, `getByText`, and accessible names. Use stable test IDs when the UI has no useful semantic locator. Reserve CSS classes, `nth()`, and DOM order for structural checks that have no better target.

   Completion: each acceptance step has a user action and an observable assertion.

3. Cover the states users can reach. For dialogs and forms, check accessible name and description, initial focus, validation, disabled controls during submission, success or error feedback, Escape and cancel behavior, outside-click behavior, and focus restoration. Exercise relevant narrow viewports and scrolling. For role-dependent flows, use a separate browser context and verify both the visible permissions and the forbidden action.

   Completion: the test covers the behavior that can regress, including the paths beyond a happy-path click sequence.

4. Capture useful evidence. Take screenshots at meaningful checkpoints with `testInfo.outputPath("descriptive-name.png")`. Preserve the repository's trace, video, and HTML-report settings. Use `test.step` for readable sections. Use run-scoped data for created records and close extra browser contexts in `finally` blocks. Keep secrets and personal data out of screenshots, logs, and test names.

   Completion: a failure identifies the first broken action and leaves an inspectable artifact when the configuration supports one.

5. Run visibly. Use the repository's Playwright UI Mode, Inspector, or headed command for developer validation. Target the smallest relevant spec or test. A headless-only command is a CI check and must be reported separately. If setup services, credentials, or browser binaries block the run, report `BLOCKED` or `UNRUN` with the missing precondition.

   Completion: the developer can inspect the visible run, or the report states exactly why it did not run.

6. Hand off the result in this format:

   ```text
   Playwright validation: PASS | FAIL | BLOCKED | UNRUN
   Command: <exact command>
   Scope: <spec/test and browser project>
   Observed: <visible result and environment label, without secrets>
   Steps: <step names and actual results; summarize passing steps>
   Artifacts: <trace, video, screenshot, HTML report, or none>
   Cause/next action: <first failure, missing precondition, or follow-up>
   ```

   `PASS` requires a visible run. `FAIL` names the first broken step and artifact. `BLOCKED` and `UNRUN` name the missing precondition.

## Good example

This follows the UI, uses accessible locators, checks dialog behavior, and leaves named screenshots:

```ts
import { expect, test } from "@playwright/test";

test("APP-001 creates a record", async ({ page }, testInfo) => {
  const code = `RUN-${Date.now()}`;

  await test.step("open the records screen", async () => {
    await page.goto("/records");
    await expect(page.getByRole("heading", { name: "Records" })).toBeVisible();
    await page.screenshot({
      path: testInfo.outputPath("records-empty.png"),
      fullPage: true,
    });
  });

  await test.step("create and verify the record", async () => {
    await page.getByRole("button", { name: "Create record" }).click();
    const dialog = page.getByRole("dialog", { name: "Create record" });
    await expect(dialog).toBeVisible();
    await expect(dialog).toHaveAccessibleDescription(/required|register/i);
    await expect(page.getByLabel("Name")).toBeFocused();
    await page.getByLabel("Name").fill(`Test record ${code}`);
    await page.getByRole("button", { name: "Save" }).click();
    await expect(page.getByRole("status")).toContainText("created");
    await expect(page.getByRole("row").filter({ hasText: code })).toBeVisible();
    await page.screenshot({
      path: testInfo.outputPath("record-created.png"),
      fullPage: true,
    });
  });
});
```

## Bad example

This hardcodes the server, relies on DOM order, waits an arbitrary duration, and bypasses the UI:

```ts
test("creates a record", async ({ page }) => {
  await page.goto("http://localhost:3000");
  await page.locator(".primary-button").nth(1).click();
  await page.waitForTimeout(1000);
  await page.evaluate(() => fetch("/api/records", { method: "POST" }));
});
```

```sh
npx playwright test e2e/records.spec.ts
```

The test gives no reliable user-facing assertion, and the default headless run gives the developer no browser session to inspect.
