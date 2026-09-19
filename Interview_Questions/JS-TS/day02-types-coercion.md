# Day 2 — Data Types & Type Coercion

Everything you need to *understand it*, *explain it out loud*, and *answer follow-ups* in an interview.

---

## 1. Primitives vs Reference Types

| | Primitives | Reference Types |
|---|---|---|
| Examples | `string`, `number`, `boolean`, `undefined`, `null`, `symbol`, `bigint` | `object`, `array`, `function` |
| Stored as | Value, directly in the variable | Reference (pointer) to a location in memory |
| Copied by | Value (a real, independent copy) | Reference (both variables point to the same object) |
| Compared with `===` | Compares actual values | Compares memory references, not contents |

### Real-time example: the classic "copy" bug

```js
// Primitive — safe copy
let priceA = 100;
let priceB = priceA;
priceB = 200;
console.log(priceA); // 100 — untouched

// Reference type — shared copy
const userA = { name: "Alice", cart: [] };
const userB = userA;
userB.cart.push("Laptop");
console.log(userA.cart); // ["Laptop"] — userA got mutated too!
```

**Why this matters in real code:** this is the #1 cause of "I didn't even touch that object, why did it change?" bugs — two variables pointing at the same object in memory, especially common when passing objects into functions or storing them in shared state (Redux, React state, etc.).

---

## 2. `==` vs `===`

- `===` (strict equality): compares value **and** type. No conversion happens.
- `==` (loose equality): converts one or both operands to a common type first (**type coercion**), then compares.

```js
"5" === 5     // false — different types, no conversion
"5" == 5      // true  — "5" is coerced to number 5, then compared

null === undefined  // false
null == undefined   // true — special case, only equal to each other

0 == false    // true — false is coerced to 0
0 === false   // false — different types
```

**Golden rule to say in interviews:** "Always use `===` and `!==` unless you have a specific, well-understood reason to use loose equality — it removes an entire category of coercion bugs."

---

## 3. Explaining `[] == false` (a favorite interview trap)

Walk through it step by step out loud — this is what interviewers actually want to see, not just the answer:

```js
[] == false
```

1. `==` sees a boolean (`false`) on one side → converts `false` to a number: `Number(false)` → `0`.
   So now: `[] == 0`.
2. Now it's an object (`[]`) compared to a number → object gets converted to a primitive via `ToPrimitive`.
   For an array, that calls `.toString()` → `[].toString()` → `""` (empty string).
   So now: `"" == 0`.
3. Now it's a string vs number → the string gets converted to a number: `Number("")` → `0`.
   So now: `0 == 0`.
4. `0 == 0` → **true**.

```js
console.log([] == false); // true
```

**Interview line:** "The `==` operator keeps coercing operands using the abstract equality algorithm until both sides are the same type — booleans become numbers, objects become primitives via `toString`/`valueOf`, and empty strings become `0`. It's a great example of why `===` is safer: it skips all of this entirely."

---

## 4. Why `typeof null === 'object'`

This is a **known, decades-old bug in JavaScript** that can never be fixed because too much existing code depends on the current behavior (fixing it would break the web).

**The real reason:** in the original JavaScript engine, values were represented internally with a type tag plus the actual value. Objects had a type tag of `0`. `null` was represented as the null pointer (`0x00` on most platforms) — which also happened to have a type tag of `0`. So `typeof` checks the tag, sees `0`, and reports `"object"`.

```js
typeof null;        // "object"  (the famous bug)
typeof undefined;   // "undefined"
typeof 42;           // "number"
typeof "hi";          // "string"
typeof true;          // "boolean"
typeof Symbol();      // "symbol"
typeof 10n;           // "bigint"
typeof function(){};  // "function"
typeof [];            // "object" (arrays are objects too — use Array.isArray() to distinguish)
```

**Interview line:** "It's a legacy implementation bug from JS's original design where `null`'s internal type tag collided with the object type tag. It's kept for backward compatibility. If you need to actually check for `null`, you compare it directly with `=== null`, not `typeof`."

---

## 5. Truthy / Falsy Values

There are only **8 falsy values** in JavaScript — everything else is truthy.

```js
false
0
-0
0n        // BigInt zero
""        // empty string
null
undefined
NaN
```

Everything else — including `"0"`, `"false"`, `[]`, `{}` — is **truthy**.

```js
if ([]) console.log("arrays are truthy, even empty ones!");   // runs
if ({}) console.log("objects are truthy, even empty ones!");  // runs
if ("0") console.log("non-empty strings are truthy, even '0'"); // runs
```

### Real-time example: a common bug in feature flags / config

```js
function getDiscount(couponCode) {
  if (couponCode) {
    return applyCoupon(couponCode);
  }
  return 0;
}

getDiscount("");     // falsy → correctly returns 0
getDiscount("0");    // truthy! → tries to apply coupon "0" — probably a bug
getDiscount(0);      // falsy → returns 0, but what if 0 was a *valid* coupon ID?
```

**Takeaway to mention in interviews:** truthy/falsy checks are convenient but dangerous when `0`, `""`, or `NaN` are legitimate valid values in your domain — that's when you should switch to an explicit check like `couponCode !== undefined && couponCode !== null`, or the nullish coalescing operator `??`.

---

## 6. `NaN` — Not a Number, but `typeof NaN === "number"`

```js
typeof NaN;      // "number" — confusing but consistent: NaN represents an invalid *numeric* result
NaN === NaN;     // false — NaN is the only value in JS not equal to itself
isNaN("hello");  // true — but isNaN() coerces its argument first (misleading!)
Number.isNaN("hello"); // false — doesn't coerce, correctly checks the value is actually NaN
```

