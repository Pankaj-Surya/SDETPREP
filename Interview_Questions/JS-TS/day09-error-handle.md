# Day 9 — Error Handling

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. `try/catch/finally` — Quick Refresher, With the Parts People Forget

```js
function parseConfig(jsonString) {
  try {
    return JSON.parse(jsonString);
  } catch (error) {
    console.error("Invalid config:", error.message);
    return null;
  } finally {
    console.log("parseConfig attempted"); // runs no matter what — success, failure, even a return above
  }
}
```

The part worth saying out loud: `finally` runs **even if you `return` inside `try` or `catch`**. That trips people up — the return value is already decided, but `finally` still executes before the function actually hands control back to the caller. It's the right place for cleanup — closing a connection, releasing a lock, stopping a loading spinner — regardless of whether the operation succeeded or failed.

```js
function demo() {
  try {
    return "from try";
  } finally {
    console.log("finally still runs"); // this logs before the function actually returns
  }
}
```

One more thing worth mentioning: you can `catch` without binding the error at all if you don't need it (`catch { ... }`), which is a small but genuinely useful ES2019 addition when you're intentionally swallowing an error you don't care to inspect.

---

## 2. `throw` and Error Propagation

`throw` immediately stops normal execution and starts unwinding the call stack, looking for the nearest `catch` that can handle it. If nothing catches it, it crashes the program (sync code) or produces an unhandled rejection (async code).

```js
function withdraw(balance, amount) {
  if (amount > balance) {
    throw new Error("Insufficient funds");
  }
  return balance - amount;
}

function processWithdrawal(account, amount) {
  // no try/catch here — the error just passes through, untouched
  return withdraw(account.balance, amount);
}

try {
  processWithdrawal({ balance: 100 }, 500);
} catch (error) {
  console.log("Caught at the top:", error.message); // "Caught at the top: Insufficient funds"
}
```

**The point to make clearly:** `processWithdrawal` doesn't need its own `try/catch` just because something it calls might throw. An error naturally travels up the call stack until something actually handles it. Wrapping every single function in `try/catch` "just in case" is a common junior habit — it adds noise and often swallows errors in the wrong place, far from where you'd actually want to decide what to do about them.

You can also `throw` anything — not just `Error` objects — but you shouldn't:

```js
throw "something broke"; // works, but terrible — no stack trace, no .message, nothing to work with
throw new Error("something broke"); // always do this instead — real stack trace, consistent shape
```

---

## 3. Designing a Custom Error Class

This comes up constantly, both as a direct interview question and in real framework code. The goal is giving your errors structure — so calling code can tell errors apart programmatically instead of parsing error message strings.

```js
class ApiError extends Error {
  constructor(message, statusCode, endpoint) {
    super(message); // sets this.message — must call this first, same rule as any subclass
    this.name = "ApiError"; // otherwise it would just say "Error" in logs/stack traces
    this.statusCode = statusCode;
    this.endpoint = endpoint;

    // keeps the stack trace clean, pointing to where ApiError was thrown,
    // not to this constructor line itself
    Error.captureStackTrace?.(this, ApiError);
  }
}

function fetchUser(id) {
  if (id < 0) {
    throw new ApiError("Invalid user id", 400, "/users/:id");
  }
  // ... actual fetch logic
}

try {
  fetchUser(-1);
} catch (error) {
  if (error instanceof ApiError) {
    console.log(`${error.statusCode} error at ${error.endpoint}: ${error.message}`);
  } else {
    throw error; // not something we know how to handle — let it keep propagating
  }
}
```

**Say these points clearly if asked to "design a custom error class":**

