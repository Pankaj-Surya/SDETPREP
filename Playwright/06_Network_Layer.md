# Playwright — Network Layer (Notes with Real-Time Examples)

Reference: https://playwright.dev/docs/network, https://playwright.dev/docs/mock

Simple language, but each topic goes a bit deeper into *how it actually works* and *when you'd reach for it* in real test automation. All examples in TypeScript.

---

## 1. Request interception — `route`

**In simple words:** Every network request the page makes (API calls, images, scripts...) can be "caught" by Playwright before it reaches the real server. `page.route()` (or `context.route()` for all pages in a context) lets you catch requests matching a URL pattern.

**How it actually works:** Once a route is registered, matching requests **pause** and wait for you to decide what happens next — you must call `route.continue()`, `route.fulfill()`, or `route.abort()`, or the request just hangs forever. Think of it as putting a checkpoint guard in front of the network.

**Real-time example:** You're testing a shopping cart page that calls `/api/cart`. You want to catch that call just to *see* what's being requested, without changing anything yet.

```ts
await page.route('**/api/cart', (route) => {
  console.log('Cart API called:', route.request().url(), route.request().method());
  route.continue(); // let it go through unchanged
});

await page.goto('https://shop.example.com/cart');
```

> Rule to remember: if two routes match the same request, the **most recently registered** one wins. Page-level routes win over context-level routes.

---

## 2. Response mocking — `fulfill`

**In simple words:** Instead of letting the request reach the real server, you hand back a **fake response** yourself — status code, headers, body, all under your control.

**How it actually works:** `route.fulfill({...})` ends the request right there. The browser thinks it got a real server response, but it never left your test.

**Real-time example (the classic one):** You need to test what the UI shows when there are **zero orders** — a state that's hard to set up on a real backend (you'd have to delete all real data). Instead, mock it.

```ts
await page.route('**/api/orders', (route) => {
  route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({ orders: [] }),
  });
});

await page.goto('https://app.example.com/orders');
await expect(page.getByText('No orders yet')).toBeVisible();
```

**Another real scenario — error states:**

```ts
await page.route('**/api/orders', (route) =>
  route.fulfill({ status: 500, body: 'Internal Server Error' })
);

await page.goto('https://app.example.com/orders');
await expect(page.getByRole('alert')).toHaveText(/something went wrong/i);
```

This is huge for testing edge cases (empty states, errors, slow/broken APIs) **without depending on a real backend being in that exact state**.

---

## 3. Request modification — `continue`

**In simple words:** `route.continue()` lets the request go through to the real server, but you can tweak it first — change headers, method, post data, or even the URL.

**How it's different from `fulfill`:** `fulfill` skips the network entirely (fake response). `continue` still hits the real server, just with your modifications applied.

**Real-time example:** Your app requires a special header (`x-feature-flag: new-checkout`) to enable a beta feature on the backend. Instead of changing the app code, you inject the header at the network level for your test.

```ts
await page.route('**/api/**', (route) => {
  const headers = {
    ...route.request().headers(),
    'x-feature-flag': 'new-checkout',
  };
  route.continue({ headers });
});

await page.goto('https://app.example.com/checkout');
```

**Another common use — stripping tracking calls' payloads, or rewriting a query param:**

```ts
await page.route('**/api/search*', (route) => {
  const url = new URL(route.request().url());
  url.searchParams.set('debug', 'true'); // force debug mode on the real API
  route.continue({ url: url.toString() });
});
```

---

## 4. HAR recording and replay

**In simple words:** HAR (HTTP Archive) is a JSON file that stores a full recording of every network request/response during a browser session — like a tape recorder for network traffic. Playwright can **record** one, then **replay** it later so your test doesn't need the real network at all.

**How it actually works:**
- **Record:** pass `recordHar` when creating a context. Playwright writes every request/response to the file as the test runs.
- **Replay:** `page.routeFromHAR()` reads that file and serves matching requests straight from disk — no real network call happens for matched URLs.

**Real-time example — a search page that calls a slow/unreliable third-party API:**

```ts
// Step 1: record once (run this against the real backend)
const context = await browser.newContext({
  recordHar: { path: 'hars/search.har', mode: 'minimal' },
});
const page = await context.newPage();
await page.goto('https://app.example.com/search');
await page.getByLabel('Search').fill('laptop');
await page.getByRole('button', { name: 'Search' }).click();
await context.close(); // HAR file is written here
```

```ts
// Step 2: replay in your actual test (fast, stable, no real network)
test('search results render correctly', async ({ page }) => {
  await page.routeFromHAR('hars/search.har', { url: '**/api/search**', update: false });

  await page.goto('https://app.example.com/search');
  await page.getByLabel('Search').fill('laptop');
  await page.getByRole('button', { name: 'Search' }).click();

  await expect(page.getByText('Results for "laptop"')).toBeVisible();
});
```

**Why this matters in real projects:** unlike hand-written `fulfill()` mocks, a HAR captures the *real* shape of a real response (headers, exact JSON structure), so it's less likely to drift out of sync with the actual API. It's great for third-party APIs (payment gateways, maps, weather) you don't control and don't want to hit in every CI run.

---

## 5. Network throttling (via CDP)

