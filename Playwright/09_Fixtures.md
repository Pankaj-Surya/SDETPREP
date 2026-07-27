# Playwright — Fixtures — Notes with Real-Time Examples

Reference: https://playwright.dev/docs/test-fixtures · https://playwright.dev/docs/api/class-test

Simple language, real test-automation scenarios, all in TypeScript.

---

## What is a fixture, really?

A fixture is a named piece of setup (and matching teardown) that Playwright resolves **lazily** and injects into your test by parameter name — like `{ page }` or `{ request }`. Unlike a `beforeEach` hook, a fixture only runs if a test actually asks for it, and it can return a **value** the test can use.

```ts
test('example', async ({ page }) => {
  // "page" is a fixture — Playwright already created and will tear it down for you
});
```

---

## 1. Built-in fixtures (`page`, `browser`, `context`, `request`)

**What they are:** Playwright Test ships these ready-made — you never construct them yourself.

| Fixture | Scope | What it gives you |
|---|---|---|
| `browser` | worker | one shared browser instance per worker process |
| `context` | test | a fresh, isolated browser context per test |
| `page` | test | a fresh page/tab inside that context |
| `request` | test | an API client (`APIRequestContext`), sharing cookies with `context` |

**Real-time example — using `page` for UI and `request` for setup, in the same test:**

```ts
test('order appears after being created via API', async ({ page, request }) => {
  await request.post('/api/orders', { data: { item: 'Laptop', qty: 1 } });

  await page.goto('https://app.example.com/orders');
  await expect(page.getByText('Laptop')).toBeVisible();
});
```

**Real-time example — using `browser` directly for a multi-user scenario within one test:**

```ts
test('two users can chat with each other', async ({ browser }) => {
  const aliceContext = await browser.newContext();
  const alicePage = await aliceContext.newPage();

  const bobContext = await browser.newContext();
  const bobPage = await bobContext.newPage();

  await alicePage.goto('https://chat.example.com?user=alice');
  await bobPage.goto('https://chat.example.com?user=bob');
  // ...
});
```

---

## 2. Custom fixtures (`test.extend`)

**What it does:** Lets you define your **own** fixtures — page objects, test data, auth tokens — so tests just ask for them by name instead of repeating setup code everywhere.

**Real-time example — a `loginPage` fixture wrapping a Page Object, so every test gets a ready-to-use page object instead of constructing it manually:**

```ts
// fixtures.ts
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

type MyFixtures = {
  loginPage: LoginPage;
};

export const test = base.extend<MyFixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await use(loginPage);          // <-- test runs here, using this value
    // (optional teardown code goes after "use")
  },
});

export { expect } from '@playwright/test';
```

```ts
// login.spec.ts
import { test, expect } from './fixtures';

test('shows error for wrong password', async ({ loginPage }) => {
  await loginPage.login('user@example.com', 'wrong-password');
  await expect(loginPage.errorMessage).toBeVisible();
});
```

---

## 3. Fixture scope (`test` / `worker`)

**What it means:**
- **`test`-scoped** (default) — created fresh for every single test, torn down right after.
- **`worker`-scoped** — created **once per worker process**, shared across every test file that worker runs, torn down only when the worker shuts down.

**Real-time example — a database connection is expensive to open, so make it worker-scoped and reuse it; test data stays test-scoped so tests never leak state into each other:**

```ts
type Fixtures = { testUser: { id: string; name: string } };
type WorkerFixtures = { db: DatabaseClient };

export const test = base.extend<Fixtures, WorkerFixtures>({
  db: [async ({}, use) => {
    const client = await DatabaseClient.connect();
    await use(client);
    await client.disconnect();
  }, { scope: 'worker' }],

  testUser: async ({ db }, use) => {
    const user = await db.createUser({ name: `test-user-${Date.now()}` });
    await use(user);
    await db.deleteUser(user.id); // cleaned up after every test
  },
});
```

