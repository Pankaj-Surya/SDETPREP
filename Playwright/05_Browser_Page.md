# Playwright — Page & Browser APIs — Notes with Real-Time Examples

Official references:
https://playwright.dev/docs/api/class-page ·
https://playwright.dev/docs/api/class-browser ·
https://playwright.dev/docs/api/class-browsercontext ·
https://playwright.dev/docs/api/class-clock ·
https://playwright.dev/docs/api/class-webstorage

Simple language, real test-automation scenarios, all in TypeScript.

---

## 1. `page.goto` / `page.reload` / `page.goBack`

**Reference:** [`page.goto()`](https://playwright.dev/docs/api/class-page#page-goto), [`page.reload()`](https://playwright.dev/docs/api/class-page#page-reload), [`page.goBack()`](https://playwright.dev/docs/api/class-page#page-go-back) *(there's also [`page.goForward()`](https://playwright.dev/docs/api/class-page#page-go-forward))*

**What it does:** Navigates like a real user typing a URL, hitting refresh, or clicking browser back/forward.

**Real-time example:** On an e-commerce site — open a product page, reload to confirm the price still renders correctly after a refresh, then go back to the listing page.

```ts
await page.goto('https://shop.example.com/product/101');
await page.reload();
await expect(page.getByText('$49.99')).toBeVisible();

await page.goBack();
await expect(page).toHaveURL(/\/products$/);
```

---

## 2. `page.waitForLoadState`

**Reference:** [`page.waitForLoadState()`](https://playwright.dev/docs/api/class-page#page-wait-for-load-state)

**What it does:** Waits for the page to reach `load`, `domcontentloaded`, or `networkidle` (no network activity for a while).

**Real-time example:** After submitting a payment on a slow gateway, wait for the network to settle before checking the confirmation message — avoids checking too early and getting a false failure.

```ts
await page.getByRole('button', { name: 'Pay Now' }).click();
await page.waitForLoadState('networkidle');
await expect(page.getByText('Payment Successful')).toBeVisible();
```

---

## 3. `page.waitForSelector` / `page.waitForFunction`

**Reference:** [`page.waitForSelector()`](https://playwright.dev/docs/api/class-page#page-wait-for-selector) *(legacy — prefer locator-based waits)*, [`page.waitForFunction()`](https://playwright.dev/docs/api/class-page#page-wait-for-function)

**What it does:** `waitForSelector` waits for an element to appear/disappear in the DOM. `waitForFunction` waits until a custom JS expression returns truthy — for things a locator assertion can't check directly (e.g. a global JS flag).

**Real-time example:** A dashboard shows a spinner, then renders a chart once a JS variable (`window.chartLoaded`) is set by a third-party charting library.

```ts
await page.waitForSelector('.spinner', { state: 'hidden' });

await page.waitForFunction(() => (window as any).chartLoaded === true);
await expect(page.locator('#sales-chart')).toBeVisible();
```

> Modern guidance: prefer `expect(locator).toBeVisible()` / `toBeHidden()` for elements — `waitForSelector` is now considered legacy for that use case; `waitForFunction` remains the right tool for non-locator JS conditions.

---

## 4. `page.evaluate` / `page.evaluateHandle`

**Reference:** [`page.evaluate()`](https://playwright.dev/docs/api/class-page#page-evaluate), [`page.evaluateHandle()`](https://playwright.dev/docs/api/class-page#page-evaluate-handle)

**What it does:** Runs JavaScript inside the browser. `evaluate` returns a plain serializable value back to Node. `evaluateHandle` returns a live in-page handle (useful for DOM nodes or large/non-serializable objects you want to keep using in the browser).

**Real-time example:** Reading a feature-flag value only available as a JS global, and the document title.

```ts
const pageTitle = await page.evaluate(() => document.title);
expect(pageTitle).toContain('Dashboard');

const featureFlag = await page.evaluate(() => (window as any).FEATURE_NEW_UI);
expect(featureFlag).toBe(true);
```

---

## 5. `page.exposeFunction`

**Reference:** [`page.exposeFunction()`](https://playwright.dev/docs/api/class-page#page-expose-function) *(also see [`page.exposeBinding()`](https://playwright.dev/docs/api/class-page#page-expose-binding) for a version that also receives page/frame source info)*

**What it does:** Injects a **Node.js function** into the page so browser-side JS can call back into your test code.

**Real-time example:** The app calls `window.trackEvent(name)` for analytics — expose a function to capture every call on the Node side and assert on it.

```ts
const trackedEvents: string[] = [];

await page.exposeFunction('trackEvent', (eventName: string) => {
  trackedEvents.push(eventName);
});

await page.goto('https://shop.example.com');
await page.getByRole('button', { name: 'Add to Cart' }).click();

expect(trackedEvents).toContain('add_to_cart');
```

---

## 6. `page.addInitScript`

**Reference:** [`page.addInitScript()`](https://playwright.dev/docs/api/class-page#page-add-init-script) *(context-wide version: [`browserContext.addInitScript()`](https://playwright.dev/docs/api/class-browsercontext#browser-context-add-init-script))*

**What it does:** Runs a script **before** the page's own scripts, on every navigation — great for setting things up before the app boots.

**Real-time example:** Pre-seed `localStorage` with a fake auth token so the app skips its login screen from the very first paint.

```ts
await page.addInitScript(() => {
  window.localStorage.setItem('authToken', 'test-token-123');
});

await page.goto('https://app.example.com/dashboard');
await expect(page.getByText('Welcome back')).toBeVisible();
```

---

## 7. `page.route` / `page.unroute`

**Reference:** [`page.route()`](https://playwright.dev/docs/api/class-page#page-route), [`page.unroute()`](https://playwright.dev/docs/api/class-page#page-unroute), [`page.unrouteAll()`](https://playwright.dev/docs/api/class-page#page-unroute-all) *(context-wide: [`browserContext.route()`](https://playwright.dev/docs/api/class-browsercontext#browser-context-route))*

**What it does:** Intercepts network requests matching a URL pattern — mock, block, or inspect them without hitting the real backend.

**Real-time example:** Testing how the UI reacts when the "Orders" API fails, without needing the real backend to actually be broken.

```ts
await page.route('**/api/orders', (route) => {
  route.fulfill({
    status: 500,
    contentType: 'application/json',
    body: JSON.stringify({ error: 'Internal Server Error' }),
  });
});

await page.goto('https://app.example.com/orders');
await expect(page.getByText('Something went wrong')).toBeVisible();

await page.unroute('**/api/orders'); // stop mocking, later steps hit the real API
```

---

## 8. `page.on` (events: `request`, `response`, `console`, `dialog`)

**Reference:** [`page.on('request')`](https://playwright.dev/docs/api/class-page#page-event-request), [`page.on('response')`](https://playwright.dev/docs/api/class-page#page-event-response), [`page.on('console')`](https://playwright.dev/docs/api/class-page#page-event-console), [`page.on('dialog')`](https://playwright.dev/docs/api/class-page#page-event-dialog) *(there's also `requestfailed`, `requestfinished`, `pageerror`, `filechooser`, `download`, `websocket`, `popup`, `worker`, `crash`)*

**What it does:** Lets you listen to background browser activity — outgoing requests, incoming responses, `console.log` output, or native dialogs (`alert`/`confirm`/`prompt`).

**Real-time example 1 — catch a silent JS console error during checkout:**

```ts
page.on('console', (msg) => {
  if (msg.type() === 'error') console.log('Browser console error:', msg.text());
});
```

**Real-time example 2 — auto-accept a "Are you sure?" confirm dialog when deleting an item:**

```ts
page.on('dialog', async (dialog) => {
  expect(dialog.message()).toContain('Are you sure you want to delete?');
  await dialog.accept();
});

await page.getByRole('button', { name: 'Delete Item' }).click();
```

**Real-time example 3 — confirm an analytics beacon actually fired:**

```ts
page.on('request', (request) => {
  if (request.url().includes('/analytics')) console.log('Analytics call fired:', request.url());
});
```

---

## 9. Browser context (`newContext`, `newPage`)

**Reference:** [`browser.newContext()`](https://playwright.dev/docs/api/class-browser#browser-new-context), [`browser.newPage()`](https://playwright.dev/docs/api/class-browser#browser-new-page), [`browserContext.newPage()`](https://playwright.dev/docs/api/class-browsercontext#browser-context-new-page)

**What it does:** A **browser context** is an isolated "profile" (own cookies, storage, cache) within the same browser process — like separate incognito windows that don't share login state.

**Real-time example:** Testing a chat app where two users need to talk to each other in one test — each needs its own isolated session.

```ts
const browser = await chromium.launch();

const userAContext = await browser.newContext();
const userAPage = await userAContext.newPage();
await userAPage.goto('https://chat.example.com/login?user=alice');

const userBContext = await browser.newContext();
const userBPage = await userBContext.newPage();
await userBPage.goto('https://chat.example.com/login?user=bob');

await userAPage.getByPlaceholder('Message').fill('Hello Bob!');
await userAPage.getByRole('button', { name: 'Send' }).click();

await expect(userBPage.getByText('Hello Bob!')).toBeVisible();
```

> With `@playwright/test`, each test already gets its own isolated `context`/`page` via fixtures — reach for `browser.newContext()` mainly for multi-user scenarios like this, inside a single test.

---

## 10. Cookies (add, clear, get)

**Reference:** [`browserContext.addCookies()`](https://playwright.dev/docs/api/class-browsercontext#browser-context-add-cookies), [`browserContext.cookies()`](https://playwright.dev/docs/api/class-browsercontext#browser-context-cookies), [`browserContext.clearCookies()`](https://playwright.dev/docs/api/class-browsercontext#browser-context-clear-cookies)

**What it does:** Cookies live on the **context** (not the page). You can inject a session cookie to skip login, inspect current cookies, or wipe them to test logged-out behavior.

**Real-time example:**

```ts
// Skip the login UI by injecting a valid session cookie
await context.addCookies([
  { name: 'session_id', value: 'abc123', domain: 'app.example.com', path: '/' },
]);

await page.goto('https://app.example.com/dashboard');
await expect(page.getByText('Welcome back')).toBeVisible();

// Read current cookies
const cookies = await context.cookies();
console.log(cookies);

// Clear all cookies to confirm logged-out behavior
await context.clearCookies();
await page.reload();
await expect(page).toHaveURL(/\/login$/);
```

---

## 11. LocalStorage / SessionStorage manipulation

**Reference:** [`page.localStorage`](https://playwright.dev/docs/api/class-page#page-local-storage) / [`page.sessionStorage`](https://playwright.dev/docs/api/class-page#page-session-storage) → [`WebStorage` class](https://playwright.dev/docs/api/class-webstorage) *(added in Playwright v1.61)*

**What changed:** Playwright now ships a **native, async Web Storage API** — no more reaching for `page.evaluate()` just to touch `localStorage`/`sessionStorage`.

```ts
await page.goto('https://app.example.com');

// Set a value
await page.localStorage.setItem('theme', 'dark');
await page.reload();
await expect(page.locator('body')).toHaveClass(/dark-theme/);

// Read a value
const theme = await page.localStorage.getItem('theme');
expect(theme).toBe('dark');

// Read everything
const all = await page.localStorage.items();
console.log(all);

// Remove one item, or clear everything
await page.localStorage.removeItem('theme');
await page.sessionStorage.clear();
```

**Real-time example:** An app remembers "dark mode" in `localStorage` — set it directly to test the theme applies on load, without clicking through settings every time.

**Older / cross-version fallback (pre-v1.61, or if you need it before the app's own scripts run):**

```ts
// Fallback via evaluate (works on any Playwright version)
await page.evaluate(() => window.localStorage.setItem('theme', 'dark'));
const theme = await page.evaluate(() => window.localStorage.getItem('theme'));

// Pre-seed BEFORE the app's own JS runs (e.g. before a login-redirect check)
await page.addInitScript(() => window.localStorage.setItem('authToken', 'test-token-123'));
```

> `sessionStorage` isn't part of `context.storageState()` — if you need to persist it across test runs, save/restore it manually via `addInitScript` (as shown above), since Playwright doesn't natively persist session storage into the storage-state file.

---

## Quick summary

| API | One-line real-world use | Reference |
|---|---|---|
| `goto` / `reload` / `goBack` | Navigate like a real user | `class-page` |
| `waitForLoadState` | Wait for page/network to settle | `class-page` |
| `waitForSelector` / `waitForFunction` | Wait for an element or custom JS condition | `class-page` |
| `evaluate` / `evaluateHandle` | Run JS in the browser, read values back | `class-page` |
| `exposeFunction` | Let browser JS call back into test code | `class-page` |
| `addInitScript` | Set things up before the app's own JS runs | `class-page` / `class-browsercontext` |
| `route` / `unroute` | Mock or block network requests | `class-page` / `class-browsercontext` |
| `page.on(...)` | Listen for requests, responses, console logs, dialogs | `class-page` |
| Browser context | Isolated "profile" — great for multi-user tests | `class-browser` |
| Cookies | Skip login by injecting a session cookie | `class-browsercontext` |
| LocalStorage/SessionStorage | Native async API (`page.localStorage`) since v1.61 | `class-webstorage` |
