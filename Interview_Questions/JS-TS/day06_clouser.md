# Day 6 — Closures

Everything you need to *understand it*, *explain it out loud*, and *answer follow-ups* in an interview.

---

## 1. Lexical Scope — the Foundation Closures Are Built On

**Lexical scope** means a function's access to variables is determined by *where it's physically written in the code*, not by where or how it's called.

```js
const outerVar = "I'm outside";

function outer() {
  const innerVar = "I'm inside";

  function inner() {
    console.log(outerVar, innerVar);
  }

  inner();
}

outer(); // "I'm outside" "I'm inside"
```

Every function carries a reference to the scope it was **defined in**, forming a chain (the "scope chain") that JS walks outward when resolving a variable name.

---

## 2. What Is a Closure? (the precise definition)

> A closure is a function that **remembers and can access variables from its outer (enclosing) scope**, even after that outer function has finished executing.

### The canonical example

```js
function makeCounter() {
  let count = 0;

  return function () {
    count++;
    return count;
  };
}

const counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

**Interview line:** "A closure is what happens when an inner function captures a reference to variables in its enclosing scope, and that reference survives even after the outer function has returned. In this example, `makeCounter` runs and finishes immediately, but the returned function keeps a live link to `count` — every call to `counter()` is reading and updating the *same* `count` variable, not a fresh one."

**Important nuance:** each *call* to `makeCounter()` creates a brand-new, independent `count`:

```js
const counterA = makeCounter();
const counterB = makeCounter();

console.log(counterA()); // 1
console.log(counterA()); // 2
console.log(counterB()); // 1 — completely separate closure
```

---

## 3. A Real, Practical Example (not just counters)

```js
function createLogger(prefix) {
  return function (message) {
    console.log(`[${prefix}] ${message}`);
  };
}

const authLogger = createLogger("AUTH");
const dbLogger = createLogger("DATABASE");

authLogger("User logged in");   // [AUTH] User logged in
dbLogger("Connection opened");  // [DATABASE] Connection opened
```

**Why this is genuinely useful:** each logger "remembers" its own prefix permanently without passing it in every call, and without any shared/global state between loggers.

---

## 4. Private Variables via Closures — Before `#privateFields` Existed

```js
function createBankAccount(initialBalance) {
  let balance = initialBalance;

  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) {
        throw new Error("Insufficient funds");
      }
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
console.log(account.balance);      // undefined — no direct access
```

**Interview line:** "The only way to read or modify `balance` is through the methods I explicitly exposed. There's no `account.balance` property; the variable lives purely inside the closure — real encapsulation, achieved through scope alone."

---

## 5. The Classic Loop + `setTimeout` Closure Bug — Full Walkthrough

```js
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100);
}
// Output: 3, 3, 3
```

### Walk through it exactly like this in an interview:

1. **"`var` is function-scoped, not block-scoped."** There is only **one** `i` for the whole loop.
2. **"Each `setTimeout` callback is a closure over that single shared `i`."** All three callbacks capture a *reference* to the one `i`, not its value at creation time.
3. **"The loop runs to completion synchronously before any callback fires."** Even with delay `0`, the loop finishes first, incrementing `i` to `3`.
4. **"By the time the callbacks run, `i` is `3` — all three log `3`."**

```js
for (let i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100);
}
// Output: 0, 1, 2
```

**Why `let` fixes it, precisely:** the spec creates a **fresh lexical binding of the loop variable for every iteration**, copying the previous value forward. Each closure captures its own independent `i`.

### The pre-ES6 fix, in terms of closures specifically

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

**Interview line:** "This is a closures problem at its core — the IIFE works because it manually creates a new scope per iteration, giving each `setTimeout` callback its own variable to close over. `let` just automates this at the language level."

---

## 6. Closures and Memory — a Good Follow-Up to Anticipate

```js
function attachHandler() {
  const hugeData = new Array(1_000_000).fill("data");

  document.getElementById("btn").addEventListener("click", function () {
    console.log("clicked");
    // hugeData stays in memory as long as this handler exists, even though unused
  });
}
```

