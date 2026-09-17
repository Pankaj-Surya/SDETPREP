# Day 1 — Variables, Scope & Hoisting

Everything you need to *understand it*, *explain it out loud*, and *answer follow-ups* in an interview.

---

## 1. The Core Difference: `var` vs `let` vs `const`

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function scope | Block scope | Block scope |
| Hoisting | Hoisted + initialized as `undefined` | Hoisted but **not initialized** (TDZ) | Hoisted but **not initialized** (TDZ) |
| Re-declaration | Allowed | Not allowed (same scope) | Not allowed |
| Re-assignment | Allowed | Allowed | **Not allowed** (binding is fixed) |
| Attaches to `window`/global object | Yes (in browsers, global scope) | No | No |

**Key mental model:** `var` doesn't care about `{ }` blocks — it only cares about functions. `let`/`const` respect every `{ }` block: `if`, `for`, `while`, even a bare `{ }`.

### Real-time example: function scope vs block scope

```js
function checkout(cartTotal) {
  if (cartTotal > 100) {
    var discountVar = 0.1;   // function-scoped
    let discountLet = 0.1;   // block-scoped
  }

  console.log(discountVar);  // 0.1 — leaks out of the if-block
  console.log(discountLet);  // ❌ ReferenceError: discountLet is not defined
}

checkout(150);
```

**Why this matters in real code:** imagine `discountVar` accidentally being reused later in a long function (e.g., in a pricing engine) — with `var`, you could silently apply a stale discount from an earlier block. `let` throws early and loudly, saving you from a production bug.

---

## 2. Hoisting — What Actually Happens

**Hoisting** = JS's compile step moves *declarations* (not initializations) to the top of their scope before code executes.

- `var x = 5;` → JS splits this into `var x;` (hoisted to top, set to `undefined`) and `x = 5;` (stays in place).
- `let`/`const` are also hoisted, but they land in the **Temporal Dead Zone (TDZ)** — a state where the variable exists but touching it throws an error.

```js
console.log(a); // undefined (not an error!)
var a = 10;

console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 20;
```

### Real-time example: why this bites people with functions

```js
function getUser() {
  console.log(typeof getUserName); // "undefined" — no crash, just confusing
  var getUserName = function () {
    return "Alice";
  };
}
```

If a teammate expects `getUserName` to be callable at the top (like a hoisted function declaration), this silently fails at runtime instead of throwing a clear error — a classic `var` footgun.

---

## 3. The Temporal Dead Zone (TDZ), precisely

The TDZ is the time between:
1. Entering the scope (block/function starts), and
2. The line where the `let`/`const` variable is actually declared.

During that window, the variable is "in scope" (JS knows it exists) but accessing it throws.

```js
{
  // TDZ for `score` starts here
  console.log(score); // ❌ ReferenceError
  let score = 100;    // TDZ ends here
  console.log(score); // 100
}
```

**Why does the TDZ exist at all?** To catch bugs early. Without it, you could read a variable before it's meaningfully assigned and get `undefined` silently (like `var` does) — leading to bugs that are hard to trace. TDZ forces a hard failure instead.

---

## 4. `const` — a common misconception

`const` doesn't mean "immutable value," it means "immutable binding" (the variable name can't be reassigned). Objects/arrays declared with `const` can still be mutated internally.

```js
const cart = { items: [], total: 0 };

cart.items.push("Laptop"); // ✅ fine — mutating the object, not reassigning
cart.total = 999;          // ✅ fine

cart = { items: [] };      // ❌ TypeError: Assignment to constant variable.
```

**Real-time example:** In a shopping cart reducer or state object, you'll almost always use `const` for the object reference and just mutate/replace its properties — this is idiomatic in real apps (before you even bring in immutability libraries like Immer).

---

## 5. The Big One: `var` vs `let` in Loops with Async Callbacks

This is the single most commonly asked practical question on this topic — and it's directly relevant to **looping over test data with `setTimeout`, API calls, or `beforeEach` hooks**.