> Rule of thumb: if setup is slow (>1-2s) and the resource is read-only/stateless, make it worker-scoped. If it's mutable or test-specific, keep it test-scoped even if slightly slower.

---

## 4. Fixture composition

**What it means:** A fixture can depend on **other fixtures** — Playwright resolves the whole dependency chain automatically, in the right order.

**Real-time example — an `adminPage` fixture that depends on a `loginPage` fixture, which itself depends on the built-in `page` fixture:**

```ts
type Fixtures = {
  loginPage: LoginPage;
  adminPage: AdminDashboardPage;
};

export const test = base.extend<Fixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },

  adminPage: async ({ page, loginPage }, use) => {
    await loginPage.login('admin@example.com', 'admin-pass');
    await use(new AdminDashboardPage(page));
  },
});
```

```ts
test('admin can see the user management panel', async ({ adminPage }) => {
  await adminPage.openUserManagement();
  await expect(adminPage.userTable).toBeVisible();
});
```

The test only asked for `adminPage`, but Playwright quietly resolved `page` → `loginPage` → `adminPage`, in order.

---

## 5. Auto-use fixtures

**What it does:** Runs for **every test automatically**, even when no test explicitly requests it. Perfect for cross-cutting concerns — logging, screenshots on failure, forcing a clean state.

**Real-time example — automatically attach a screenshot whenever a test fails, without adding this to every single test:**

```ts
export const test = base.extend<{ failureScreenshot: void }>({
  failureScreenshot: [async ({ page }, use, testInfo) => {
    await use();
    if (testInfo.status !== testInfo.expectedStatus) {
      const screenshot = await page.screenshot();
      await testInfo.attach('failure-screenshot', { body: screenshot, contentType: 'image/png' });
    }
  }, { auto: true }],
});
```

**Real-time example — always start every test logged out, unless a test explicitly asks for a logged-in fixture:**

```ts
export const test = base.extend<{ forceLoggedOut: void }>({
  forceLoggedOut: [async ({ context }, use) => {
    await context.clearCookies();
    await use();
  }, { auto: true }],
});
```

---

## 6. Shared fixtures across files

**What it means:** For large projects, different fixture files (e.g., one for auth, one for API clients, one for page objects) can be merged into a **single** `test` object using `mergeTests` — so any spec file only needs one import.

**Real-time example:**

```ts
// auth.fixtures.ts
export const authTest = base.extend<{ authToken: string }>({
  authToken: async ({ request }, use) => {
    const res = await request.post('/auth/login', { data: { email: 'user@example.com', password: 'pass' } });
    await use((await res.json()).token);
  },
});
```

```ts
// pages.fixtures.ts
export const pageTest = base.extend<{ dashboardPage: DashboardPage }>({
  dashboardPage: async ({ page }, use) => {
    await use(new DashboardPage(page));
  },
});
```

```ts
// fixtures.ts — combine both into one, so every spec file imports from here
import { mergeTests } from '@playwright/test';
import { authTest } from './auth.fixtures';
import { pageTest } from './pages.fixtures';

export const test = mergeTests(authTest, pageTest);
export { expect } from '@playwright/test';
```

```ts
// dashboard.spec.ts
import { test, expect } from './fixtures';

test('dashboard shows the logged-in user name', async ({ authToken, dashboardPage }) => {
  await dashboardPage.gotoWithToken(authToken);
  await expect(dashboardPage.userName).toBeVisible();
});
```

---

## Quick summary

| Concept | One-line real-world use |
|---|---|
| Built-in fixtures | `page`/`context`/`browser`/`request` are already there — no setup needed |
| Custom fixtures | Wrap page objects/data/tokens so tests just ask for them by name |
| Fixture scope | `worker` for expensive shared resources (DB, auth token); `test` for isolated per-test data |
| Composition | Fixtures can depend on other fixtures — Playwright resolves the chain automatically |
| Auto-use fixtures | Runs for every test without being requested — logging, screenshots, forced clean state |
| Shared across files | `mergeTests` combines fixture modules into one `test` export for the whole project |
