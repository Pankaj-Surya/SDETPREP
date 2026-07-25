# Playwright — Test Organisation — Notes with Real-Time Examples

Reference: https://playwright.dev/docs/api/class-test · https://playwright.dev/docs/test-annotations · https://playwright.dev/docs/test-parallel · https://playwright.dev/docs/api/class-testconfig

Simple language, real test-automation scenarios, all in TypeScript.

---

## 1. `test()` / `test.describe()` / `test.describe.parallel()`

**What they do:** `test()` defines one test case. `test.describe()` groups related tests (shared setup, shared reporting section). `test.describe.parallel()` runs the tests **inside that group** concurrently across workers, instead of one after another.

**Real-time example — grouping all "checkout" tests together, and running independent product-filter tests in parallel since they don't share state:**

```ts
test.describe('Checkout flow', () => {
  test('shows order summary before payment', async ({ page }) => { /* ... */ });
  test('applies a valid promo code', async ({ page }) => { /* ... */ });
  test('rejects an expired promo code', async ({ page }) => { /* ... */ });
});

test.describe.parallel('Product filters', () => {
  test('filter by price', async ({ page }) => { /* ... */ });
  test('filter by brand', async ({ page }) => { /* ... */ });
  test('filter by rating', async ({ page }) => { /* ... */ });
});
```

> Modern preferred syntax: `test.describe.configure({ mode: 'parallel' })` (or `'serial'`/`'default'`) inside a describe block — same effect, and the recommended way going forward.

### Execution Modes

- parallel: Runs individual tests in the block concurrently. Requires more than one worker to actually run simultaneously. Hooks like beforeAll run for each test instance. 

- serial: Runs tests in the block one after another in strict order. If a test fails, all subsequent tests in that block are skipped. Retries restart from the first failed test in sequence. 

- default: Runs tests sequentially in order, but unlike serial, a test failure does not skip subsequent tests. Useful to override a global fullyParallel: true configuration for specific files.

```ts
test.describe('Product filters', () => {
  test.describe.configure({ mode: 'parallel' });
  test('filter by price', async ({ page }) => { /* ... */ });
});
```

---

## 2. `test.beforeAll` / `afterAll` / `beforeEach` / `afterEach`

**What they do:** Setup/teardown hooks. `beforeAll`/`afterAll` run **once** for the whole file or describe block. `beforeEach`/`afterEach` run **around every single test**.

**Real-time example — log in once for a whole file (beforeAll), but reset the cart before every individual test (beforeEach):**

```ts
test.describe('Cart tests', () => {
  let authToken: string;

  test.beforeAll(async ({ request }) => {
    const res = await request.post('/auth/login', { data: { email: 'user@example.com', password: 'pass' } });
    authToken = (await res.json()).token;
  });

  test.beforeEach(async ({ page, request }) => {
    await request.post('/api/cart/clear', { headers: { Authorization: `Bearer ${authToken}` } });
    await page.goto('https://shop.example.com/cart');
  });

  test('empty cart shows a friendly message', async ({ page }) => {
    await expect(page.getByText('Your cart is empty')).toBeVisible();
  });

  test.afterEach(async ({ page }, testInfo) => {
    if (testInfo.status !== testInfo.expectedStatus) {
      await page.screenshot({ path: `failures/${testInfo.title}.png` });
    }
  });

  test.afterAll(async ({ request }) => {
    await request.post('/auth/logout', { headers: { Authorization: `Bearer ${authToken}` } });
  });
});
```

---

## 3. `test.skip` / `test.only` / `test.fixme` / `test.fail`

