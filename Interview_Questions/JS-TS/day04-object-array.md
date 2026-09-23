# Day 4 — Objects & Arrays (Deep Dive)

Everything you need to *understand it*, *explain it out loud*, and *answer follow-ups* in an interview.

---

## 1. Object & Array Destructuring

Destructuring lets you unpack values from objects/arrays into variables directly, instead of accessing properties one by one.

```js
// Object destructuring
const user = { id: 1, name: "Alice", role: "admin" };
const { name, role } = user;
console.log(name, role); // Alice admin

// Renaming while destructuring
const { name: userName } = user;
console.log(userName); // Alice

// Default values (kicks in only if the property is undefined)
const { age = 25 } = user;
console.log(age); // 25 — `age` doesn't exist on `user`

// Array destructuring — positional, not by key
const scores = [90, 85, 70];
const [first, second] = scores;
console.log(first, second); // 90 85

// Skipping elements
const [, , third] = scores;
console.log(third); // 70
```

### Real-time example: destructuring function parameters (very common in real apps)

```js
function createOrder({ customerId, items, discount = 0 }) {
  const total = items.reduce((sum, item) => sum + item.price, 0);
  return { customerId, total: total - discount };
}

createOrder({
  customerId: 42,
  items: [{ price: 100 }, { price: 50 }],
  discount: 10,
});
// { customerId: 42, total: 140 }
```

**Interview line:** "Destructuring function parameters is a really common pattern for functions that take a config/options object — it documents exactly what properties the function expects right in the signature, and lets you set defaults per-property instead of one big fallback object."

### Nested destructuring

```js
const response = {
  data: { user: { id: 1, name: "Alice" } },
  status: 200,
};

const {
  status,
  data: {
    user: { name },
  },
} = response;

console.log(status, name); // 200 Alice
```

---

## 2. Spread and Rest — Same Syntax (`...`), Opposite Directions

- **Spread** *expands* an iterable/object into individual elements — used when *creating* something.
- **Rest** *collects* multiple elements into a single array/object — used when *receiving* something (destructuring or function params).

```js
// Spread — expanding
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];       // [1, 2, 3, 4, 5]

const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 };     // { a: 1, b: 2, c: 3 }

// Rest — collecting
const [head, ...tail] = arr2;       // head = 1, tail = [2, 3, 4, 5]
const { a, ...rest } = obj2;        // a = 1, rest = { b: 2, c: 3 }

function sum(...numbers) {          // rest in function params
  return numbers.reduce((a, b) => a + b, 0);
}
```

### Real-time example: merging config with overrides (extremely common pattern)

```js
const defaultConfig = { retries: 3, timeout: 5000, verbose: false };

function runTask(overrides = {}) {
  const config = { ...defaultConfig, ...overrides };
  console.log(config);
}

runTask({ verbose: true });
// { retries: 3, timeout: 5000, verbose: true } — later spread wins on conflicts
```

**Key gotcha to mention:** spread on objects only performs a **shallow merge** — later keys overwrite earlier ones only at the top level, not recursively. Nested objects are fully replaced, not merged.

---

## 3. Shallow Copy vs Deep Copy

This is the conceptual core of the whole day — and the most commonly tested idea in this topic.

**Shallow copy:** copies the top-level properties only. If a property's value is itself an object/array (a reference type), the *reference* is copied, not the nested object itself — so both copies still point to the same nested object in memory.

**Deep copy:** recursively copies every level, so the copy is 100% independent — no shared references anywhere, at any depth.

```js
const original = {
  name: "Alice",
  address: { city: "NYC", zip: "10001" },
};

// Shallow copy — via spread or Object.assign
const shallowCopy = { ...original };

shallowCopy.name = "Bob";                 // fine — top-level primitive, independent
shallowCopy.address.city = "LA";          // ⚠️ mutates the SHARED nested object!

console.log(original.name);          // "Alice" — untouched, primitives copy fine
console.log(original.address.city);  // "LA" — oops! original was mutated too
```

**Why:** `{ ...original }` copies `address` as a *reference* to the same object in memory — it doesn't create a new `address` object. Only the top-level keys get fresh, independent slots.

### Same problem with arrays of objects

```js
const users = [{ id: 1, active: true }];
const usersCopy = [...users]; // shallow copy of the array itself

usersCopy[0].active = false;
console.log(users[0].active); // false — the array is new, but the object inside is shared!
```

---

## 4. How to Deep Clone an Object Without a Library

This is a direct, common interview question — know multiple approaches and their trade-offs.