1. **Extend `Error`, call `super(message)` first.** Same rule as any subclass constructor — you need the parent set up before adding your own fields.
2. **Set `this.name`.** Without it, every custom error still prints as `Error: ...` in logs and stack traces, which defeats the purpose of having a named error type at all.
3. **Attach whatever structured data is useful** — a status code, a field name that failed validation, a resource ID — so calling code can branch on real data (`error.statusCode === 404`) instead of fragile string matching on `error.message`.
4. **Check types with `instanceof`, not message text.** This is the entire point of custom error classes — `if (error instanceof ApiError)` is reliable and refactor-safe; `if (error.message.includes("Invalid"))` breaks the moment someone tweaks the wording.

### A small hierarchy, for when you have several related error types

```js
class AppError extends Error {
  constructor(message) {
    super(message);
    this.name = this.constructor.name; // automatically uses the actual subclass name
  }
}

class ValidationError extends AppError {
  constructor(message, field) {
    super(message);
    this.field = field;
  }
}

class NotFoundError extends AppError {
  constructor(message, resourceId) {
    super(message);
    this.resourceId = resourceId;
  }
}
```

`this.constructor.name` is a nice touch here — `ValidationError` automatically gets `name = "ValidationError"` and `NotFoundError` gets `name = "NotFoundError"`, without repeating `this.name = "..."` in every subclass.

---

## 4. Handling Errors in Async Code vs Sync Code — the Core Interview Question

This is the question to be sharpest on. The short version: **`try/catch` works the same way for `async/await`, but it does NOT catch errors from plain callback-based or unhandled Promise code.**

### Sync code — straightforward

```js
try {
  JSON.parse("not valid json");
} catch (error) {
  console.log("Caught:", error.message); // works, plain and simple
}
```

### `async/await` — `try/catch` works exactly like sync code

```js
async function getUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error(`Request failed: ${response.status}`);
    return await response.json();
  } catch (error) {
    console.log("Caught in async function:", error.message); // works, because `await` effectively "pauses" execution here
  }
}
```

**Why this works:** `await` makes the function pause and wait for the promise to settle, and if it rejects, that rejection is converted into a thrown error right at the `await` line — which `try/catch` can catch, exactly as if it were sync code throwing. This is the main reason `async/await` is genuinely easier to reason about than raw `.then()` chains.

### Plain Promises (`.then()`) — you need `.catch()`, not `try/catch`

```js
// ❌ This does NOT work — try/catch can't see into a .then() callback
try {
  fetch("/api/users").then((res) => res.json());
} catch (error) {
  console.log("This will never run for a rejected fetch");
}

// ✅ Correct — use .catch() on the promise chain itself
fetch("/api/users")
  .then((res) => res.json())
  .catch((error) => console.log("Caught properly:", error.message));
```

### Callback-based async code (older Node-style) — `try/catch` does nothing at all

```js
// ❌ Completely broken — this try/catch is USELESS
try {
  fs.readFile("config.json", (err, data) => {
    if (err) throw err; // this throw happens LATER, inside a different call stack — the outer try/catch is long gone
  });
} catch (error) {
  console.log("This will never catch anything");
}

// ✅ Correct — handle the error INSIDE the callback itself
fs.readFile("config.json", (err, data) => {
  if (err) {
    console.log("Handled correctly, inside the callback:", err.message);
    return;
  }
  // use data
});
```

**The one sentence that answers this interview question well:** "`try/catch` only catches errors that happen synchronously within the same call stack it's wrapped around. `async/await` works with `try/catch` because `await` turns a rejected promise into a thrown error at that exact line, inside the function's own execution. But a `.then()` callback or a Node-style callback runs later, on its own separate turn of the event loop — by the time it throws, the original `try/catch` has already finished and is gone, so it can never catch anything from inside those callbacks. That's why Promises need `.catch()` and callback-style code needs to handle errors directly inside the callback."

### Unhandled rejections — worth mentioning proactively

```js
async function risky() {
  throw new Error("Oops");
}

risky(); // no .catch(), no try/catch around the call — this becomes an UNHANDLED REJECTION

// Good practice: a global safety net, not a replacement for proper handling
process.on("unhandledRejection", (reason) => {
  console.error("Unhandled rejection:", reason);
});
```

