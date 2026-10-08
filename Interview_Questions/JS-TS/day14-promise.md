# Day 14 — Promises

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. Promise States — the Mental Model

A Promise is an object representing a value that isn't available yet but will be (or will fail). It's always in exactly one of three states:

- **Pending** — the work hasn't finished yet. Initial state.
- **Fulfilled** — the work finished successfully, and the promise now holds a result value.
- **Rejected** — the work failed, and the promise now holds a reason (usually an `Error`).

Once a promise moves from pending to either fulfilled or rejected, it's **settled**, and it can never change state again. This is worth saying out loud, because it's a real guarantee: a promise resolves once, and that result is permanent.

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) resolve("Data loaded");
    else reject(new Error("Load failed"));
  }, 1000);
});

promise
  .then((result) => console.log(result))       // runs if fulfilled
  .catch((error) => console.error(error))      // runs if rejected
  .finally(() => console.log("Cleanup done")); // runs either way
```

**Interview line:** "A promise is a one-time, settle-once container for a future value. Pending means still working, fulfilled means it succeeded with a value, rejected means it failed with a reason. Once it settles, that outcome is locked in permanently — calling `resolve` or `reject` again after that does nothing."

---

## 2. `.then`, `.catch`, `.finally` — and the Chaining Rule People Get Wrong

Every `.then()` returns a **new promise**, which is what makes chaining possible. What that new promise resolves to depends entirely on what your callback returns.

```js
Promise.resolve(5)
  .then((n) => n * 2)          // returns 10 → next .then receives 10
  .then((n) => n + 1)          // returns 11 → next .then receives 11
  .then((n) => console.log(n)); // 11
```

**The rule worth stating clearly:** if your `.then` callback returns a plain value, the next link receives that value. If it returns **another promise**, the chain waits for that promise to settle and passes along its result — this is what lets you flatten sequential async steps instead of nesting them (recall Day 13's callback hell).

```js
getUser(1)
  .then((user) => getOrders(user.id))   // returns a promise → chain waits for it
  .then((orders) => orders.length)
  .then((count) => console.log(count))
  .catch((error) => console.error("Something failed:", error.message));
```

### The common mistake: forgetting to `return` inside `.then`

```js
// ❌ Broken — the inner promise isn't returned, so the chain doesn't wait for it
getUser(1)
  .then((user) => {
    getOrders(user.id); // missing return!
  })
  .then((orders) => console.log(orders)); // undefined — chain moved on before getOrders finished

// ✅ Fixed
getUser(1)
  .then((user) => {
    return getOrders(user.id);
  })
  .then((orders) => console.log(orders)); // actual orders
```

This is probably the single most common real-world Promise bug in older codebases — it doesn't throw, it just silently passes `undefined` down the chain, which makes it hard to spot.

### How `.catch` and `.finally` actually behave

- **`.catch()`** handles a rejection from anywhere earlier in the chain. After it handles the error, the chain continues as *fulfilled* with whatever `.catch` returned — so a `.catch` that doesn't re-throw effectively "recovers" the chain.
- **`.finally()`** runs regardless of outcome, receives no arguments, and passes the original result or error through unchanged — it's for cleanup, not for transforming values.

```js
fetchData()
  .then((data) => process(data))
  .catch((error) => {
    console.error("Failed:", error.message);
    return []; // recovers — the chain continues with an empty array instead of staying rejected
  })
  .then((result) => console.log("Final:", result)) // runs, either with real data or []
  .finally(() => hideLoadingSpinner());
```

---

## 3. The Four Combinators — `all`, `allSettled`, `race`, `any`

These all take an array of promises and return a single promise — the difference is how they decide when to settle, and with what.

| Method | Fulfills when | Rejects when | Result |
|---|---|---|---|
| `Promise.all` | **All** fulfill | **Any one** rejects (immediately) | Array of values, in input order |
| `Promise.allSettled` | **All** settle (either way) | Never rejects | Array of `{status, value/reason}` objects |
| `Promise.race` | First promise to settle **fulfills** | First promise to settle **rejects** | Whichever settled first |
| `Promise.any` | First promise to **fulfill** | **All** reject | First fulfilled value |

### `Promise.all` — all or nothing, fails fast

```js
const [user, orders, prefs] = await Promise.all([
  getUser(1),
  getOrders(1),
  getPreferences(1),
]);
// All three run in parallel; total time ≈ the slowest one, not the sum of all three
```

**What happens if one rejects?** This is the direct interview question, so be precise: `Promise.all` **rejects immediately with the first rejection's reason**, without waiting for the other promises to finish. Important nuance worth adding: the other promises **don't get cancelled** — they keep running in the background, their results are just ignored by `Promise.all`. JavaScript has no built-in way to cancel a promise once it's started.

```js
try {
  await Promise.all([
    Promise.resolve("ok"),
    Promise.reject(new Error("boom")),
    new Promise((r) => setTimeout(() => r("slow but fine"), 5000)),
  ]);
} catch (error) {
  console.log(error.message); // "boom" — fires immediately, doesn't wait the 5 seconds
}
```

### `Promise.allSettled` — wait for everything, never throws

```js
const results = await Promise.allSettled([
  getUser(1),
  getOrders(999), // might fail
  getPreferences(1),
]);