### Option 1: `structuredClone()` — the modern, built-in answer (best default in 2024+)

```js
const original = { name: "Alice", address: { city: "NYC" }, tags: ["a", "b"] };
const deepCopy = structuredClone(original);

deepCopy.address.city = "LA";
console.log(original.address.city); // "NYC" — fully independent, deep clone worked
```

**Pros:** built into modern JS engines and Node (18+), no dependency needed, correctly handles `Date`, `Map`, `Set`, circular references, nested arrays/objects.
**Cons:** doesn't clone functions or class prototypes/methods (it clones plain data, not behavior); not available in very old environments.

### Option 2: `JSON.parse(JSON.stringify(obj))` — the classic "trick," with real limitations

```js
const deepCopy = JSON.parse(JSON.stringify(original));
```

**Limitations to mention (this shows real depth in an interview):**
- Loses `undefined` values, functions, and `Symbol` properties entirely (they're silently dropped)
- Converts `Date` objects into strings (loses the `Date` type)
- Throws on circular references
- Loses `Map`/`Set`/`RegExp` (they don't round-trip through JSON correctly)

```js
const tricky = { date: new Date(), fn: () => {}, undef: undefined, num: NaN };
console.log(JSON.parse(JSON.stringify(tricky)));
// { date: "2024-01-01T00:00:00.000Z", num: null } — fn and undef gone, date is now a string, NaN became null
```

### Option 3: Manual recursive clone (shows you understand the *mechanics*, great for interviews)

```js
function deepClone(value) {
  if (value === null || typeof value !== "object") {
    return value; // primitives are already "copied" by value
  }
  if (Array.isArray(value)) {
    return value.map(deepClone);
  }
  const cloned = {};
  for (const key in value) {
    if (Object.hasOwn(value, key)) {
      cloned[key] = deepClone(value[key]); // recurse into nested objects/arrays
    }
  }
  return cloned;
}
```

**Interview line:** "For a real deep clone I'd reach for `structuredClone()` first since it's now built into the language and handles edge cases like `Date` and circular references correctly. `JSON.parse(JSON.stringify())` is the classic trick but it's lossy — it silently drops functions and `undefined`, turns dates into strings, and throws on circular references. If I needed to explain the underlying mechanics, I'd write it as a recursive function: primitives return as-is, and objects/arrays get walked recursively so every nested level gets its own fresh copy instead of sharing a reference."

---

## 5. `Object.freeze()` — Shallow Immutability

```js
const config = Object.freeze({ env: "production", limits: { maxUsers: 100 } });

config.env = "staging";              // silently fails (throws in strict mode)
console.log(config.env);             // "production" — unchanged

config.limits.maxUsers = 999;        // ⚠️ this WORKS — freeze is shallow!
console.log(config.limits.maxUsers); // 999 — nested object was NOT frozen
```

**Interview line:** "`Object.freeze()` only prevents reassignment and addition/removal of the object's own top-level properties — it does *not* recursively freeze nested objects. To get real deep immutability you'd need a recursive freeze function, or reach for a library like Immer for a more practical, everyday immutable-update pattern."

```js
function deepFreeze(obj) {
  Object.values(obj).forEach((value) => {
    if (value && typeof value === "object") {
      deepFreeze(value);
    }
  });
  return Object.freeze(obj);
}
```

---

## 6. Test Tie-In: Cloning Fixtures/Test Data So Tests Don't Mutate Shared State

This is one of the most common real-world sources of **flaky, order-dependent tests** — and it's a direct consequence of shallow vs deep copy.

### The bug: a shared fixture object gets mutated by one test and leaks into another

```js
// fixtures.js — one shared object used across many test files
export const baseUser = {
  id: 1,
  name: "Alice",
  preferences: { theme: "dark", notifications: true },
};
```

```js
// ❌ test-a.test.js
import { baseUser } from "./fixtures";

it("disables notifications", () => {
  const user = { ...baseUser };              // shallow copy!
  user.preferences.notifications = false;    // mutates the SHARED preferences object
  expect(user.preferences.notifications).toBe(false); // passes... but at a cost
});
```

```js
// ❌ test-b.test.js — runs afterward, imports the SAME shared object
import { baseUser } from "./fixtures";

it("defaults to notifications enabled", () => {
  expect(baseUser.preferences.notifications).toBe(true);
  // ❌ FAILS — it's now `false`, because test-a mutated the shared object!
  // This test passes in isolation, but fails when run after test-a — classic test order dependency
});
```

**Why this happens:** `{ ...baseUser }` only copies the top-level keys. `preferences` is still the *same object in memory* as the one in the shared `fixtures.js` module — every test that spreads `baseUser` is secretly sharing that nested object.

### The fix: deep clone fixtures before mutating them in a test

```js
// ✅ test-a.test.js
import { baseUser } from "./fixtures";

it("disables notifications", () => {
  const user = structuredClone(baseUser); // fully independent copy, every level
  user.preferences.notifications = false;
  expect(user.preferences.notifications).toBe(false);
});
```

```js
// ✅ test-b.test.js — completely unaffected now, regardless of run order
import { baseUser } from "./fixtures";

it("defaults to notifications enabled", () => {
  expect(baseUser.preferences.notifications).toBe(true); // always passes
});
```

**Even better long-term pattern:** use a factory function instead of a shared static object, so every test gets a brand-new object automatically — no cloning needed at the call site at all.

```js
// fixtures.js
export function createBaseUser(overrides = {}) {
  return {
    id: 1,
    name: "Alice",
    preferences: { theme: "dark", notifications: true },
    ...overrides,
  };
}

// test file
const user = createBaseUser(); // fresh object every call, nothing shared
```

**Interview line:** "Shared test fixtures are a classic source of flaky, order-dependent tests, because a shallow copy or spread still shares nested objects by reference. If one test mutates a nested property on a 'copy,' it's actually mutating the original shared fixture, which then leaks into every other test that imports it. The fix is either to deep clone the fixture before mutating it — `structuredClone()` is perfect for this — or better yet, use a factory function that returns a brand-new object on every call so there's nothing to accidentally share in the first place."

---

## 7. Interview Q&A Script

**Q: How do you deep clone an object without a library?**
> "The modern built-in answer is `structuredClone()` — it handles nested objects, arrays, `Date`, `Map`, `Set`, and even circular references correctly. Before that existed, people used `JSON.parse(JSON.stringify(obj))`, but that's lossy: it drops functions and `undefined` values, converts dates to strings, and throws on circular references. If I needed to show the underlying mechanics, I'd write a small recursive function: return primitives as-is, and for objects/arrays, recursively clone every property so nested structures get independent copies instead of shared references."

**Q: Difference between shallow and deep copy — why does it matter for test data setup?**
> "A shallow copy only creates new references for the top-level properties — any nested object or array is still shared with the original. A deep copy recursively copies every level, so there's no shared state anywhere. For test data, this matters because if you shallow-copy a shared fixture and then mutate a nested property, you're actually mutating the original fixture object, which can leak into other tests and cause order-dependent, flaky failures. Deep cloning (or better, a factory function that builds a fresh object per test) avoids that entirely."

**Q: Does `Object.freeze()` create a fully immutable object?**
> "No — it's shallow. It prevents reassignment or addition/removal of top-level properties, but any nested object inside it is still fully mutable. To get deep immutability you need a recursive freeze, or a dedicated immutability library."

**Q: What's the difference between spread and rest syntax?**
> "They use the same `...` syntax but do opposite things depending on context. Spread expands an array or object into individual elements — used when constructing something new, like merging objects or arrays. Rest collects multiple elements into a single array or object — used when receiving something, like in destructuring or function parameters."

**Q: If I do `{ ...obj1, ...obj2 }`, which one wins on a key conflict?**
> "The later spread wins — object spread applies left to right, so if both objects have the same key, whatever comes last in the spread order overwrites the earlier value. That's the standard pattern for merging a default config object with user-provided overrides."

---

## 8. One-Page Cheat Sheet

- **Destructuring:** unpack object properties by key (with renaming/defaults) or array elements by position. Great for function parameters — self-documents what a function expects.
- **Spread (`...`)** expands (creating something new); **rest (`...`)** collects (receiving something). Same syntax, opposite direction, context tells them apart.
- **Shallow copy** (`{ ...obj }`, `Object.assign()`, `[...arr]`) only copies top-level keys — nested objects/arrays are still shared references.
- **Deep copy** recursively copies every level — no shared references anywhere. Best modern tool: `structuredClone()`.
- **`JSON.parse(JSON.stringify())`** is a lossy deep-clone trick: drops functions/`undefined`/symbols, turns `Date` into a string, throws on circular refs.
- **`Object.freeze()`** is shallow — top-level immutable, nested objects still mutable. Need a recursive `deepFreeze` for true deep immutability.
- **Testing rule:** never let tests share a mutable object across files/tests. Deep clone shared fixtures (`structuredClone`) or, better, use a factory function that returns a brand-new object every call — this eliminates order-dependent, flaky test failures caused by accidental shared-reference mutation.
