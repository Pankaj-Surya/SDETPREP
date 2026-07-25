# Playwright — API Testing (`APIRequestContext`) — Notes with Real-Time Examples

Reference: https://playwright.dev/docs/api-testing, https://playwright.dev/docs/api/class-apirequestcontext

Simple language, but in-depth on *how it works under the hood* and *why* you'd use it. All examples in TypeScript. 2–3 real-time examples per topic.

---

## What is `APIRequestContext`, really?

It's Playwright's built-in HTTP client — like Postman or `axios`, but living inside your test framework. It lets you call REST APIs directly, **without opening a browser at all**. Every test also gets a ready-made one for free via the `request` fixture, already wired up with your config's `baseURL` and headers.

```ts
import { test, expect } from '@playwright/test';

test('get list of users', async ({ request }) => {
  const response = await request.get('/api/users');
  expect(response.ok()).toBeTruthy();
});
```

---

## 1. `request.get` / `post` / `put` / `patch` / `delete`

**In simple words:** These are just the HTTP verbs — the same actions any API client uses. `get` reads data, `post` creates, `put`/`patch` update, `delete` removes.

**How it works under the hood:** Each of these is really a shortcut for the more general `request.fetch(url, { method: ... })`. They return an `APIResponse` object with `.status()`, `.ok()`, `.json()`, `.text()`, `.headers()`.

### Example 1 — Reading data (GET) to check a product exists before testing the UI

```ts
test('product exists in catalog', async ({ request }) => {
  const response = await request.get('/api/products/101');
  expect(response.status()).toBe(200);
});
```

### Example 2 — Creating data (POST) to set up test state, instead of clicking through the UI

```ts
test('create a new task via API', async ({ request }) => {
  const response = await request.post('/api/tasks', {
    data: { title: 'Buy groceries', completed: false },
  });
  expect(response.status()).toBe(201);
});
```

### Example 3 — Updating (PATCH) and cleaning up (DELETE) in the same test, a common "setup → verify → teardown" pattern

```ts
test('update and delete a task', async ({ request }) => {
  const created = await request.post('/api/tasks', { data: { title: 'Draft note' } });
  const { id } = await created.json();

  const updated = await request.patch(`/api/tasks/${id}`, { data: { completed: true } });
  expect(updated.ok()).toBeTruthy();

  const deleted = await request.delete(`/api/tasks/${id}`);
  expect(deleted.status()).toBe(204);
});
```

---

## 2. `request.newContext` (baseURL, headers, auth)

**In simple words:** The `request` fixture is convenient, but sometimes you need your **own separate** API client — for example, one that talks to a different service, or one you set up once and reuse across many tests. `playwright.request.newContext()` creates that standalone client.

**How it works under the hood:** Options passed here (`baseURL`, `extraHTTPHeaders`, `httpCredentials`, `storageState`) apply to **every request** made through that context, so you don't repeat them each time. It's independent of any browser — no page needed at all. Always call `.dispose()` when done, to release the connection.

### Example 1 — A dedicated context for a third-party API with its own base URL and auth header

```ts
let githubApi: APIRequestContext;

test.beforeAll(async ({ playwright }) => {
  githubApi = await playwright.request.newContext({
    baseURL: 'https://api.github.com',
    extraHTTPHeaders: {
      Accept: 'application/vnd.github.v3+json',
      Authorization: `token ${process.env.GITHUB_TOKEN}`,
    },
  });
});

test.afterAll(async () => {
  await githubApi.dispose();
});

test('repo exists', async () => {
  const response = await githubApi.get('/repos/microsoft/playwright');
  expect(response.ok()).toBeTruthy();
});
```

### Example 2 — Setting up test data in `beforeAll` for an entire test file (create once, use in many tests, delete once at the end)

```ts
let apiContext: APIRequestContext;
let repoName: string;

test.beforeAll(async ({ playwright }) => {
  apiContext = await playwright.request.newContext({ baseURL: 'https://api.example.com' });
  const res = await apiContext.post('/repos', { data: { name: 'test-repo-1' } });
  repoName = (await res.json()).name;
});

test.afterAll(async () => {
  await apiContext.delete(`/repos/${repoName}`);
  await apiContext.dispose();
});
```

### Example 3 — `httpCredentials` for an API protected by HTTP Basic auth (common on staging/internal environments)

```ts
const apiContext = await playwright.request.newContext({
  baseURL: 'https://staging-api.example.com',
  httpCredentials: { username: 'qa-user', password: 'qa-pass' },
});
```

---

## 3. JSON body / form data

**In simple words:** Most APIs expect either a **JSON body** (typical for modern REST APIs) or **form-encoded data** (`application/x-www-form-urlencoded`, common in older/legacy endpoints and some login forms).

**How it works under the hood:** Pass a plain JS object to `data` — Playwright auto-serializes it to JSON and sets `Content-Type: application/json` for you. For form-urlencoded data, pass a plain object to the `form` option instead, and Playwright encodes it the traditional HTML-form way.

### Example 1 — JSON body for creating a resource

