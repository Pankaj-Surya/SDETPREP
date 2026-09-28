# Day 5 — Array Methods (map / filter / reduce / find / etc.)

Everything you need to *understand it*, *explain it out loud*, and *answer follow-ups* in an interview.

---

## 1. The Core Iteration Methods, Side by Side

| Method | Returns | Purpose | Stops early? |
|---|---|---|---|
| `forEach` | `undefined` | Run side effects per element | No |
| `map` | New array (same length) | Transform each element | No |
| `filter` | New array (subset) | Keep elements matching a condition | No |
| `reduce` | Single accumulated value | Combine all elements into one result | No |
| `find` | First matching element (or `undefined`) | Get one item | **Yes** |
| `findIndex` | Index of first match (or `-1`) | Get one item's position | **Yes** |
| `some` | Boolean | "Does at least one match?" | **Yes** |
| `every` | Boolean | "Do all match?" | **Yes** |

**Interview line to open with:** "The big mental model split is: `map`/`filter` always return a new array of the same or smaller size, `reduce` collapses everything into a single value, and `find`/`some`/`every` short-circuit — they stop iterating as soon as they have their answer, which matters for performance on large arrays."

---

## 2. `forEach` vs `map` — the most common mix-up

```js
const prices = [10, 20, 30];

// forEach — just runs a function per item, returns undefined
const result1 = prices.forEach((p) => p * 2);
console.log(result1); // undefined — forEach does NOT build a new array

// map — transforms each item and returns a brand-new array
const result2 = prices.map((p) => p * 2);
console.log(result2); // [20, 40, 60]
```

### Real-time example: the classic anti-pattern

```js
// ❌ Anti-pattern — using map when you don't need the returned array (side effects only)
prices.map((p) => console.log(p)); // "works" but wastes memory building an unused array

// ✅ Correct — forEach is the right tool when you're not building anything
prices.forEach((p) => console.log(p));

// ❌ Anti-pattern — using forEach when you actually need a transformed array
const doubled = [];
prices.forEach((p) => doubled.push(p * 2)); // works, but verbose and mutates an external array

// ✅ Correct — map is the right tool for transformation
const doubled2 = prices.map((p) => p * 2);
```

**Interview line:** "Use `forEach` for side effects — logging, pushing to an external system, DOM updates. Use `map` when you actually want a new array back. If I see a `map` call whose return value is never used, that's a code smell — it should be a `forEach`."

---

## 3. `filter` — keep what matches

```js
const users = [
  { id: 1, name: "Alice", active: true },
  { id: 2, name: "Bob", active: false },
  { id: 3, name: "Eve", active: true },
];

const activeUsers = users.filter((u) => u.active);
console.log(activeUsers); // [{ id: 1, ... }, { id: 3, ... }]
```

`filter` always returns an **array** — even if zero elements match (`[]`), or even if only one matches.

```js
users.filter((u) => u.id === 999); // [] — not undefined, not null, an empty array
```

---

## 4. `find` — get the first match, not an array

```js
const user = users.find((u) => u.id === 2);
console.log(user); // { id: 2, name: "Bob", active: false }

const missing = users.find((u) => u.id === 999);
console.log(missing); // undefined — not an empty array!
```

### `find` vs `filter` — the key distinction to state clearly in an interview

> "`filter` always returns an array, even with zero or one match — you use it when you expect (or want to handle) multiple results. `find` returns the actual element itself (or `undefined`), and stops iterating the moment it finds a match — you use it when you expect at most one result and want the object directly, not wrapped in an array."

```js
// ❌ Awkward — using filter when you want a single item
const bob = users.filter((u) => u.id === 2)[0]; // works, but scans the whole array and wraps/unwraps needlessly

// ✅ Correct — find is built for exactly this
const bob2 = users.find((u) => u.id === 2); // stops as soon as it's found, returns the object directly
```

---

## 5. `some` and `every` — boolean checks that short-circuit

```js
const hasInactiveUser = users.some((u) => !u.active);
console.log(hasInactiveUser); // true — stops at Bob, doesn't check Eve

const allActive = users.every((u) => u.active);
console.log(allActive); // false — stops at Bob immediately, doesn't check Eve
```

**Real-time example: form/data validation**

```js
function isValidOrder(order) {
  return order.items.length > 0 && order.items.every((item) => item.quantity > 0);
}

isValidOrder({ items: [{ quantity: 2 }, { quantity: 1 }] }); // true
isValidOrder({ items: [{ quantity: 2 }, { quantity: 0 }] }); // false — one bad item invalidates the order
```

---

## 6. `reduce` — the most powerful and most misunderstood

`reduce(callback, initialValue)` walks the array, carrying an **accumulator** forward through each step, and returns the final accumulator value.

