Here are the full notes on Assertions — I'll share it directly in chat since file creation isn't available in this turn.

# Playwright — Assertions (`expect`) — Notes with Real-Time Examples & Default Timeouts

Reference: https://playwright.dev/docs/test-assertions

## Introduction — the basic idea

`expect(value)` + a matcher = an assertion. Two families exist:

- **Generic matchers** (`toEqual`, `toBeTruthy`...) — check a plain value **once**, no waiting.
- **Web-first matchers** (`toBeVisible`, `toHaveText`...) — check something on the page and **keep retrying** until it passes or times out.

```ts
expect(success).toBeTruthy();                                       // generic — instant
await expect(page.getByTestId('status')).toHaveText('Submitted');   // web-first — retries
```

**Default timeout for all web-first assertions: 5,000 ms.** Set globally via `expect: { timeout: 10_000 }` in config, or per call via `{ timeout: 10_000 }`.

---

## Auto-retrying assertions (web-first) — **default timeout: 5,000 ms** each

**`toBeVisible` / `toBeHidden`** — Confirm a "Payment Successful" banner shows up, and a spinner disappears.
```ts
await expect(page.getByText('Payment Successful')).toBeVisible();
await expect(page.locator('.loading-spinner')).toBeHidden();
```

**`toBeEnabled` / `toBeDisabled`** — A "Submit" button stays disabled until required fields are filled.
```ts
await expect(page.getByRole('button', { name: 'Submit' })).toBeDisabled();
await page.getByLabel('Email').fill('user@example.com');
await page.getByLabel('Password').fill('Secret123');
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
```

**`toHaveText` / `toContainText`** — exact text vs. "contains" text. Good for a cart total shown inside extra wording.
```ts
await expect(page.getByTestId('cart-total')).toContainText('$49.99');
await expect(page.getByTestId('order-status')).toHaveText('Delivered');
```

**`toHaveValue` / `toHaveCount`** — an input's current value, and how many elements matched.
```ts
await page.getByPlaceholder('Search').fill('laptop');
await expect(page.getByPlaceholder('Search')).toHaveValue('laptop');
await expect(page.locator('.product-card')).toHaveCount(5);
```

**`toHaveAttribute` / `toHaveClass`** — a button gets `disabled` while saving; a row gets `class="selected"` after clicking.
```ts
await expect(page.getByRole('button', { name: 'Save' })).toHaveAttribute('aria-busy', 'true');
await page.getByRole('row', { name: 'Order #1023' }).click();
await expect(page.getByRole('row', { name: 'Order #1023' })).toHaveClass(/selected/);
```

**`toHaveURL` / `toHaveTitle`** — after login, confirm URL and tab title changed.
```ts
await page.getByRole('button', { name: 'Login' }).click();
await expect(page).toHaveURL(/\/dashboard/);
await expect(page).toHaveTitle('Dashboard — MyApp');
```

**`toBeChecked` / `toBeFocused`** — "Remember me" checked by default; search box gets keyboard focus.
```ts
await expect(page.getByLabel('Remember me')).toBeChecked();
await page.getByPlaceholder('Search').click();
await expect(page.getByPlaceholder('Search')).toBeFocused();
```

---

## Non-retrying assertions (generic matchers) — **default timeout: none, instant**

Same `expect` library Jest uses (`toBe`, `toEqual`, `toContain`, `toBeTruthy`...). Use for values you already have — API responses, computed numbers — not for anything still changing on the page.

```ts
const response = await request.get('/api/users/1');
const user = await response.json();
expect(user.role).toEqual('admin');
expect(user.tags).toContain('vip');
```

```ts
const totalPrice = 25 + 24.99;
expect(totalPrice).toBeCloseTo(49.99, 2);
```

> Common mistake: `expect(await locator.textContent()).toBe('Submitted')` won't retry. Use `await expect(locator).toHaveText('Submitted')` for anything on the page.

---

## Asymmetric matchers — **default timeout: none, synchronous**

Match **part** of a value, ignoring fields you don't care about (like a generated ID or timestamp).

```ts
const response = await request.post('/api/users', { data: { name: 'Asha' } });
const body = await response.json();
expect(body).toEqual({ id: expect.any(String), name: 'Asha', createdAt: expect.any(String) });
```
```ts
expect(body.message).toEqual(expect.stringContaining('successfully created'));
```

---

## Negating matchers — **default timeout: same as the matcher being negated**

```ts
await expect(page.getByText('Invalid email')).not.toBeVisible();
```
```ts
expect(orderTotal).not.toBe(0);
expect(tags).not.toContain('banned-user');
```

---

## Soft assertions (`expect.soft`) — **default timeout: same as underlying matcher (5,000 ms for web-first)**

Records a failure but doesn't stop the test — useful for catching several mismatches (price, ETA, status) in one run.

```ts
test('order confirmation shows correct details', async ({ page }) => {
  await expect.soft(page.getByTestId('order-total')).toHaveText('$49.99');
  await expect.soft(page.getByTestId('eta')).toHaveText('2 days');
  await expect.soft(page.getByTestId('status')).toHaveText('Confirmed');
  await page.getByRole('link', { name: 'Continue Shopping' }).click(); // test keeps going
});
```
The test is still marked failed overall if any soft assertion failed.

---

## Custom expect message — **no effect on timeout**

```ts
await expect(page.getByTestId('welcome-banner'), 'user should be logged in after submit').toBeVisible();
```
On failure, the report shows your message instead of a generic locator error — much faster to debug in a big suite.