**Interview line:** "If an async function throws and nobody's awaiting it inside a `try/catch` or chaining a `.catch()`, you get an unhandled promise rejection — in Node, that can even crash the process depending on version and configuration. I treat a global `unhandledRejection` handler as a safety net for logging and alerting, never as the actual error-handling strategy — every async call that matters should have its own explicit handling."

---

## 5. Test Tie-In: Flaky Element-Not-Found Errors and Custom Assertion Messages

This is where error handling stops being abstract and becomes something you deal with every day running a test suite.

### The problem: a generic "element not found" error tells you almost nothing

```js
// ❌ What you get by default from most frameworks
// Error: Waiting for selector `#submit-button` failed: timeout 5000ms exceeded
```

That message doesn't tell you *why* it wasn't found — slow page load? Wrong selector after a UI change? An unexpected modal blocking it? A genuinely flaky test? You end up re-running locally just to find out, which is wasted time multiplied across however many times that test runs in CI.

### A wrapper that adds real context before the error propagates

```js
class ElementNotFoundError extends Error {
  constructor(selector, context = {}) {
    super(`Element not found: "${selector}"${context.page ? ` on page "${context.page}"` : ""}`);
    this.name = "ElementNotFoundError";
    this.selector = selector;
    this.context = context;
  }
}

async function safeClick(driver, selector, options = {}) {
  try {
    await driver.waitForSelector(selector, { timeout: options.timeout ?? 5000 });
    await driver.click(selector);
  } catch (originalError) {
    // don't just let the generic timeout error bubble up — wrap it with useful context
    throw new ElementNotFoundError(selector, {
      page: options.pageName,
      action: "click",
      originalMessage: originalError.message,
    });
  }
}
```

```js
try {
  await safeClick(driver, "#submit-button", { pageName: "CheckoutPage" });
} catch (error) {
  if (error instanceof ElementNotFoundError) {
    console.log(`Failed on ${error.context.page}: ${error.message}`);
    // now CI logs immediately show which page and which selector, no digging required
  }
}
```

**Interview line:** "A raw timeout error from the framework tells you a selector wasn't found, but not *where* in your test flow or *why* it mattered. Wrapping it in a custom error — attaching the page name, the action being attempted, maybe a screenshot path — means when a test fails in CI at 2am, the failure message alone gives you most of what you'd otherwise have to reproduce the failure locally to find out."

### Custom assertion error messages — making failures self-explanatory

```js
function assertOrderTotal(actual, expected, orderId) {
  if (actual !== expected) {
    throw new Error(
      `Order ${orderId}: expected total ${expected} but got ${actual} ` +
      `(difference: ${(actual - expected).toFixed(2)})`
    );
  }
}

// vs. the generic version most assertion libraries give you by default:
// expect(actual).toBe(expected);
// AssertionError: expected 149.98 to be 150
```

Most test frameworks let you pass a custom message alongside the built-in matcher too — no need to hand-roll everything:

```js
expect(actual, `Order ${orderId} total mismatch`).toBe(expected);
```

**Interview line:** "A generic assertion failure tells you two numbers didn't match. A custom message tells you *which* order, *what* that mismatch likely means, and sometimes the actual magnitude of the difference — which matters a lot, because `149.99` vs `150` is probably a rounding bug, while `0` vs `150` is probably a completely broken calculation. The goal with any custom error or assertion message is always the same: whoever reads this failure in a CI log, without any other context, should understand roughly what went wrong and where to start looking."

### Retrying flaky operations without swallowing the real error

```js
async function retryUntilSuccess(fn, { retries = 3, delayMs = 1000 } = {}) {
  let lastError;
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      console.log(`Attempt ${attempt} failed: ${error.message}`);
      if (attempt < retries) await new Promise((r) => setTimeout(r, delayMs));
    }
  }
  throw lastError; // after exhausting retries, throw the LAST real error — don't invent a new generic one
}