```js
const numbers = [1, 2, 3, 4];

const total = numbers.reduce((accumulator, current) => accumulator + current, 0);
// step by step: 0+1=1 → 1+2=3 → 3+3=6 → 6+4=10
console.log(total); // 10
```

### `reduce` can replace `map` and `filter` (good to mention, shows depth — but don't overuse it)

```js
// reduce doing filter's job
const evens = numbers.reduce((acc, n) => (n % 2 === 0 ? [...acc, n] : acc), []);
// [2, 4]

// reduce doing map's job
const doubled = numbers.reduce((acc, n) => [...acc, n * 2], []);
// [2, 4, 6, 8]
```

**Interview line:** "`reduce` is the most general-purpose array method — `map`, `filter`, even `find` can technically be implemented in terms of `reduce`. That said, in real code I still reach for `map`/`filter` when that's literally what I'm doing, since it reads more clearly — I use `reduce` when I'm building something that doesn't fit those shapes, like grouping, counting, or flattening into a single object."

### Real-time example: grouping data (a `reduce` use case `map`/`filter` can't do cleanly)

```js
const orders = [
  { customer: "Alice", amount: 100 },
  { customer: "Bob", amount: 50 },
  { customer: "Alice", amount: 30 },
];

const totalsByCustomer = orders.reduce((acc, order) => {
  acc[order.customer] = (acc[order.customer] || 0) + order.amount;
  return acc;
}, {});

console.log(totalsByCustomer); // { Alice: 130, Bob: 50 }
```

**Common gotcha to mention:** forgetting the `initialValue` (the second argument). Without it, `reduce` uses the array's first element as the initial accumulator and starts iterating from index 1 — which breaks on empty arrays (`TypeError: Reduce of empty array with no initial value`) and can produce subtly wrong results for non-numeric accumulation.

```js
[].reduce((acc, n) => acc + n); // ❌ TypeError — no initial value, empty array
[].reduce((acc, n) => acc + n, 0); // ✅ 0 — safe, always provide initialValue
```

---

## 7. Implement `reduce` From Scratch (classic interview whiteboard question)

```js
function myReduce(array, callback, initialValue) {
  let accumulator = initialValue;
  let startIndex = 0;

  // If no initialValue was provided, use the first element and start from index 1
  if (accumulator === undefined) {
    if (array.length === 0) {
      throw new TypeError("Reduce of empty array with no initial value");
    }
    accumulator = array[0];
    startIndex = 1;
  }

  for (let i = startIndex; i < array.length; i++) {
    accumulator = callback(accumulator, array[i], i, array);
  }

  return accumulator;
}

// Test it
console.log(myReduce([1, 2, 3, 4], (acc, n) => acc + n, 0)); // 10
console.log(myReduce([1, 2, 3, 4], (acc, n) => acc + n));    // 10 (no initial value)
```

**Talking through it in the interview (say this out loud while you write):**
1. "I need an accumulator variable that starts at `initialValue`."
2. "If `initialValue` wasn't passed, I fall back to the array's first element and start looping from index 1 instead of 0 — that matches real `reduce` behavior."
3. "Then I loop through the rest of the array, calling the callback with `(accumulator, currentElement, index, array)` each time and reassigning the accumulator to whatever the callback returns."
4. "Edge case: an empty array with no initial value should throw, just like the real `Array.prototype.reduce` does."

This is a great chance to also mention: "I'd handle the edge case of `initialValue` genuinely being `undefined` on purpose — real `reduce` actually checks `arguments.length` rather than checking if the value `=== undefined`, to correctly distinguish 'no second argument passed' from 'second argument passed as `undefined`.' I simplified that here, but I know the distinction."

---

## 8. Test Tie-In: Filtering API Responses & Extracting Fields to Assert On

This is exactly what these methods look like in real test code — not abstract exercises.

### Real-time example: validating a paginated API response

```js
async function getActiveAdmins(apiResponse) {
  return apiResponse.users
    .filter((u) => u.active && u.role === "admin")
    .map((u) => u.email);
}

const mockResponse = {
  users: [
    { id: 1, email: "alice@co.com", role: "admin", active: true },
    { id: 2, email: "bob@co.com", role: "user", active: true },
    { id: 3, email: "eve@co.com", role: "admin", active: false },
  ],
};

const admins = await getActiveAdmins(mockResponse);
console.log(admins); // ["alice@co.com"] — filter narrows, map extracts
```

Test for it:

```js
it("returns only active admins' emails", async () => {
  const result = await getActiveAdmins(mockResponse);
  expect(result).toEqual(["alice@co.com"]); // deep equality on the array — see Day 2!
});
```

**Interview line:** "This `filter` → `map` chain is one of the most common real-world patterns: `filter` narrows an API response down to the records you care about, then `map` extracts just the field(s) you actually want to assert on, instead of asserting against the full noisy response object."