### The broken version (`var`)

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log("var i:", i);
  }, 100);
}
// Output:
// var i: 3
// var i: 3
// var i: 3
```

**Why:** `var` is function-scoped, so there is only **one** `i` shared across all loop iterations. By the time the callbacks run (after the loop finishes), `i` is already `3`.

### The fixed version (`let`)

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log("let i:", i);
  }, 100);
}
// Output:
// let i: 0
// let i: 1
// let i: 2
```

**Why:** `let` is block-scoped, and JS engines create a **new binding of `i` for every iteration** of the loop. Each `setTimeout` closure captures its own snapshot of `i`.

### Direct testing tie-in

```js
const testCases = [
  { input: 2, expected: 4 },
  { input: 3, expected: 9 },
  { input: 4, expected: 16 },
];

for (var i = 0; i < testCases.length; i++) {
  it(`squares ${testCases[i].input} correctly`, () => {
    // BUG: by the time this callback runs (Mocha/Jest schedule these),
    // `i` might already be testCases.length, causing
    // "Cannot read property 'input' of undefined" or wrong assertions
    expect(square(testCases[i].input)).toBe(testCases[i].expected);
  });
}
```

Switch `var` → `let` and each `it()` callback correctly closes over its own `i`. This exact pattern shows up constantly when dynamically generating test cases in a loop.

### Pre-`let` workaround (good to know, shows depth)

Before ES6, engineers used an IIFE (Immediately Invoked Function Expression) to force a new scope per iteration:

```js
for (var i = 0; i < 3; i++) {
  (function (capturedI) {
    setTimeout(() => console.log(capturedI), 100);
  })(i);
}
```

Mentioning this in an interview shows you understand *why* `let` was such a big deal when it was introduced — it's solving a problem that used to require a manual pattern.

---

## 6. Interview Q&A Script

**Q: What happens if you access a `let` variable before its declaration?**
> "It throws a `ReferenceError` because of the Temporal Dead Zone. The variable is hoisted to the top of the block, but it's not initialized until the actual `let` line executes. JS deliberately blocks access during that window instead of returning `undefined`, so bugs surface immediately instead of silently propagating."

**Q: Why avoid `var` in modern code?**
> "Three main reasons: it's function-scoped instead of block-scoped, which lets variables leak out of `if`/`for` blocks and cause naming collisions; it's hoisted and initialized as `undefined`, which can mask bugs since accessing it early doesn't throw; and it doesn't create a new binding per loop iteration, which breaks closures in loops with async code — a real bug I've seen with `setTimeout` and test generation loops."

**Q: Is `const` truly immutable?**
> "No — `const` only prevents reassignment of the variable binding, not mutation of the underlying value. Objects and arrays declared with `const` can still have their properties/elements changed."

**Q: What's the difference between hoisting of `var` and hoisting of function declarations?**
> "Function declarations are hoisted with their full body, so you can call them before their definition in the code. `var` is hoisted but only the declaration — the assignment stays in place, so the variable exists as `undefined` until the assignment line actually runs."

**Q: Why does the loop/closure bug matter in real projects, not just for interviews?**
> "It shows up anywhere you loop and schedule async work per iteration — event listeners in a UI list, batched API calls, or dynamically generated test cases (`for` loop generating `it()` blocks). Using `let` (or an IIFE in older code) ensures each iteration's callback captures the correct value."

---

## 7. One-Page Cheat Sheet (for quick review before the interview)

- **Scope:** `var` = function scope. `let`/`const` = block scope (`{ }`).
- **Hoisting:** All three are hoisted. `var` → initialized to `undefined`. `let`/`const` → uninitialized, sit in the **TDZ** until their line runs.
- **Re-declare/re-assign:** `var` allows both. `let` allows reassignment only. `const` allows neither (binding-wise; contents of objects/arrays are still mutable).
- **Loop + async gotcha:** `var` shares one binding across all iterations → stale/final value used in callbacks. `let` creates a fresh binding per iteration → correct value captured.
- **Old-school fix:** IIFE to manually create per-iteration scope before `let` existed.
- **Golden rule to say in interviews:** "Default to `const`, use `let` when reassignment is needed, avoid `var` entirely in modern code."