```ts
test('create user with JSON body', async ({ request }) => {
  const response = await request.post('/api/users', {
    data: { name: 'Asha Rao', email: 'asha@example.com', role: 'tester' },
  });
  expect(response.status()).toBe(201);
});
```

### Example 2 — Form data for a legacy login endpoint expecting `application/x-www-form-urlencoded`

```ts
test('login via form-encoded endpoint', async ({ request }) => {
  const response = await request.post('/login', {
    form: { username: 'asha', password: 'Secret123' },
  });
  expect(response.ok()).toBeTruthy();
});
```

### Example 3 — Sending query params alongside a JSON body (common for search/filter APIs)

```ts
test('search with query params', async ({ request }) => {
  const response = await request.get('/api/search', {
    params: { q: 'laptop', page: 1, limit: 20 },
  });
  const results = await response.json();
  expect(results.items.length).toBeGreaterThan(0);
});
```

---

## 4. File upload via API

**In simple words:** Uploading a file to an API (like uploading a profile picture, resume, or CSV) uses `multipart/form-data` — the same format an HTML `<input type="file">` form uses.

**How it works under the hood:** Pass a `multipart` object; Playwright builds the multipart body and sets the right `Content-Type` (including the boundary) automatically. You give each file a `name`, `mimeType`, and `buffer` (raw file bytes).

### Example 1 — Uploading a profile picture

```ts
import fs from 'fs';

test('upload profile picture via API', async ({ request }) => {
  const response = await request.post('/api/profile/avatar', {
    multipart: {
      file: {
        name: 'avatar.png',
        mimeType: 'image/png',
        buffer: fs.readFileSync('test-assets/avatar.png'),
      },
    },
  });
  expect(response.ok()).toBeTruthy();
});
```

### Example 2 — Uploading a file **plus** other form fields together (common for "attach document with metadata" flows)

```ts
test('upload document with metadata', async ({ request }) => {
  const response = await request.post('/api/documents', {
    multipart: {
      title: 'Q3 Report',
      category: 'finance',
      file: {
        name: 'q3-report.pdf',
        mimeType: 'application/pdf',
        buffer: fs.readFileSync('test-assets/q3-report.pdf'),
      },
    },
  });
  expect(response.status()).toBe(201);
});
```

### Example 3 — Using an uploaded file's ID to verify it shows up correctly in the UI (bridges API + UI)