### `find` vs `filter` in test data validation — the exact interview question

```js
const testCases = [
  { id: "TC-001", status: "passed" },
  { id: "TC-002", status: "failed" },
  { id: "TC-003", status: "passed" },
];

// Use `find` when you expect exactly one match and want the object itself
const failedCase = testCases.find((tc) => tc.status === "failed");
expect(failedCase).toBeDefined();
expect(failedCase.id).toBe("TC-002");

// Use `filter` when you expect multiple matches, or want to assert on a set
const passedCases = testCases.filter((tc) => tc.status === "passed");
expect(passedCases).toHaveLength(2);
expect(passedCases.map((tc) => tc.id)).toEqual(["TC-001", "TC-003"]);
```

**Interview line:** "I use `find` in test validation when I'm asserting about a single expected record — like 'there should be exactly one failed test case, and here's what its ID should be.' I use `filter` when I'm validating a subset or a count — like 'there should be exactly two passed test cases.' Using `filter()[0]` when you really mean `find` is a common smell — it works, but it scans the whole array unnecessarily and returns `undefined` in a less explicit way if nothing matches."

### Using `some`/`every` to assert on a whole collection at once

```js
it("all API responses returned status 200", () => {
  const responses = [{ status: 200 }, { status: 200 }, { status: 200 }];
  expect(responses.every((r) => r.status === 200)).toBe(true);
});

it("at least one response failed", () => {
  const responses = [{ status: 200 }, { status: 500 }];
  expect(responses.some((r) => r.status >= 400)).toBe(true);
});
```

---

## 9. Interview Q&A Script

**Q: Implement `reduce` from scratch.**
> *(see full walkthrough in section 7 — talk through the accumulator, the initialValue fallback, and the empty-array edge case out loud as you write it)*

**Q: When would you use `find` vs `filter` in test data validation?**
> "`find` when I expect at most one matching record and want the object itself directly — like locating the one failed test case to assert on its details. `filter` when I expect multiple matches and want to assert on the whole subset — like the count of passed test cases, or extracting a list of IDs. If I only ever use index `[0]` after a `filter`, that's usually a sign I actually wanted `find`."

**Q: What's the difference between `forEach` and `map`?**
> "`forEach` runs a callback for its side effects and always returns `undefined` — it doesn't build anything. `map` returns a brand-new array of the same length, with each element transformed by the callback. If you're not using the return value, you want `forEach`; if you need a transformed array back, you want `map`."

**Q: Why do `find`, `some`, and `every` matter for performance compared to `filter`?**
> "They short-circuit — `find` and `some` stop as soon as they get a match, `every` stops as soon as it gets a failure. `filter` always processes every element, since it needs to check all of them to build the complete result array. On a large array, if you only need to know 'does one exist' or 'get the first one,' `some`/`find` can be significantly faster than filtering the whole thing."

**Q: What happens if you call `reduce` without an initial value on an empty array?**
> "It throws a `TypeError: Reduce of empty array with no initial value`, because there's nothing to use as a starting accumulator and no elements to iterate. That's exactly why I always pass an explicit `initialValue` — it also protects against subtle bugs even on non-empty arrays, since without it the accumulator's type is determined by the first array element instead of being explicit."

**Q: Can `reduce` do everything `map` and `filter` can do? Should you use it that way?**
> "Technically yes — `reduce` is the most general iteration method and can implement `map`, `filter`, even `find`. But I wouldn't do that in real code just to prove a point — `map`/`filter` communicate intent more clearly to whoever reads it next. I reach for `reduce` specifically when the transformation doesn't fit the 'same-length array' or 'subset array' shape — like grouping, counting, or building an object from an array."

---

## 10. One-Page Cheat Sheet

- **`forEach`** → side effects, returns `undefined`. **`map`** → transform, returns a new same-length array. Never use `map` and ignore the result — that's a `forEach`.
- **`filter`** → always returns an array (possibly empty). **`find`** → returns the element itself or `undefined`, stops at first match. Don't do `filter(...)[0]` — use `find`.
- **`some`** → "any match?" (true/false, short-circuits on first `true`). **`every`** → "all match?" (true/false, short-circuits on first `false`).
- **`reduce`** → collapses the array into one value via an accumulator. Always pass an explicit `initialValue` to avoid `TypeError` on empty arrays and avoid implicit type assumptions.
- **`reduce` can implement `map`/`filter`,** but prefer the specific method for readability; reach for `reduce` for grouping, counting, or building non-array/non-boolean results.
- **Real-world test pattern:** `filter` narrows an API response to relevant records → `map` extracts the specific field(s) to assert on → deep-equality assertion (`toEqual`) on the resulting array.
- **Test validation rule:** `find` for "exactly one expected record," `filter` for "a subset/count of records," `some`/`every` for "at least one" / "all" boolean checks across a whole collection.