**Interview line:** "Closures keep their entire enclosing scope alive for as long as the closure exists — even unused variables. This matters for long-lived closures like event listeners or timers that hold onto large data unnecessarily."

---

## 7. Test Tie-In: The Module Pattern for Page Objects / Test Config with Private State

### Real-time example: a Page Object with private selectors and helpers

```js
function createLoginPage(driver) {
  const selectors = {
    username: "#username",
    password: "#password",
    submitBtn: "#login-submit",
  };

  async function fillField(selector, value) {
    const el = await driver.findElement(selector);
    await el.sendKeys(value);
  }

  return {
    async login(username, password) {
      await fillField(selectors.username, username);
      await fillField(selectors.password, password);
      await (await driver.findElement(selectors.submitBtn)).click();
    },
    async getErrorMessage() {
      const el = await driver.findElement("#error-message");
      return el.getText();
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

**Interview line:** "This is the module pattern applied to test automation — the Page Object exposes a clean public API while keeping selectors and low-level DOM interaction private via closure. If the HTML changes, I update `createLoginPage` in one place; every test using `loginPage.login(...)` keeps working."

### Real-time example: test config with private, protected state

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
    request: async (path) => {
      return fetch(`${baseUrl}${path}`, {
        headers: { Authorization: `Bearer ${apiKey}` },
      });
    },
  };
}

const config = createTestConfig("staging");
await config.request("/users"); // apiKey used internally, never exposed
```

**Interview line:** "A shared test config object holding sensitive values like an API key privately. Tests make authenticated requests through `config.request(...)` without ever touching the key directly, reducing the risk of accidentally logging or leaking it."

---

## 8. Interview Q&A Script

**Q: What is a closure? Give a real example.**
> "A closure is a function that retains access to variables from its enclosing scope, even after that outer function has finished running. A practical example beyond the classic counter: a `createLogger(prefix)` factory, where each returned logger permanently remembers its own `prefix` via closure, without a shared global or repeating the prefix on every call."

**Q: Classic loop + `setTimeout` closure bug — walk through it.**
> *(walk through the full 4-step explanation in section 5, out loud, then show the `let` fix and explain precisely why it works — fresh binding per iteration — not just "let fixes it")*

**Q: How would you create private variables in JavaScript before class private fields existed?**
> "Using a closure — return an object of methods from a factory function, keeping actual state as local variables inside that function. Since nothing outside has a reference to those variables, they're inaccessible except through the methods you explicitly return."

**Q: Can closures cause memory leaks?**
> "Yes — a closure keeps its entire enclosing scope alive as long as it's reachable, even variables the inner function doesn't use. Worth watching for in long-lived closures like event listeners or intervals that hold onto large data structures unnecessarily."

**Q: Why is the module pattern useful for test automation specifically?**
> "It hides implementation details — selectors, helper functions, sensitive config like API keys — behind a clean public API using closures. Page Objects are the classic example: tests call `loginPage.login(...)` without knowing the actual selectors, so UI changes only require updating the Page Object, not every test file."

---

## 9. One-Page Cheat Sheet

- **Lexical scope:** a function's variable access is determined by where it's *defined*, not where it's called.
- **Closure:** an inner function retaining a live reference to enclosing-scope variables even after the outer function returns. Each call to the outer function creates its own independent closure.
- **Private variables via closure:** return an object of methods from a factory function; keep real state as local variables with no external access — the pre-`#field` way to get encapsulation.
- **Loop + `setTimeout` bug:** `var` shares one binding across iterations → every closure sees the same final value. `let` creates a fresh binding per iteration → each closure captures its own value. The old IIFE fix manually recreates what `let` now automates.
- **Memory:** closures keep their entire enclosing scope alive, not just the variables they use — watch for this in long-lived closures holding large unused data.
- **Testing pattern:** the module pattern (Page Objects, test config objects) uses closures to expose a clean public API while hiding selectors, helpers, and sensitive values as private internal state.