console.log(results);
// [
//   { status: "fulfilled", value: { id: 1, name: "Alice" } },
//   { status: "rejected",  reason: Error("Orders not found") },
//   { status: "fulfilled", value: { theme: "dark" } },
// ]

const failures = results.filter((r) => r.status === "rejected");
```

### The direct interview answer: `all` vs `allSettled`

> "`Promise.all` is all-or-nothing and fails fast — if any single promise rejects, the whole thing rejects immediately with that reason, and I lose the results of everything else, even ones that succeeded. `Promise.allSettled` always waits for every promise to finish, regardless of outcome, and gives me a per-promise report of what succeeded and what failed — it never rejects itself. I use `all` when the whole operation is meaningless if any part fails, like loading data a page can't render without. I use `allSettled` when I want to run independent operations and see every result, like running a batch of independent checks where one failure shouldn't hide the others."

### `Promise.race` — first to settle wins, success or failure

```js
// Classic use case: adding a timeout to an operation that doesn't have one
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`Timed out after ${ms}ms`)), ms)
  );
  return Promise.race([promise, timeout]);
}

await withTimeout(fetch("/api/slow-endpoint"), 3000); // rejects if fetch takes longer than 3s
```

### `Promise.any` — first *success* wins, ignores failures until all fail

```js
// Try several mirrors/endpoints, take whichever responds successfully first
const data = await Promise.any([
  fetch("https://primary.api.com/data"),
  fetch("https://backup1.api.com/data"),
  fetch("https://backup2.api.com/data"),
]);
// Only rejects (with an AggregateError) if ALL of them fail
```

**The `race` vs `any` distinction worth stating clearly:** `race` settles with whichever promise finishes first, *even if that's a rejection* — a fast failure beats a slower success. `any` ignores rejections and waits specifically for the first success — it only rejects if every single promise rejects.

---

## 4. Test Tie-In: Parallel API Validations and Multiple Element States

### Running independent API validations in parallel

Sequential awaiting is a common, quietly expensive mistake in API test suites — each `await` waits for the previous call to finish before even starting the next one, even when the calls don't depend on each other.

```js
// ❌ Sequential — total time = sum of all three calls
const users = await getUsers();
const products = await getProducts();
const orders = await getOrders();

// ✅ Parallel — total time ≈ the single slowest call
const [users, products, orders] = await Promise.all([
  getUsers(),
  getProducts(),
  getOrders(),
]);

expect(users.status).toBe(200);
expect(products.status).toBe(200);
expect(orders.status).toBe(200);
```

**When to use `allSettled` here instead — and this is the senior-level point:**

```js
// A smoke test hitting many independent endpoints — you want ALL results, not just the first failure
const endpoints = ["/users", "/products", "/orders", "/inventory", "/health"];

const results = await Promise.allSettled(
  endpoints.map((path) => fetch(`${baseUrl}${path}`).then((res) => ({ path, status: res.status })))
);

const failures = results
  .filter((r) => r.status === "rejected" || r.value.status !== 200)
  .map((r) => (r.status === "rejected" ? r.reason.message : `${r.value.path} returned ${r.value.status}`));

expect(failures, `These endpoints failed:\n${failures.join("\n")}`).toEqual([]);
```

**Interview line:** "For a smoke test across several independent endpoints, I use `allSettled` rather than `all` — if three of ten endpoints are down, I want one test run to tell me exactly which three, not just fail on the first rejection and hide the rest. That's far more useful in CI than finding out about failures one at a time across repeated runs."

### Waiting on multiple element states at once

```js
// Wait for several independent conditions on a page to all be true
await Promise.all([
  page.waitForSelector("#header", { state: "visible" }),
  page.waitForSelector("#user-menu", { state: "visible" }),
  page.waitForResponse((res) => res.url().includes("/api/profile") && res.status() === 200),
]);
// Proceeds only once the header, the menu, AND the profile API call have all completed
```

```js
// Using race to handle two possible outcomes — success or an error banner, whichever appears first
const outcome = await Promise.race([
  page.waitForSelector("#success-message").then(() => "success"),
  page.waitForSelector("#error-banner").then(() => "error"),
]);

