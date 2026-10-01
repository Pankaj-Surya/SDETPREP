# Day 6 — Closures

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. Lexical Scope — Quick Refresher

Lexical scope just means: a function looks for variables based on **where it was written in the code**, not based on who calls it or when.

```js
const env = "staging";

function printEnv() {
  console.log(env); // looks up `env` in the scope where printEnv was DEFINED
}

function runSomewhereElse() {
  const env = "production"; // a different `env`, but irrelevant here
  printEnv(); // still prints "staging" — not affected by this local `env`
}

runSomewhereElse(); // "staging"
```

This is the foundation closures sit on top of, so it's worth stating clearly before jumping into closures themselves.

---

## 2. What Is a Closure — the Real Definition

A closure is a function that **keeps access to variables from the scope it was created in**, even after that outer scope has already finished running.

Here's the point people miss: normally, once a function finishes executing, its local variables get cleaned up. But if you return a function from inside it, and that inner function references those local variables, JS keeps them alive — just for that one inner function.

```js
function createCounter() {
  let count = 0; // this would normally disappear once createCounter() finishes

  return function () {
    count++;
    return count;
  };
}

const counter = createCounter(); // createCounter() already finished executing
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

**How to say this out loud in an interview:** "`createCounter` runs once and returns. Normally `count` would be gone after that. But the function I returned still holds a live reference to `count`, so calling `counter()` later keeps reading and updating the same variable. That's a closure — the inner function 'closed over' `count`."

One thing worth adding, because it often comes up as a follow-up: every time you call `createCounter()` again, you get a **brand-new, independent** `count`:

```js
const counterA = createCounter();
const counterB = createCounter();

console.log(counterA()); // 1
console.log(counterA()); // 2
console.log(counterB()); // 1 — totally separate, its own closure
```

### A real example, not just a counter

Interviewers see the counter example a hundred times a week. Give them something closer to actual code:

```js
function createApiLogger(serviceName) {
  return function (message) {
    console.log(`[${serviceName}] ${message}`);
  };
}

const authLogger = createApiLogger("auth-service");
const paymentLogger = createApiLogger("payment-service");

authLogger("Token validated");      // [auth-service] Token validated
paymentLogger("Charge processed");  // [payment-service] Charge processed
```

Each logger permanently remembers its own `serviceName` — no global state, no passing the name into every single log call. That's closures doing real work, not just a demo.

---

## 3. Private Variables via Closures

Before `#privateField` syntax existed in classes, this was the standard way to get real privacy in JS — and it still shows up a lot, especially outside class-based code.

```js
function createBankAccount(startingBalance) {
  let balance = startingBalance; // nothing outside this function can reach `balance` directly

  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) throw new Error("Insufficient funds");
      balance -= amount;
      return balance;
    },
    getBalance() {
      return balance;
    },
  };
}

const account = createBankAccount(100);
account.deposit(50);
console.log(account.getBalance()); // 150
console.log(account.balance);      // undefined — there's no actual property called `balance`
```

**The key point to say clearly:** there is no `account.balance` at all. The only way in or out is through the methods you deliberately exposed. That's real encapsulation, built entirely with scope — no special syntax required.

---

## 4. The Loop + `setTimeout` Closure Bug — Full Walkthrough

This is the single most common closures question, and the cleanest way to answer it is to walk through it step by step, slowly, instead of just saying "use let."

```js
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100);
}
// Output: 3, 3, 3
```

**Walk through it like this:**

1. `var` doesn't create a new variable per loop iteration — there's only **one** `i` for the entire loop, shared across all three iterations.
2. Each `setTimeout` callback is a closure that holds a reference to that one shared `i` — not a snapshot of its value at that moment, a live reference to the actual variable.
3. The loop finishes running completely — all three iterations, synchronously — before any `setTimeout` callback actually fires. Even with a delay of `0`, `setTimeout` always waits for the current code to finish first.
4. By the time the callbacks run, the loop has already finished and `i` is `3`. Since all three callbacks are looking at the same `i`, they all print `3`.

```js
for (let i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100);
}
// Output: 0, 1, 2
```

