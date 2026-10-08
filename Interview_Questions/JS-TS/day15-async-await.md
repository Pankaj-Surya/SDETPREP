# Day 15 — Async/Await

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. What `async/await` Actually Is

It's syntax on top of Promises, not a separate mechanism. Two rules cover almost everything:

1. An `async` function **always returns a Promise** — even if you return a plain value, it gets wrapped.
2. `await` pauses *that function* until the promise settles, then gives you the fulfilled value (or throws the rejection reason). It does **not** block the whole program — other code keeps running while that function waits.

```js
async function getNumber() {
  return 42; // automatically wrapped: this returns Promise<42>, not 42
}

getNumber().then((n) => console.log(n)); // 42

async function main() {
  const n = await getNumber(); // unwraps the promise
  console.log(n); // 42
}
```

**Interview line:** "`await` pauses only the function it's in, not the event loop. While that function waits, JS is free to run other code — the rest of the function resumes later as a microtask once the promise settles. That's why it reads like blocking code but isn't actually blocking anything."

You can only use `await` inside an `async` function — or at the top level of an ES module (Day 10's top-level `await`).

---

## 2. Error Handling with `try/catch`

Since `await` turns a rejection into a thrown error at that line, ordinary `try/catch` works.

```js
async function loadUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error("Could not load user:", error.message);
    throw error; // re-throw if the caller needs to know — don't silently swallow it
  } finally {
    console.log("loadUser finished"); // cleanup, runs either way
  }
}
```

Two points worth making clearly:

**`fetch` only rejects on network failure, not on a 404 or 500.** A 500 response still resolves successfully — you have to check `response.ok` yourself, as above. This catches people constantly in API tests: the call "worked," the test got a response, and the failure only shows up later as a confusing assertion error.

**`return await` inside `try` matters; outside, it doesn't.**

```js
async function a() {
  try {
    return fetchData(); // ❌ the promise is returned without being awaited,
                        // so a rejection escapes the catch block entirely
  } catch (e) {
    console.log("never runs for fetchData's rejection");
  }
}

async function b() {
  try {
    return await fetchData(); // ✅ rejection is thrown here, inside the try — catch works
  } catch (e) {
    console.log("caught");
  }
}
```

### The "async function with no error handling" trap

```js
async function risky() {
  throw new Error("boom");
}

risky(); // no await, no .catch — becomes an unhandled promise rejection
```

Calling an async function without awaiting it or attaching a `.catch()` means errors disappear into an unhandled rejection, which may only show up as a warning — or crash Node depending on version.

---

## 3. Sequential vs Parallel Awaits

This is where most real performance mistakes happen.

```js
// Sequential — each call waits for the previous to finish
async function sequential() {
  const users = await getUsers();       // 1s
  const products = await getProducts(); // 1s
  const orders = await getOrders();     // 1s
  // total ≈ 3 seconds
}

// Parallel — all three start at once
async function parallel() {
  const [users, products, orders] = await Promise.all([
    getUsers(),
    getProducts(),
    getOrders(),
  ]);
  // total ≈ 1 second (the slowest single call)
}
```

**When sequential is *correct*:** when one call depends on the previous result.

```js
const user = await getUser(id);
const orders = await getOrders(user.id); // genuinely needs user first — must be sequential
```

**A subtle version of the same mistake — starting promises first, awaiting later:**

```js
// Also parallel, and sometimes clearer when you need the results at different points
const usersPromise = getUsers();       // starts immediately
const productsPromise = getProducts(); // starts immediately

const users = await usersPromise;      // wait only when you actually need it
const products = await productsPromise;
```

**Interview line:** "The question I ask myself is whether the calls depend on each other. If they don't, sequential `await`s are just leaving time on the table — I'd start them together and await the combined result. Sequential is correct only when a later call genuinely needs an earlier call's output."

---

## 4. Why Awaiting in a Loop Can Be a Performance Problem

Direct interview question — and the answer is "it depends on whether the iterations are independent."

```js
// ❌ Sequential by accident — 100 users × 200ms each ≈ 20 seconds
async function fetchAllUsers(ids) {
  const users = [];
  for (const id of ids) {
    const user = await getUser(id); // each iteration waits for the previous one to finish
    users.push(user);
  }
  return users;
}

// ✅ Parallel — all requests in flight at once ≈ 200ms total (plus whatever the server can handle)
async function fetchAllUsersFast(ids) {
  return Promise.all(ids.map((id) => getUser(id)));
}
```

**Why the first version is slow:** `await` inside a `for` loop pauses the whole loop on every iteration, so independent requests that could overlap run strictly one after another.

**The follow-up a strong candidate raises unprompted — parallel isn't always the right answer:**

1. **Order or dependency matters.** If iteration 2 depends on iteration 1's result, sequential is required.
2. **Firing 1,000 requests at once can overwhelm a server or hit rate limits.** Unbounded `Promise.all` over a huge list can get you throttled or banned, or exhaust sockets.
3. **Sometimes you deliberately want sequential** — like creating test data where order affects IDs, or UI steps that must happen in order.

A middle ground — controlled concurrency, processing in batches:

```js
async function fetchInBatches(ids, batchSize = 10) {
  const results = [];
  for (let i = 0; i < ids.length; i += batchSize) {
    const batch = ids.slice(i, i + batchSize);
    const batchResults = await Promise.all(batch.map((id) => getUser(id)));
    results.push(...batchResults);
  }
  return results;
}
```

### A classic gotcha: `forEach` with `await` doesn't do what it looks like

```js
// ❌ Does NOT wait — forEach ignores the promises returned by async callbacks
async function broken(ids) {
  ids.forEach(async (id) => {
    await processItem(id);
  });
  console.log("done"); // prints BEFORE any processing has finished
}

// ✅ Sequential: for...of
for (const id of ids) {
  await processItem(id);
}

// ✅ Parallel: map + Promise.all
await Promise.all(ids.map((id) => processItem(id)));
```

`forEach` calls your callback and throws away whatever it returns, including the promise — so nothing waits for it. Same applies to `filter`, and to `reduce` unless you handle it deliberately.

---

## 5. Convert This Promise Chain to `async/await` (live-coding question)

```js
// Original
function getUserSummary(userId) {
  return getUser(userId)
    .then((user) => {
      return getOrders(user.id).then((orders) => {
        return { name: user.name, orderCount: orders.length };
      });
    })
    .catch((error) => {
      console.error("Failed:", error.message);
      return null;
    })
    .finally(() => {
      console.log("Lookup complete");
    });
}
```

```js
// Converted
async function getUserSummary(userId) {
  try {
    const user = await getUser(userId);
    const orders = await getOrders(user.id);
    return { name: user.name, orderCount: orders.length };
  } catch (error) {
    console.error("Failed:", error.message);
    return null;
  } finally {
    console.log("Lookup complete");
  }
}
```

**How to approach it out loud:**

1. Mark the function `async`.
2. Each `.then(x => ...)` becomes `const x = await ...` on its own line — and the nesting disappears, since each `await` just continues on the next line.
3. `.catch` becomes `catch`, `.finally` becomes `finally`.
4. Call out the behavior that's preserved: it still returns a Promise (now resolving to the summary or `null`), and the callers don't need to change.

**Then raise the improvement if the chain allows it:** "If `getOrders` didn't actually depend on `getUser`, I'd start both together with `Promise.all` instead of awaiting sequentially." Mentioning this unprompted shows you're thinking about performance, not just syntax.

---

## 6. Test Tie-In: This *Is* Playwright Syntax

Every Playwright action returns a Promise, so every one needs `await`. Knowing this cold is non-negotiable.

```js
import { test, expect } from "@playwright/test";

test("user can log in", async ({ page }) => {
  await page.goto("https://app.com/login");
  await page.fill("#username", "alice@test.com");
  await page.fill("#password", "secret123");
  await page.click("#submit");

  await expect(page.locator("#welcome")).toBeVisible();
  await expect(page.locator("#welcome")).toHaveText("Welcome, Alice");
});
```

### The #1 Playwright bug: a missing `await`

```js
test("broken login", async ({ page }) => {
  page.goto("https://app.com/login");      // ❌ no await — test races ahead before navigation finishes
  page.fill("#username", "alice");         // ❌ may run before the page even loads
  expect(page.locator("#welcome")).toBeVisible(); // ❌ creates a promise, never awaited — assertion never actually checks anything
});
```

**Why this one is nasty:** the test can *pass* without actually asserting anything. `expect(...).toBeVisible()` without `await` returns a Promise that nobody waits for — if it would have failed, the failure happens after the test has already finished, or becomes an unhandled rejection. Tests that mysteriously pass, or fail randomly in CI, are very often this.

**How to catch it:** the ESLint rule `@typescript-eslint/no-floating-promises` (or `no-floating-promises` equivalents) flags any Promise that isn't awaited, returned, or handled. Worth mentioning in an interview — it shows you prevent the bug systematically, not just by being careful.

### Parallel awaits in Playwright

```js
// Independent checks on a page — run together
await Promise.all([
  expect(page.locator("#header")).toBeVisible(),
  expect(page.locator("#footer")).toBeVisible(),
  expect(page.locator("#nav")).toBeVisible(),
]);
```

```js
// Classic Playwright pattern: start waiting BEFORE the action that triggers it
const [response] = await Promise.all([
  page.waitForResponse((res) => res.url().includes("/api/login") && res.status() === 200),
  page.click("#submit"), // triggers the request
]);
// If you clicked first and THEN started waiting, a fast response could arrive before you were listening
```

**Interview line:** "The `Promise.all` pattern for `waitForResponse` plus the click is one I use all the time. If you click first and only afterward start waiting for the network response, you can miss it on a fast server — the listener has to be set up *before* the action that triggers the event. `Promise.all` starts both together, so there's no gap."

### Auto-waiting: `await` isn't the same as "waits until ready"

```js
await page.click("#submit"); // Playwright auto-waits for the element to be visible, enabled, stable
await expect(page.locator("#result")).toBeVisible(); // web-first assertion — retries until true or timeout
```

Worth distinguishing: `await` here means "wait for this Promise to complete," and the Promise itself has auto-waiting logic inside it. That's why the web-first assertion `await expect(locator).toBeVisible()` is preferred over manually reading a value and asserting on it:

```js
// ❌ Reads once, no retry — flaky
const text = await page.locator("#result").textContent();
expect(text).toBe("Done");

// ✅ Retries until it matches or times out
await expect(page.locator("#result")).toHaveText("Done");
```

This connects directly back to Day 12: no hard `sleep()`, and no one-shot reads of values that may not be ready — let the framework poll for the real condition.

---

## 7. Interview Q&A Script

**Q: Why can awaiting in a loop be a performance problem?**
> "Because `await` pauses the loop on every iteration, so independent operations that could overlap run strictly one after another — a hundred 200ms requests take around twenty seconds instead of about two hundred milliseconds. If the iterations don't depend on each other, I'd map them into an array of promises and use `Promise.all`. That said, I wouldn't blindly fire thousands at once — that can hit rate limits or overwhelm a server — so for large lists I'd process in batches. And if the order or dependencies genuinely matter, sequential is the correct choice."

**Q: Convert this promise chain to `async/await`.**
> *(Follow the four steps from section 5 — mark `async`, turn each `.then` into an `await` line, `.catch`→`catch`, `.finally`→`finally` — then mention the parallel improvement if the calls turn out to be independent.)*

**Q: Does `await` block the thread?**
> "No. It pauses only the async function it's inside. The rest of the program, and the event loop, keep running. When the awaited promise settles, the function resumes as a microtask. It reads like blocking code, but nothing is actually blocked."

**Q: What's wrong with `array.forEach(async (x) => { await ... })`?**
> "`forEach` ignores the return value of its callback, so the promises the async callbacks return are thrown away — nothing waits for them. Code after the `forEach` runs before the work is done, and errors become unhandled rejections. I'd use `for...of` with `await` for sequential work, or `Promise.all` with `map` for parallel work."

**Q: What happens in Playwright if you forget `await` on an assertion?**
> "The `expect(...)` call returns a Promise that nobody waits on, so the test can finish — and pass — before the assertion is ever evaluated. If it would have failed, the failure shows up too late or as an unhandled rejection. It's one of the most common causes of tests that pass when they shouldn't, which is why I enable the `no-floating-promises` lint rule to catch it automatically."

**Q: What does `return await` do inside a `try` block versus just `return`?**
> "Inside a `try`, `return await promise` makes the rejection get thrown within the `try` block, so the `catch` handles it. A plain `return promise` hands the promise straight to the caller without awaiting it, so a rejection bypasses the `catch` entirely. Outside a `try/catch`, the two behave essentially the same."

---

## 8. One-Page Cheat Sheet

- **`async` functions always return a Promise**; `await` unwraps a Promise's value or throws its rejection. It pauses only that function, not the event loop.
- **Errors:** `try/catch/finally` works because `await` converts rejections into thrown errors. Use `return await` inside `try` if you want `catch` to handle the rejection. `fetch` doesn't reject on 404/500 — check `response.ok`.
- **Unawaited async calls** produce unhandled rejections — always `await`, return, or `.catch()` them.
- **Sequential vs parallel:** independent calls → start together with `Promise.all`; dependent calls → sequential `await`s. Total time for sequential is the sum; for parallel it's the slowest one.
- **Await in a loop:** slow when iterations are independent (use `map` + `Promise.all`), correct when order/dependency matters, and batch it when the list is large to avoid rate limits.
- **`forEach` + `async` doesn't wait.** Use `for...of` (sequential) or `Promise.all(arr.map(...))` (parallel).
- **Converting a chain:** mark `async`, each `.then` → `const x = await ...`, `.catch` → `catch`, `.finally` → `finally`; point out parallelism opportunities.
- **Playwright rules:** every action and every `expect(...)` needs `await`. A missing `await` on an assertion can make a test pass without checking anything — use the `no-floating-promises` lint rule. Prefer web-first assertions (`await expect(locator).toHaveText(...)`) over one-shot reads, and use `Promise.all` to start `waitForResponse` *before* the click that triggers it.