```ts
test('uploaded resume appears in UI', async ({ request, page }) => {
  const upload = await request.post('/api/resumes', {
    multipart: { file: { name: 'resume.pdf', mimeType: 'application/pdf', buffer: fs.readFileSync('resume.pdf') } },
  });
  const { id } = await upload.json();

  await page.goto(`https://app.example.com/resumes/${id}`);
  await expect(page.getByText('resume.pdf')).toBeVisible();
});
```

---

## 5. Response assertions

**In simple words:** After calling an API, you check that it responded the way you expected — right status code, right body content, right headers.

**How it works under the hood:** `APIResponse` gives you `.status()`, `.ok()` (true for 2xx), `.headers()`, `.json()`, and `.text()`. You can use these with regular `expect()`, or Playwright's own web-first `expect(response)` matchers like `toBeOK()`.

### Example 1 — Checking status code and response body shape

```ts
test('GET /api/users/1 returns correct user', async ({ request }) => {
  const response = await request.get('/api/users/1');
  expect(response.status()).toBe(200);

  const user = await response.json();
  expect(user).toHaveProperty('id', 1);
  expect(user.email).toContain('@');
});
```

### Example 2 — Using `expect(response).toBeOK()` for a cleaner, self-documenting failure message

```ts
test('health check endpoint is healthy', async ({ request }) => {
  const response = await request.get('/api/health');
  await expect(response).toBeOK(); // fails with the actual status + body if not 2xx
});
```

### Example 3 — Asserting response headers (e.g., checking caching or content-type behavior)

```ts
test('API returns JSON content-type', async ({ request }) => {
  const response = await request.get('/api/products');
  expect(response.headers()['content-type']).toContain('application/json');
});
```

---

## 6. API + UI hybrid tests

**In simple words:** Use the API to quickly **set up** or **verify** data, and the browser only for the part you actually want to test visually. This makes tests much faster and less flaky than doing everything by clicking through the UI.

**How it works under the hood:** `page.request` (or the `request` fixture) shares cookies/session with the browser context, so if you log in or create data via API, the browser page can immediately "see" it as if a real user had done it.

### Example 1 — Create data via API, then verify it shows up in the UI (skips slow "fill a form and click submit" setup)

```ts
test('new issue appears at top of the list', async ({ page, request }) => {
  await request.post('/api/issues', { data: { title: 'Login button broken' } });

  await page.goto('https://app.example.com/issues');
  await expect(page.locator('.issue-row').first()).toHaveText(/Login button broken/);
});
```

### Example 2 — Do an action in the UI, then verify the result via API (faster and more precise than scraping the DOM for confirmation)

```ts
test('deleting a task via UI removes it from backend', async ({ page, request }) => {
  await page.goto('https://app.example.com/tasks');
  await page.getByRole('row', { name: 'Buy groceries' }).getByRole('button', { name: 'Delete' }).click();

  const response = await request.get('/api/tasks?title=Buy groceries');
  const tasks = await response.json();
  expect(tasks.length).toBe(0);
});
```

### Example 3 — Full "hybrid" flow: seed data via API, log in via API (skip login screen), then only test the actual feature in the browser

```ts
test('checkout flow with pre-seeded cart', async ({ page, request }) => {
  await request.post('/api/cart', { data: { productId: 55, quantity: 2 } });

  await page.goto('https://shop.example.com/checkout');
  await expect(page.getByText('2 items in cart')).toBeVisible();

  await page.getByRole('button', { name: 'Place Order' }).click();
  await expect(page.getByText('Order confirmed')).toBeVisible();
});
```

---

## 7. Auth flows (Bearer, Basic, OAuth2, API Key)

**In simple words:** Most real APIs need you to prove who you are before they let you in. Different APIs expect this proof in different ways.

### Bearer token

**How it works:** Send `Authorization: Bearer <token>` on every request. Usually you get the token by logging in via API first.

```ts
test('bearer token auth', async ({ playwright }) => {
  const loginContext = await playwright.request.newContext({ baseURL: 'https://api.example.com' });
  const loginRes = await loginContext.post('/auth/login', {
    data: { email: 'user@example.com', password: 'Secret123' },
  });
  const { token } = await loginRes.json();

  const apiContext = await playwright.request.newContext({
    baseURL: 'https://api.example.com',
    extraHTTPHeaders: { Authorization: `Bearer ${token}` },
  });

  const response = await apiContext.get('/orders');
  expect(response.ok()).toBeTruthy();
});
```

### Basic auth

**How it works:** Username + password get base64-encoded and sent automatically — Playwright handles the encoding when you use `httpCredentials`.

```ts
test('basic auth on staging environment', async ({ playwright }) => {
  const apiContext = await playwright.request.newContext({
    baseURL: 'https://staging.example.com',
    httpCredentials: { username: 'qa', password: 'qa-pass' },
  });

  const response = await apiContext.get('/internal/status');
  expect(response.ok()).toBeTruthy();
});
```

### API Key (custom header)

**How it works:** Many third-party APIs (weather, payments, maps) use a fixed key sent as a custom header or query param — no login step needed.

```ts
test('api key auth', async ({ playwright }) => {
  const apiContext = await playwright.request.newContext({
    baseURL: 'https://api.weather.example.com',
    extraHTTPHeaders: { 'X-API-Key': process.env.WEATHER_API_KEY! },
  });

  const response = await apiContext.get('/forecast?city=Mumbai');
  expect(response.ok()).toBeTruthy();
});
```

### OAuth2 (client credentials flow — common for service-to-service testing)

**How it works:** You first exchange a client ID + secret for an access token at the provider's token endpoint, then use that token as a Bearer token for the actual API calls.

```ts
test('oauth2 client credentials flow', async ({ playwright }) => {
  const authContext = await playwright.request.newContext({ baseURL: 'https://auth.example.com' });

  const tokenRes = await authContext.post('/oauth/token', {
    form: {
      grant_type: 'client_credentials',
      client_id: process.env.CLIENT_ID!,
      client_secret: process.env.CLIENT_SECRET!,
    },
  });
  const { access_token } = await tokenRes.json();

  const apiContext = await playwright.request.newContext({
    baseURL: 'https://api.example.com',
    extraHTTPHeaders: { Authorization: `Bearer ${access_token}` },
  });

  const response = await apiContext.get('/reports');
  expect(response.ok()).toBeTruthy();
});
```

### Bonus — reusing auth across API and UI tests via `storageState`

A very common real pattern: log in once via API, save the session, and reuse it in both API and browser tests — so you never need to fill in a login form in every single test.

```ts
// global-setup.ts (runs once before all tests)
const apiContext = await request.newContext({ baseURL: 'https://app.example.com' });
await apiContext.post('/auth/login', { data: { email: 'user@example.com', password: 'Secret123' } });
await apiContext.storageState({ path: 'storageState.json' });
```

```ts
// playwright.config.ts
export default defineConfig({
  use: { storageState: 'storageState.json' }, // every test starts already logged in
});
```

---

## Quick summary

| Topic | Key idea |
|---|---|
| `get/post/put/patch/delete` | HTTP verbs — set up and clean up test data fast, no UI needed |
| `newContext` | A standalone, reusable API client with its own baseURL/headers/auth |
| JSON / form data | `data` → JSON auto-serialized; `form` → classic urlencoded form |
| File upload | `multipart` object with `name`, `mimeType`, `buffer` |
| Response assertions | `.status()`, `.ok()`, `.json()`, or `expect(response).toBeOK()` |
| API + UI hybrid | API for setup/verification, browser only for the real feature under test |
| Auth flows | Bearer (token in header), Basic (`httpCredentials`), API Key (custom header), OAuth2 (exchange client creds for token), or reuse a saved `storageState` across everything |