**What they do:**
- `test.skip()` — don't run this test at all (e.g., feature not built yet, or not relevant on this browser).
- `test.only()` — run **only** this test (or group) while debugging — great locally, should never be committed to CI.
- `test.fixme()` — mark as broken/needs-fixing; Playwright **skips** it but flags it clearly in reports (different from `skip`, which implies "not applicable").
- `test.fail()` — mark a test as **expected to fail**; Playwright runs it and complains if it unexpectedly passes (useful for tracking a known bug until it's fixed).

**Real-time example — a feature only enabled on desktop, a test you're mid-debugging, a known-broken test, and a documented bug:**

```ts
test.skip(({ isMobile }) => isMobile, 'Advanced filters are desktop-only');
test('advanced filters panel works', async ({ page }) => { /* ... */ });

test.only('debugging flaky login redirect', async ({ page }) => { /* ... */ });

test.fixme('date picker crashes on Safari — JIRA-1234', async ({ page }) => { /* ... */ });

test('checkout total rounds incorrectly — known bug BUG-556', async ({ page }) => {
  test.fail();
  await expect(page.getByTestId('total')).toHaveText('$49.99'); // currently shows $49.98
});
```

---

## 4. `test.step` (named steps for reporting)

**What it does:** Wraps part of a test in a named, collapsible section that shows up clearly in the HTML report timeline — makes long tests much easier to read and debug when they fail.

**Real-time example — a checkout test broken into clear phases:**

```ts
test('complete checkout as a returning customer', async ({ page }) => {
  await test.step('Log in', async () => {
    await page.goto('https://shop.example.com/login');
    await page.getByLabel('Email').fill('asha@example.com');
    await page.getByLabel('Password').fill('Secret123');
    await page.getByRole('button', { name: 'Log In' }).click();
  });

  await test.step('Add item to cart', async () => {
    await page.goto('https://shop.example.com/product/101');
    await page.getByRole('button', { name: 'Add to Cart' }).click();
  });

  await test.step('Complete payment', async () => {
    await page.goto('https://shop.example.com/checkout');
    await page.getByRole('button', { name: 'Pay Now' }).click();
    await expect(page.getByText('Order Confirmed')).toBeVisible();
  });
});
```

When this fails, the HTML report shows exactly *which* step failed ("Complete payment") instead of one long undifferentiated test log.

---

## 5. Annotations (tag, slow, retries per test)

**What they do:** Attach metadata to tests — for filtering (`tag`), for giving a legitimately slow test more time (`slow`), or documentation (`annotation`). `retries` isn't a per-test annotation itself, but is commonly set per-project/config, or a test can be told it's flaky via a tag for reporting.

**Real-time example — tagging tests for selective CI runs, marking a heavy report-export test as slow, and documenting a known issue:**

```ts
test('exports a 10,000-row report', { tag: '@slow' }, async ({ page }) => {
  test.slow(); // triples this test's timeout (e.g. 30s -> 90s)
  await page.getByRole('button', { name: 'Export' }).click();
  await expect(page.getByText('Export complete')).toBeVisible({ timeout: 60_000 });
});

test(
  'checkout works on Safari',
  {
    tag: ['@smoke', '@checkout'],
    annotation: { type: 'issue', description: 'https://github.com/org/repo/issues/1234' },
  },
  async ({ page }) => { /* ... */ }
);
```

Run only smoke tests in CI:
```bash
npx playwright test --grep @smoke
```

Retries are usually set globally/per-project in `playwright.config.ts` (retry a flaky test up to N times before marking it failed):
```ts
export default defineConfig({
  retries: process.env.CI ? 2 : 0, // retry twice on CI, never locally
});
```

---

## 6. `test.use` (override config per describe)

**What it does:** Overrides fixture values/options for a whole file or a specific `describe` block — without touching the global config. Must be called at the top level or inside `describe`, **not** inside `beforeEach`/`beforeAll`.

**Real-time example — running one group of tests as a mobile viewport, and another group already logged in via a saved session:**

```ts
test.describe('Mobile navigation menu', () => {
  test.use({ viewport: { width: 390, height: 844 } });

  test('hamburger menu opens the nav drawer', async ({ page }) => {
    await page.goto('https://app.example.com');
    await page.getByRole('button', { name: 'Menu' }).click();
    await expect(page.getByRole('navigation')).toBeVisible();
  });
});

test.describe('Admin settings', () => {
  test.use({ storageState: 'playwright/.auth/admin.json' }); // skip login for this whole group

  test('can change site theme', async ({ page }) => {
    await page.goto('https://app.example.com/admin/settings');
    await expect(page.getByText('Settings')).toBeVisible();
  });
});
```

---

## Quick summary

| Feature | One-line real-world use |
|---|---|
| `test` / `test.describe` / `test.describe.parallel` | Organize tests into logical groups; run independent groups faster |
| `beforeAll`/`afterAll`/`beforeEach`/`afterEach` | Login once per file, reset state before each test, capture failure screenshots |
| `skip` / `only` / `fixme` / `fail` | Skip irrelevant tests, isolate one during debugging, track known bugs |
| `test.step` | Break a long test into named, readable sections in the report |
| Annotations (`tag`, `slow`, `annotation`) | Filter CI runs (`--grep @smoke`), extend timeout for genuinely slow tests, link a bug ticket |
| `test.use` | Different viewport, locale, or pre-authenticated session for one group of tests |