await retryUntilSuccess(() => safeClick(driver, "#submit-button"), { retries: 3 });
```

**Interview line:** "When I retry a flaky step, I always re-throw the *actual* last error after exhausting retries, not a generic 'retries exhausted' message — the real error still has the useful detail, like which selector or which assertion actually failed on the final attempt. I treat retries as a tool for known, understood flakiness — like a slow-loading element — not a blanket fix for test instability. If a test needs retries to pass reliably, that's usually worth investigating on its own, not just papering over."

---

## 6. Interview Q&A Script

**Q: How do you handle errors in async code vs sync code?**
> "For sync code and `async/await`, `try/catch` works the same way — `await` converts a rejected promise into a thrown error right at that line, so `try/catch` catches it naturally. For plain `.then()` chains, you need `.catch()` on the promise instead, since `try/catch` can't see into a callback that runs on a later turn of the event loop. For older Node-style callbacks, you have to check and handle the error directly inside the callback itself — a `try/catch` wrapped around the function call does nothing, because the callback runs after that `try/catch` has already finished executing."

**Q: Design a custom error class.**
> *(Walk through section 3 live: extend `Error`, call `super(message)` first, set `this.name`, attach structured data like a status code, and explain that the entire point is enabling `instanceof` checks instead of fragile string matching on `error.message`.)*

**Q: Why shouldn't you wrap every function in `try/catch`?**
> "Errors naturally propagate up the call stack until something actually handles them, so a function that doesn't know what to do about an error shouldn't catch it — it should let it pass through to somewhere that does. Catching everything everywhere tends to either swallow real errors silently or handle them in a place with no useful context to actually decide what the right recovery is."

**Q: What's the difference between `throw new Error(...)` and just `throw "a string"`?**
> "Throwing a plain string works technically, but you lose the stack trace, `.message`, `.name` — basically everything that makes debugging an error possible later. `Error` objects are the standard shape every tool, logger, and test framework expects, so I always throw real `Error` instances, or custom classes that extend `Error`."

**Q: Why wrap a generic "element not found" error in your own error class in test automation?**
> "A default timeout error just tells you a selector wasn't found — not which page, which test step, or what that likely means. Wrapping it with context — page name, the action being attempted, maybe the original error's message — means a CI failure is diagnosable from the log alone, without needing to reproduce it locally first."

---

## 7. One-Page Cheat Sheet

- **`finally`** always runs — even after a `return` inside `try`/`catch` — the right place for cleanup regardless of success or failure.
- **`throw`** unwinds the call stack looking for the nearest `catch`. Don't wrap every function in `try/catch` "just in case" — let errors propagate to where they can actually be handled meaningfully.
- **Always throw real `Error` objects** (or subclasses), never plain strings — you need the stack trace and consistent shape.
- **Custom error class checklist:** extend `Error`, call `super(message)` first, set `this.name`, attach structured data (status code, field, resource ID), and check types with `instanceof` — never with `error.message` string matching.
- **Sync & `async/await`:** `try/catch` works the same for both, because `await` converts a promise rejection into a thrown error at that line.
- **Plain `.then()` chains:** need `.catch()` — `try/catch` can't see into a `.then()` callback.
- **Node-style callbacks:** handle the error directly inside the callback — a `try/catch` around the function call is a no-op, since the callback runs on a later turn of the event loop after that `try/catch` is already gone.
- **Unhandled rejections** are a safety-net-only concept — a global handler for logging/alerting, never a substitute for handling errors where they actually happen.
- **Test automation rule:** wrap raw framework errors (timeouts, element-not-found) with context — page, action, original message — so CI failures are diagnosable from the log alone. On retries, always re-throw the real last error, not a generic "retries exhausted" message, and treat retries as a fix for known flakiness, not a blanket patch for unstable tests.