**Why `let` actually fixes this** (say this part precisely, it's what separates a good answer from a great one): with `let`, the loop creates a **new `i` for every single iteration** — not one shared variable. So each `setTimeout` callback closes over its own private copy of `i`, frozen at that iteration's value.

### The old fix, before `let` existed

```js
for (var i = 0; i < 3; i++) {
  (function (capturedI) {
    setTimeout(function () {
      console.log(capturedI);
    }, 100);
  })(i);
}
// Output: 0, 1, 2
```

**Worth mentioning if asked "is there another way to fix this":** this IIFE creates a brand-new function scope on every loop iteration, and `capturedI` becomes its own separate variable each time, holding whatever `i` was at that exact moment it was called. `let` just does this automatically now, which is exactly why it mattered so much when it was introduced.

---

## 5. Closures and Memory — a Good Point to Raise Yourself

Since closures keep their outer scope alive, they can quietly hold onto more than you intend. Worth bringing up proactively — it signals you've actually debugged something like this before, not just read about closures.

```js
function setupHandler() {
  const largeDataset = new Array(1_000_000).fill("data");

  document.getElementById("btn").addEventListener("click", function () {
    console.log("clicked");
    // this callback never touches largeDataset, but as long as it exists,
    // the whole outer scope — including largeDataset — stays in memory
  });
}
```

**Interview line:** "A closure keeps its entire surrounding scope alive, not just the specific variables the inner function actually uses. In a long-running app, if you attach a closure as an event listener or a timer and never clean it up, anything large sitting in that same scope stays in memory longer than you'd expect — I've seen this cause real memory growth in long-lived test runners and dashboards."

---

## 6. Test Tie-In: Module Pattern for Page Objects / Test Config

This is where closures stop being a theory question and start being something you actually use every week in automation code — hiding implementation details behind a clean, stable interface.

### Page Object with private selectors

```js
function createLoginPage(driver) {
  // private — test files should never need to know these selectors directly
  const selectors = {
    username: "#username",
    password: "#password",
    submitBtn: "#login-submit",
  };

  async function fillField(selector, value) {
    await driver.fill(selector, value);
  }

  // public API — this is all a test file ever touches
  return {
    async login(username, password) {
      await fillField(selectors.username, username);
      await fillField(selectors.password, password);
      await driver.click(selectors.submitBtn);
    },
    async getErrorMessage() {
      return driver.textContent("#error-message");
    },
  };
}
```

```js
const loginPage = createLoginPage(driver);
await loginPage.login("alice@test.com", "wrongpassword");
const error = await loginPage.getErrorMessage();
expect(error).toBe("Invalid credentials");
```

**Why this matters in practice:** if the login page's HTML changes, you update `selectors` inside `createLoginPage` once. Every test file calling `loginPage.login(...)` keeps working without touching a single test — because they never had direct access to the selectors to begin with. That's the real payoff of closures here: it's not just "hiding variables," it's keeping your tests stable when the app under test changes.

### Test config holding something genuinely sensitive

```js
function createTestConfig(env) {
  let baseUrl;
  let apiKey;

  if (env === "staging") {
    baseUrl = "https://staging.api.com";
    apiKey = process.env.STAGING_KEY;
  } else {
    baseUrl = "https://prod.api.com";
    apiKey = process.env.PROD_KEY;
  }

  return {
    getBaseUrl: () => baseUrl,
    // deliberately no getApiKey() — nothing outside this function can read the key directly
    request: async (path) => {
      return fetch(`${baseUrl}${path}`, {
        headers: { Authorization: `Bearer ${apiKey}` },
      });
    },
  };
}

const config = createTestConfig("staging");
await config.request("/users"); // apiKey is used internally, never exposed anywhere
```

**Interview line:** "This pattern is especially useful for something like an API key in a test config — the tests can make authenticated calls through `config.request(...)`, but there's no `config.apiKey` anyone could accidentally log, print in a failure message, or leak into a report. The closure makes the sensitive value physically unreachable from outside, not just discouraged from being touched."

---

## 7. Interview Q&A Script

**Q: What is a closure? Give a real example.**
> "A closure is a function that keeps access to variables from the scope it was created in, even after that outer function has finished running. Beyond the typical counter example, a practical one is a logger factory — `createApiLogger('auth-service')` returns a function that permanently remembers its own service name through closure, so you don't have to pass that name into every single log call or rely on a global variable."

**Q: Walk through the classic loop + `setTimeout` closure bug.**
> *(Go through the 4 steps in section 4 out loud — one shared `i` with `var`, the loop finishing before any callback runs, all three callbacks pointing at the same final value — then explain precisely why `let` fixes it: a fresh `i` per iteration, not just "let is block-scoped.")*

**Q: How did people create private variables before `#privateField` existed?**
> "Through closures — a factory function returns an object of methods, and the actual state lives as local variables inside that function, never exposed as a property anyone can reach. There's no `object.property` to access directly; the only way in or out is through the methods you deliberately return."

**Q: Can closures cause memory issues?**
> "Yes — a closure keeps its whole enclosing scope alive, not just the variables it actually uses. If you create a closure inside a scope that also holds something large, and that closure sticks around — an event listener that's never removed, a timer that's never cleared — that large data stays in memory longer than it needs to. It's worth watching for in long-running processes like test runners or dashboards."

**Q: Why use the module pattern for page objects instead of just exposing everything?**
> "It keeps implementation details — selectors, internal helper functions, sometimes sensitive values like API keys — private, while exposing a small, stable public API. If the underlying page or config changes, I update it in one place, inside the factory function, and every test using that page object or config keeps working without changes, because they never depended on the private details directly."

---

## 8. One-Page Cheat Sheet

- **Lexical scope:** a function resolves variables based on where it was *written*, not where it's called from.
- **Closure:** a function that keeps a live link to variables from its creation scope, even after that outer function has returned. Each separate call to the outer function creates its own independent closure.
- **Private variables via closure:** return an object of methods from a factory function; keep the real state as local variables with no exposed property — nothing outside can touch it directly.
- **Loop + `setTimeout` bug:** `var` shares one `i` across the whole loop, so every callback sees the final value after the loop ends. `let` creates a fresh `i` per iteration, so each callback gets its own frozen value. The old IIFE trick manually recreated what `let` now does automatically.
- **Memory:** a closure keeps its *entire* surrounding scope alive, not just the variables it uses — watch for this with long-lived closures like event listeners or timers that are never cleaned up.
- **Automation use case:** the module pattern — page objects and test config objects — uses closures to hide selectors, helper logic, and sensitive values like API keys, exposing only a small, stable public interface that keeps tests working even as the underlying implementation changes.