### Real-time example: why global `isNaN` is dangerous

```js
isNaN("hello");         // true  — coerces "hello" to NaN first, then checks
Number.isNaN("hello");  // false — "hello" is a string, not NaN, no coercion

isNaN(undefined);        // true  — Number(undefined) is NaN
Number.isNaN(undefined); // false — undefined is not literally NaN
```

**Interview line:** "Global `isNaN()` coerces its argument to a number before checking, which produces false positives for things like strings. `Number.isNaN()` is the safe version — it only returns `true` if the value is *actually* `NaN`, with no coercion. I always use `Number.isNaN()` in real code."

### How to actually check for `NaN`

```js
const result = 0 / 0;
result === NaN;          // false! (see above — NaN !== NaN)
Number.isNaN(result);    // true — correct way
Object.is(result, NaN);  // true — also correct, and handles -0 vs 0 edge cases too
```

---

## 7. Test Tie-In: Assertion Pitfalls with `===` and Deep Equality

This is a real, everyday bug in test suites — not just theory.

```js
// Reference types are compared by reference, not contents
const expected = { id: 1, name: "Alice" };
const actual = { id: 1, name: "Alice" };

expected === actual;   // false — different objects in memory, even though contents match!
```

If you write a naive assertion like this in a test:

```js
// ❌ This will FAIL even though the data is "correct"
assert(actual === expected);
```

...it fails, not because the logic is wrong, but because `===` on objects/arrays checks **identity**, not **structural equality**.

### Why deep-equality matchers exist

```js
// Jest
expect(actual).toEqual(expected);        // ✅ deep equality — compares contents
expect(actual).toBe(expected);           // ❌ reference equality — would fail here

// Chai
expect(actual).to.deep.equal(expected);  // ✅ deep equality
expect(actual).to.equal(expected);       // ❌ reference equality

// Node's assert module
assert.deepStrictEqual(actual, expected); // ✅ deep equality
assert.strictEqual(actual, expected);     // ❌ reference equality
```

**Real-time example: array of test results**

```js
function getActiveUsers(users) {
  return users.filter(u => u.active);
}

const result = getActiveUsers([
  { id: 1, active: true },
  { id: 2, active: false },
]);

// ❌ Wrong — new array reference every time, will always fail
expect(result).toBe([{ id: 1, active: true }]);

// ✅ Right — checks structural/content equality
expect(result).toEqual([{ id: 1, active: true }]);
```

**Interview line:** "`toBe`/`===`-style checks compare object identity, so two structurally identical objects or arrays will never be equal unless they're literally the same reference in memory. That's exactly why testing libraries ship a separate deep-equality matcher like `toEqual` or `deepStrictEqual` — they recursively compare properties/elements instead of the reference pointer. I've seen flaky-looking test failures that were actually just someone using `toBe` instead of `toEqual` on an object."

---

## 8. Interview Q&A Script

**Q: Explain `[] == false`.**
> "Walk through the abstract equality algorithm: `false` gets converted to `0`. Then `[]`, being an object, gets converted to a primitive via `toString()`, giving `""`. Then `""` gets converted to a number, giving `0`. So it ends up as `0 == 0`, which is `true`. It's a good example of why I default to `===`."

**Q: Why is `typeof null === 'object'`?**
> "It's a legacy bug from JS's original implementation — `null` and objects shared the same internal type tag (`0`), so `typeof` reports `'object'` for both. It can't be fixed without breaking the web, so it's a permanent quirk. To check for `null` specifically, use `=== null`, not `typeof`."

**Q: What's the difference between primitives and reference types when passed to functions?**
> "Primitives are passed by value — the function gets an independent copy, so changes inside the function don't affect the original. Reference types are passed by reference — the function gets a pointer to the same object, so mutating properties inside the function *does* affect the original object outside it."

**Q: Why shouldn't you use `toBe` (or `===`) to compare objects/arrays in tests?**
> "Because `===` and `toBe` check reference identity, not content. Two objects with identical properties are still different objects in memory unless they're the exact same reference, so the assertion fails even when the data is logically correct. That's what deep-equality matchers like `toEqual` or `deepStrictEqual` are for — they recursively compare structure and values instead of memory addresses."

**Q: What are the 8 falsy values in JS?**
> "`false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`. Everything else — including empty arrays, empty objects, and the string `'0'` — is truthy."

---

## 9. One-Page Cheat Sheet

- **Primitives** copy by value; **reference types** (objects/arrays/functions) copy by reference — mutating a shared reference affects every variable pointing at it.
- **`===`** compares value + type, no conversion. **`==`** coerces types first, then compares — avoid it except for the `== null` idiom (catches both `null` and `undefined`).
- **`[] == false`** → `true`, via `false→0`, `[]→""→0`, then `0==0`.
- **`typeof null === "object"`** — a legacy engine bug (shared type tag with objects), permanent for backward compatibility.
- **Falsy values (memorize all 8):** `false, 0, -0, 0n, "", null, undefined, NaN`. Everything else is truthy — including `[]`, `{}`, and `"0"`.
- **`NaN`** is the only value not equal to itself. Use `Number.isNaN()`, not global `isNaN()` (which coerces first).
- **Testing rule:** use `toEqual` / `deepStrictEqual` for objects and arrays (structural comparison); use `toBe` / `===` / `strictEqual` only for primitives (reference/value comparison is the same thing for primitives).
