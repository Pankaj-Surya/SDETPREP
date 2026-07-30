# Playwright — Debugging & Tooling — Notes with Real-Time Examples

Reference: https://playwright.dev/docs/trace-viewer · https://playwright.dev/docs/debug · https://playwright.dev/docs/codegen · https://playwright.dev/docs/test-ui-mode · https://playwright.dev/docs/getting-started-vscode

Simple language, real test-automation scenarios, all in TypeScript.

---

## 1. Trace Viewer (`trace: on` / `retain-on-failure`)

**What it does:** Records a full timeline of a test run — DOM snapshots at every step, network requests, console logs, and actions — so you can **rewind and inspect exactly what the browser saw**, without re-running the test.

**Real-time example — a flaky test that fails only in CI, never locally: capture a trace only on the first retry so you're not storing traces for every passing test.**

```ts
// playwright.config.ts
export default defineConfig({
  use: {
    trace: 'on-first-retry', // good default: cheap on passing runs, full detail on failures
  },
});
```

Other common values: `'on'` (always, useful while actively debugging locally), `'retain-on-failure'` (keep only failing traces, don't retry-gate it), `'off'`.

**Real-time example — opening a trace from a failed CI run to see exactly what the page looked like right before a click failed:**

```bash
npx playwright show-trace trace.zip
```

You get a full timeline: hover over any step to see the DOM at that moment, the network calls, and console output — usually enough to spot the bug without adding a single `console.log`.

---

## 2. Playwright Inspector (`PWDEBUG=1`)

**What it does:** Opens a GUI that lets you **step through your test action by action**, pause execution, inspect/edit locators live, and see exactly what Playwright is doing in real time.

**Real-time example — a login test that intermittently fails right after clicking "Log In"; run it in inspector mode to watch what actually happens at that exact moment:**

```bash
npx playwright test example.spec.ts:10 --debug
```


The Inspector opens alongside the browser — you can click "Step over" to advance one action at a time, or use the "Pick locator" button to click any element on the page and get its recommended locator instantly.

---

## 3. Codegen (`npx playwright codegen`)

**What it does:** Opens a real browser where **your clicks and typing are recorded and turned into Playwright code** automatically — great for scaffolding a new test quickly, or discovering good locators for a page you're unfamiliar with.

**Real-time example — quickly generating the skeleton for a new "add to cart" test instead of writing locators from scratch:**

```bash
npx playwright codegen https://shop.example.com
```

Click "Add to Cart" in the opened browser, and codegen instantly writes:
```ts
await page.getByRole('button', { name: 'Add to Cart' }).click();
```

**Real-time example — generating code for a mobile viewport to test responsive behavior:**

```bash
npx playwright codegen --device="iPhone 13" https://shop.example.com
```

> Codegen output is a great **starting point**, not a finished test — always review and clean up the generated locators/assertions afterward.

---

## 4. VS Code extension

**What it does:** Adds a "Testing" sidebar in VS Code showing every Playwright test, with green "run" and "debug" icons right in the gutter next to each `test(...)` block — no terminal commands needed for day-to-day work.

**Real-time example — a developer just wrote a new checkout test and wants to run just that one test with a live debugger, directly from the editor:**

1. Click the green ▶ icon next to `test('completes checkout', ...)` in the gutter.
2. Click the 🐞 debug icon instead to pause on failures and step through with breakpoints.
3. Use "Pick locator" from the sidebar to hover elements in a running browser and get the exact locator Playwright recommends.

The extension also shows inline pass/fail indicators directly in the file after a run, so you don't need to switch to a terminal or HTML report to see results while iterating.

---

## 5. `--ui` mode (interactive test runner)

**What it does:** A dedicated UI (separate from the VS Code extension) that shows your whole test suite, lets you filter/run subsets, watch tests re-run live as you edit code, and gives you a timeline + trace view per test — all without leaving one window.

**Real-time example — iterating on a new "checkout" test suite, wanting instant feedback and a visual timeline for every run:**

```bash
npx playwright test --ui
```

Inside UI mode you can:
- Filter to just failing tests, or tests matching a name.
- Watch a test re-run automatically as you save the file.
- Click any step to see the DOM snapshot at that exact moment (like an always-on Trace Viewer).

---

## 6. Step-through debugging (breakpoints)

**What it does:** Pause a running test at a specific line — either a real IDE breakpoint (VS Code) or Playwright's own `page.pause()` — and inspect the live browser state before continuing.

**Real-time example 1 — pausing mid-test to manually poke around the page and figure out why a locator isn't matching:**

```ts
test('debug a flaky checkout step', async ({ page }) => {
  await page.goto('/checkout');
  await page.pause(); // execution stops here; Inspector opens, browser stays open
  await page.getByRole('button', { name: 'Place Order' }).click();
});
```

**Real-time example 2 — a real breakpoint in VS Code, set by clicking in the gutter next to a line, then running the test in Debug mode:**

```ts
test('checks cart total updates correctly', async ({ page }) => {
  await page.getByRole('button', { name: 'Add to Cart' }).click();
  const total = await page.getByTestId('cart-total').textContent(); // <- breakpoint here
  expect(total).toBe('$49.99');
});
```

With a real breakpoint, you get full IDE debugging — inspect variables, step into/over function calls, and evaluate expressions in the debug console, while the actual browser stays open and interactive.

---

## Quick summary

| Tool | One-line real-world use |
|---|---|
| Trace Viewer | Replay exactly what happened in a failed CI run, no re-run needed |
| Playwright Inspector (`PWDEBUG=1`) | Step through a flaky test action-by-action with a live locator picker |
| Codegen | Record real clicks/typing to scaffold a new test fast |
| VS Code extension | Run/debug individual tests from the editor gutter, no terminal needed |
| `--ui` mode | Interactive dashboard: filter, watch-mode re-runs, per-step DOM snapshots |
| Breakpoints (`page.pause()` / IDE) | Freeze execution mid-test to inspect the live browser or step through code |