**In simple words:** Simulates a slow internet connection (like 3G) so you can see how your app behaves for real users on bad networks — loading spinners, timeouts, skeleton screens.

**How it actually works:** This isn't a plain Playwright API — it uses the **Chrome DevTools Protocol (CDP)** directly, so it **only works in Chromium-based browsers** (not Firefox/WebKit). You open a `CDPSession` and send the `Network.emulateNetworkConditions` command.

**Real-time example:** You want to confirm a loading skeleton shows up (and doesn't get skipped) when the network is slow — a common bug where the UI "flashes" and only shows content correctly on fast connections.

```ts
const client = await page.context().newCDPSession(page);

await client.send('Network.emulateNetworkConditions', {
  offline: false,
  downloadThroughput: (750 * 1024) / 8, // ~750 kbps, like slow 3G
  uploadThroughput: (250 * 1024) / 8,
  latency: 400, // ms of extra delay per request
});

await page.goto('https://app.example.com/dashboard');
await expect(page.getByTestId('loading-skeleton')).toBeVisible();
await expect(page.getByTestId('dashboard-content')).toBeVisible({ timeout: 15_000 });
```

To turn throttling back off:

```ts
await client.send('Network.emulateNetworkConditions', {
  offline: false,
  latency: 0,
  downloadThroughput: -1,
  uploadThroughput: -1,
});
```

---

## 6. Abort requests

**In simple words:** `route.abort()` blocks a request entirely — the browser gets a network error, as if the request never reached anywhere.

**How it's different from mocking:** `fulfill()` says "here's a fake but successful-looking answer." `abort()` says "this request failed outright" (like a dropped connection, DNS failure, blocked resource).

**Real-time example 1 — testing what happens when a critical API call totally fails (not just returns an error status, but fails at the network level):**

```ts
await page.route('**/api/payment', (route) => route.abort('failed'));

await page.goto('https://shop.example.com/checkout');
await page.getByRole('button', { name: 'Pay Now' }).click();
await expect(page.getByText('Network error. Please try again.')).toBeVisible();
```

**Real-time example 2 — speeding up tests by blocking non-essential resources** (analytics, ads, images) so pages load faster and tests aren't slowed down by things irrelevant to what you're testing:

```ts
await page.route('**/*.{png,jpg,jpeg,gif}', (route) => route.abort());
await page.route('**/analytics/**', (route) => route.abort());

await page.goto('https://app.example.com'); // loads noticeably faster
```

---

## 7. WebSocket interception

**In simple words:** Some apps (live chat, stock tickers, live dashboards) don't use normal HTTP requests — they use a **WebSocket**, a persistent open connection where the server can push messages anytime. `page.routeWebSocket()` lets you intercept and mock that too.

**How it actually works:** By default, a routed WebSocket **does not connect to the real server** — Playwright fakes the whole connection, and you decide what messages "arrive" by calling `ws.send()`. If you want to let it talk to the real server but tweak messages in transit, call `ws.connectToServer()` first.

**Real-time example — fully mocking a live price ticker**, so your test can trigger a specific price update deterministically instead of waiting for a real, unpredictable live feed:

```ts
await page.routeWebSocket('wss://app.example.com/ticker', (ws) => {
  ws.onMessage((message) => {
    if (message === 'subscribe:AAPL') {
      ws.send(JSON.stringify({ symbol: 'AAPL', price: 182.5 }));
    }
  });
});

await page.goto('https://app.example.com/stocks');
await expect(page.getByTestId('AAPL-price')).toHaveText('182.50');
```

**Real-time example — intercepting a real chat connection to inject a test message**, while still letting normal traffic flow to the real server:

```ts
await page.routeWebSocket('/ws/chat', (ws) => {
  const server = ws.connectToServer(); // stays connected to the real backend

  ws.onMessage((message) => {
    if (message === 'ping-test') {
      server.send('inject-test-message'); // forward a custom message to the server
    } else {
      server.send(message); // forward everything else normally
    }
  });
});
```

---

## Quick comparison — which one do I use?

| Goal | Use |
|---|---|
| Just observe requests, don't change anything | `route()` + `route.continue()` |
| Fake a response entirely (no real backend call) | `route.fulfill()` |
| Let the real call happen but tweak headers/URL/body | `route.continue({...})` |
| Reuse a real recorded response across many test runs | HAR record + `routeFromHAR()` |
| Simulate a slow/bad connection | CDP `Network.emulateNetworkConditions` |
| Simulate the request completely failing | `route.abort()` |
| Speed up tests by skipping images/analytics | `route.abort()` on those patterns |
| Mock or intercept a live/streaming connection | `routeWebSocket()` |

---

## Quick summary

- All of this lives at the **network layer** — it sits between your page and the real internet, so you can watch, change, fake, slow down, or block traffic.
- `route()` is the entry point for HTTP; `routeWebSocket()` is the equivalent for WebSockets.
- `fulfill` = fake it completely. `continue` = let it through, maybe tweaked. `abort` = kill it.
- HAR is best when you want *realistic* recorded data reused across many tests.
- CDP throttling only works in Chromium — use it to test real-world slow-network behavior.
- These techniques make tests **fast, reliable, and independent of a real backend's state** — which is why they're so heavily used in real-world test automation.