if (outcome === "error") {
  throw new Error("Form submission showed an error banner instead of success");
}
```

**Interview line:** "`race` is genuinely useful in UI tests when a page can end up in one of two states — success or an error — and I want to react to whichever shows up first, instead of waiting out a full timeout on the one that never appears. It's much faster than waiting for the success element to time out before checking for an error."

### A real gotcha: unhandled rejections hiding inside `Promise.all`

```js
// ⚠️ If you start promises BEFORE passing them to Promise.all, and one rejects early,
// but you haven't attached a handler yet — Node may log an "unhandled rejection" warning
const p1 = getUser(1);       // starts running immediately
const p2 = getOrders(999);   // starts running immediately, might reject fast
await somethingElse();       // while waiting here, p2 may already have rejected with no handler attached
const results = await Promise.all([p1, p2]);
```

Safest habit: create the promises and pass them to `Promise.all` in the same step, without unrelated `await`s in between.

---

## 5. Interview Q&A Script

**Q: Difference between `Promise.all` and `Promise.allSettled`?**
> *(Use the direct answer from section 3: `all` is all-or-nothing and fails fast; `allSettled` always waits for everything and reports each outcome individually, never rejecting. Then give the "when I'd use each" examples — `all` when the whole operation is meaningless if any part fails, `allSettled` when independent results should all be visible regardless of individual failures.)*

**Q: What happens if one promise in `Promise.all` rejects?**
> "The whole `Promise.all` rejects immediately with that first rejection's reason — it doesn't wait for the other promises to finish. Importantly, the other promises aren't cancelled; they keep running in the background and their results are simply ignored, because JavaScript promises have no built-in cancellation mechanism. If I need the results of the ones that did succeed, I'd use `allSettled` instead."

**Q: What's the difference between `Promise.race` and `Promise.any`?**
> "`race` settles with whichever promise finishes first, whether that's a success or a failure — a fast rejection beats a slower success. `any` ignores rejections and waits for the first *successful* result, and only rejects — with an `AggregateError` — if every single promise rejects. I'd use `race` for something like adding a timeout to an operation, and `any` for something like trying several redundant endpoints and taking whichever responds successfully first."

**Q: Can a promise change state after it's settled?**
> "No — once a promise is fulfilled or rejected, it's permanently settled. Calling `resolve` or `reject` again afterward is silently ignored. That's a deliberate guarantee, and it's what makes promises safe to pass around and attach multiple handlers to without worrying the result will change later."

**Q: Why is running independent API calls with sequential `await`s a problem?**
> "Each `await` pauses until that call finishes before the next one even starts, so total time is the sum of all the calls, even when none of them depend on each other. Passing them to `Promise.all` starts them all at once, and total time drops to roughly the slowest single call. In a test suite hitting many endpoints, that adds up to real CI time savings."

---

## 6. One-Page Cheat Sheet

- **States:** pending → fulfilled or rejected (settled, permanent). Settles exactly once; later `resolve`/`reject` calls are ignored.
- **`.then`** returns a *new* promise; returning a value passes it along, returning a promise makes the chain wait for it. **Forgetting to `return` inside `.then` is the #1 silent chaining bug** — it passes `undefined` down the chain without any error.
- **`.catch`** handles earlier rejections and *recovers* the chain if it doesn't re-throw. **`.finally`** runs either way, takes no arguments, and passes the original result/error through untouched — cleanup only.
- **`Promise.all`:** all fulfill → array of values (in input order); any one rejects → rejects immediately with that reason. Other promises are *not cancelled*, just ignored.
- **`Promise.allSettled`:** always waits for everything, never rejects, returns `{status, value|reason}` per promise — best for batches of independent checks where you want every result.
- **`Promise.race`:** first to *settle* (success or failure) wins — good for timeouts, or reacting to whichever of two UI outcomes shows up first.
- **`Promise.any`:** first to *succeed* wins, ignores failures until all fail (then rejects with `AggregateError`) — good for redundant endpoints.
- **Testing rules:** run independent API calls with `Promise.all` instead of sequential `await`s for speed; use `allSettled` for smoke tests across many endpoints so one failure doesn't hide the others; use `race` to handle "either success or error banner" UI outcomes instead of waiting out a timeout; always create and pass promises to `Promise.all` together to avoid unhandled-rejection warnings.