---

## `expect.configure` — **default timeout: 5,000 ms unless overridden**

Build a reusable, pre-configured `expect` for recurring needs (a slow dashboard, always-soft checks).

```ts
import { expect as baseExpect } from '@playwright/test';

export const slowExpect = baseExpect.configure({ timeout: 15_000 });
export const softExpect = baseExpect.configure({ soft: true });

await slowExpect(page.getByTestId('report-table')).toBeVisible();
await softExpect(page.getByTestId('kpi-card')).toHaveText('98%');
```

---

## `expect.poll` — **default timeout: 5,000 ms, default intervals `[100, 250, 500, 1000]` ms**

Turns any synchronous check into a retrying one — great for polling a backend job status.

```ts
await expect.poll(async () => {
  const response = await request.get('/api/reports/55/status');
  const body = await response.json();
  return body.state;
}, { timeout: 15_000, message: 'report should finish processing' }).toBe('completed');
```

---

## `expect.toPass` — **default timeout: 0 (no timeout — bounded only by the test timeout), same default intervals**

Retries a **whole block** of assertions until everything inside passes.

```ts
await expect(async () => {
  const stored = await page.evaluate(() => window.localStorage.getItem('user'));
  expect(stored).toContain('Asha Rao');
}).toPass({ timeout: 10_000 });
```

---

## Custom matchers via `expect.extend` — **no built-in default; you control it**

```ts
import { expect as baseExpect } from '@playwright/test';
import type { Locator } from '@playwright/test';

export const expect = baseExpect.extend({
  async toHaveAmount(locator: Locator, expected: number) {
    await baseExpect(locator).toHaveAttribute('data-amount', String(expected));
    return { pass: true, message: () => `Expected locator not to have amount ${expected}` };
  },
});

await expect(page.getByTestId('cart-total')).toHaveAmount(49.99);
```

---

## Compatibility with `expect` library

Playwright's `expect` is built on the same `expect` npm package Jest uses, so `toBe`, `toEqual`, `toMatchObject`, etc. behave identically — Jest experience carries over directly for non-retrying checks.

```ts
expect(order).toMatchObject({ status: 'confirmed', items: expect.any(Array) });
```

---

## Combine custom matchers from multiple modules

Merge separately-defined matcher sets (e.g., a UI matcher file and an API matcher file) with `mergeExpects`.

```ts
import { mergeExpects } from '@playwright/test';
import { expect as uiExpect } from './ui-matchers';
import { expect as apiExpect } from './api-matchers';

export const expect = mergeExpects(uiExpect, apiExpect);

await expect(page.getByTestId('total')).toHaveAmount(49.99);
await expect(responseBody).toHaveValidSchema();
```

---

## Snapshot assertions

**`toMatchSnapshot` (text) — default timeout: 5,000 ms, single check, no retry.** Compares against a saved file.
```ts
test('invoice text matches snapshot', async ({ page }) => {
  const invoiceText = await page.getByTestId('invoice').innerText();
  expect(invoiceText).toMatchSnapshot('invoice.txt');
});
```

**`toHaveScreenshot` (visual) — default timeout: 5,000 ms, retries internally until visually stable.**
```ts
test('pricing page looks correct', async ({ page }) => {
  await page.goto('https://app.example.com/pricing');
  await expect(page).toHaveScreenshot('pricing-page.png', { maxDiffPixelRatio: 0.02 });
});
```

---

## Advanced: low-level timeouts

You shouldn't normally need these — if tests are flaky, look elsewhere first (locators, waits, real bugs).

| Timeout | Default | Description | Set in config | Override in test |
|---|---|---|---|---|
| Action timeout | no timeout | Timeout for each action | `{ use: { actionTimeout: 10_000 } }` | `locator.click({ timeout: 10_000 })` |
| Navigation timeout | no timeout | Timeout for each navigation | `{ use: { navigationTimeout: 30_000 } }` | `page.goto('/', { timeout: 30_000 })` |
| Global timeout | no timeout | Timeout for the whole test run | `{ globalTimeout: 3_600_000 }` | — |
| `beforeAll`/`afterAll` timeout | 30,000 ms | Timeout for the hook | — | `test.setTimeout(60_000)` |
| Fixture timeout | no timeout | Timeout for a fixture | `{ scope: 'test', timeout: 30_000 }` | — |

```ts
// playwright.config.ts
export default defineConfig({
  use: { actionTimeout: 10_000, navigationTimeout: 30_000 },
});
```
```ts
await page.goto('/reports/annual', { timeout: 60_000 }); // one-off override for a known-slow page
```

---

## Quick reference — every default timeout

| Assertion type | Default timeout |
|---|---|
| Web-first (`toBeVisible`, `toHaveText`...) | 5,000 ms |
| Generic (`toEqual`, `toBeTruthy`...) | none — instant |
| Asymmetric matchers | none — synchronous |
| Negating (`.not`) | same as matcher negated |
| Soft assertions | same as underlying matcher |
| Custom expect message | n/a |
| `expect.configure` | 5,000 ms unless set |
| `expect.poll` | 5,000 ms |
| `expect.toPass` | 0 (no timeout) |
| Custom matchers | whatever you build |
| `toMatchSnapshot` | 5,000 ms, no retry |
| `toHaveScreenshot` | 5,000 ms, retries |
| Action timeout | no timeout |
| Navigation timeout | no timeout |
| Global timeout | no timeout |
| `beforeAll`/`afterAll` | 30,000 ms |
| Fixture timeout | no timeout |
