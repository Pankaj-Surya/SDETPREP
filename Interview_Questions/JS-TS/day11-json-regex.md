# Day 11 — JSON & Regex

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. `JSON.parse` / `JSON.stringify` — the Parts People Forget

```js
const data = { name: "Alice", age: 30, active: true };

const jsonString = JSON.stringify(data);
console.log(jsonString); // '{"name":"Alice","age":30,"active":true}'

const parsed = JSON.parse(jsonString);
console.log(parsed.name); // "Alice"
```

Basic usage is everyone's starting point — the stuff worth knowing well beyond that:

### `stringify` silently drops things

```js
const obj = {
  name: "Alice",
  greet: function () { return "hi"; }, // functions are dropped
  nothing: undefined,                   // undefined properties are dropped
  id: Symbol("id"),                     // symbols are dropped
};

console.log(JSON.stringify(obj)); // '{"name":"Alice"}' — only name survives
```

**Interview line:** "`JSON.stringify` only serializes data that has a real JSON equivalent — functions, `undefined` values, and symbols are silently skipped rather than causing an error. That's worth knowing because it can hide bugs — if you expect a field in your serialized output and it's missing, check whether the source value was actually `undefined` or a function before assuming the serialization logic is broken."

### `stringify`'s third argument — pretty-printing

```js
console.log(JSON.stringify(data, null, 2));
// {
//   "name": "Alice",
//   "age": 30,
//   "active": true
// }
```

Genuinely useful for debug logs and snapshot files, not just cosmetic.

### `stringify`'s second argument — a replacer, for filtering or transforming

```js
const user = { name: "Alice", password: "secret123", role: "admin" };

const safeJson = JSON.stringify(user, (key, value) => {
  if (key === "password") return undefined; // drop sensitive fields before logging
  return value;
});

console.log(safeJson); // '{"name":"Alice","role":"admin"}'
```

**Real use case:** scrubbing sensitive fields (passwords, tokens, API keys) before writing a request/response object to a test log or CI artifact.

### `parse`'s second argument — a reviver, for transforming while parsing

```js
const json = '{"createdAt":"2024-01-15T10:00:00Z","name":"Alice"}';

const parsed = JSON.parse(json, (key, value) => {
  if (key === "createdAt") return new Date(value); // convert the string back into a real Date
  return value;
});

console.log(parsed.createdAt instanceof Date); // true
```

**Interview line worth adding:** "A common gotcha — `JSON.stringify` turns `Date` objects into ISO strings, and `JSON.parse` doesn't automatically turn them back. If a test is comparing a parsed API response's date field against a `Date` object, that comparison will fail unless you convert it back explicitly, either manually or with a reviver function like this."

### `parse` throws on invalid JSON — always wrap it

```js
try {
  JSON.parse("{ invalid json");
} catch (error) {
  console.log("Malformed JSON:", error.message);
}
```

Worth saying out loud: a huge number of "random" test failures in API testing come down to exactly this — a server returning an HTML error page (like a 502) instead of JSON, and `JSON.parse` throwing an unhelpful `SyntaxError: Unexpected token <` because it tried to parse `<html>...` as JSON.

---

## 2. How Do You Compare Two JSON Objects for Equality? (direct interview question)

The honest answer has layers — give the short one first, then show you understand why it's not that simple.

**Short, wrong-ish answer:** `JSON.stringify(a) === JSON.stringify(b)`.

```js
const a = { name: "Alice", age: 30 };
const b = { age: 30, name: "Alice" }; // same data, different key order

console.log(JSON.stringify(a) === JSON.stringify(b)); // false! key order affects the string
```

**Why this is a real trap, not a nitpick:** `JSON.stringify` preserves key insertion order. Two objects with identical data but keys inserted in a different order produce different strings, so this "trick" gives false negatives constantly — especially with API responses, where key order isn't guaranteed to be consistent between calls.

**The actual correct approach: deep equality, not string comparison.**

```js
// Using a testing library's built-in deep equality (the right tool, most of the time)
expect(a).toEqual(b); // true — Jest's toEqual does real structural comparison, order-independent

// Node's built-in option
const assert = require("assert");
assert.deepStrictEqual(a, b); // passes
```

**If you genuinely need to write it yourself** (a fair ask in an interview, to test whether you understand what "deep equality" actually means):

```js
function deepEqual(a, b) {
  if (a === b) return true; // handles primitives and same-reference objects

  if (typeof a !== "object" || typeof b !== "object" || a === null || b === null) {
    return false; // one is a primitive/null and they weren't === above, so not equal
  }

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);

  if (keysA.length !== keysB.length) return false;

  return keysA.every((key) => deepEqual(a[key], b[key])); // recurse into each property
}

console.log(deepEqual(a, b)); // true — correct, regardless of key order
```

**Interview line:** "`JSON.stringify` comparison looks tempting because it's one line, but it's actually comparing serialized text, not data — key order, which JSON.stringify preserves, makes it unreliable for real-world objects like API responses. The correct approach is a real deep-equality check that recursively compares keys and values regardless of order — which is exactly what `toEqual` or `deepStrictEqual` do, and exactly why those exist instead of everyone just using `JSON.stringify` comparisons."

---

## 3. Regex — the Patterns That Actually Come Up

### Writing a regex to validate an email (the classic ask)

The honest, senior answer starts with a caveat, then gives something practical:

```js
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

emailRegex.test("alice@test.com");     // true
emailRegex.test("alice.smith@co.uk");  // true
emailRegex.test("not-an-email");       // false
emailRegex.test("missing@domain");     // false — no dot after @
```

Breaking it down, piece by piece, is what interviewers actually want to hear:

- `^` and `$` — anchor the match to the start and end of the whole string, so `"alice@test.comXYZ"` doesn't partially match.
- `[^\s@]+` — one or more characters that are **not** whitespace and **not** `@` (the local part, before the `@`).
- `@` — a literal `@`.
- `[^\s@]+` — again, one or more non-whitespace, non-`@` characters (the domain).
- `\.` — a literal dot (escaped, since `.` normally means "any character" in regex).
- `[^\s@]+` — the part after the dot (the TLD, like `com` or `co.uk`'s `uk`).

**The important thing to say out loud, unprompted:** "The real RFC spec for valid email addresses is famously complicated — technically things like quoted strings and unusual characters are allowed. In practice, nobody validates emails with a regex that covers 100% of the spec; you validate for 'reasonably well-formed' with something like this, and the real confirmation that an email is genuinely valid and reachable is sending a verification email, not a regex. I'd rather give a simple, readable pattern that catches real mistakes than a 200-character regex nobody can maintain."

### Phone number validation

```js
// A reasonably flexible US-style pattern: optional country code, optional separators
const phoneRegex = /^(\+1[\s-]?)?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}$/;

phoneRegex.test("555-123-4567");     // true
phoneRegex.test("(555) 123-4567");   // true
phoneRegex.test("+1 555-123-4567");  // true
phoneRegex.test("5551234567");       // true
phoneRegex.test("555-12-34567");     // false — wrong grouping
```

**Interview line:** "Phone formats vary enormously by country, so I'd always clarify what formats actually need to be accepted before writing this for real — a global product can't realistically use one regex for every country's phone format. This pattern covers common US formatting variations, which is usually what's actually being asked for unless stated otherwise."

### Date validation (format-level, not calendar-aware)

```js
// YYYY-MM-DD format check
const dateRegex = /^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$/;

dateRegex.test("2024-01-15"); // true
dateRegex.test("2024-13-01"); // false — month 13 doesn't exist, caught by (0[1-9]|1[0-2])
dateRegex.test("2024-02-30"); // true — ⚠️ matches the FORMAT, but Feb 30 isn't a real date!
```

**The important point to raise yourself here:** "This regex validates the *shape* of a date string — correct number of digits, a month between 01 and 12, a day between 01 and 31 — but it can't know that February only has 28 or 29 days, or that April has 30, not 31. Regex is fundamentally pattern matching on text, not calendar logic. For actually validating that a date is real, I'd parse it with something like `Date` or a library like `date-fns`/`dayjs` and check the result, rather than trying to encode calendar rules into a regex."

```js
// A more complete validity check, combining the regex with actual date logic
function isValidDate(dateString) {
  if (!dateRegex.test(dateString)) return false;
  const date = new Date(dateString);
  return date.toISOString().slice(0, 10) === dateString; // round-trips back to the same string if truly valid
}

isValidDate("2024-02-30"); // false — correctly rejected now
isValidDate("2024-02-29"); // true — 2024 is a leap year
```

---

## 4. Test Tie-In: Validating API Response Schemas and Parsing Config/Test Data

This is where JSON and regex stop being separate topics and start showing up together, constantly, in real API test suites.

### Validating a response "shape" without a schema library

```js
function validateUserResponse(response) {
  const errors = [];

  if (typeof response.id !== "number") errors.push("id must be a number");
  if (typeof response.email !== "string" || !emailRegex.test(response.email)) {
    errors.push("email is missing or invalid");
  }
  if (!["active", "inactive", "pending"].includes(response.status)) {
    errors.push(`status must be one of active/inactive/pending, got "${response.status}"`);
  }

  return errors;
}

const response = { id: 1, email: "not-an-email", status: "unknown" };
const errors = validateUserResponse(response);
console.log(errors);
// ["email is missing or invalid", "status must be one of active/inactive/pending, got \"unknown\""]
```

**Worth mentioning proactively:** "For anything beyond a handful of fields, I'd reach for a real schema validation library — `zod`, `ajv`, or `joi` — instead of hand-rolling checks like this. They give you one declarative schema, better error messages out of the box, and they handle nested objects and arrays properly, which gets messy fast if you're doing it manually. I'd only hand-write validation like this for something quick and small, or in an interview setting where the point is demonstrating understanding, not production code."

```js
// The same validation with zod — worth mentioning even if you can't install it live in an interview
import { z } from "zod";

const userSchema = z.object({
  id: z.number(),
  email: z.string().email(),
  status: z.enum(["active", "inactive", "pending"]),
});

const result = userSchema.safeParse(response);
if (!result.success) {
  console.log(result.error.issues); // structured, detailed errors for every failing field
}
```

### Parsing a config/test data file safely

```js
import fs from "fs";

function loadTestConfig(path) {
  let raw;
  try {
    raw = fs.readFileSync(path, "utf-8");
  } catch (error) {
    throw new Error(`Could not read config file at ${path}: ${error.message}`);
  }

  let config;
  try {
    config = JSON.parse(raw);
  } catch (error) {
    throw new Error(`Config file at ${path} is not valid JSON: ${error.message}`);
  }

  if (!config.baseUrl || !config.timeout) {
    throw new Error(`Config file at ${path} is missing required fields (baseUrl, timeout)`);
  }

  return config;
}
```

**Interview line:** "I always wrap both the file read and the `JSON.parse` separately, with their own error messages, rather than one generic `try/catch` around the whole thing. If a teammate accidentally commits a config file with a trailing comma or a missing bracket, the error message should say exactly that — 'this file isn't valid JSON' — instead of some unrelated downstream error like 'cannot read property baseUrl of undefined,' which sends you looking in the wrong place entirely."

### Using regex to sanity-check extracted data before using it in a test

```js
async function getOrderIdFromConfirmation(page) {
  const confirmationText = await page.textContent("#order-confirmation");
  const match = confirmationText.match(/Order #(\d{6,})/);

  if (!match) {
    throw new Error(
      `Could not extract order ID from confirmation text: "${confirmationText}"`
    );
  }

  return match[1]; // the captured group — just the digits
}
```

**Interview line:** "When I'm pulling structured data out of page text or a log line with regex, I always check whether the match actually succeeded before using it — `match` returns `null` if nothing matches, and blindly doing `match[1]` on that throws a confusing 'cannot read properties of null' error that doesn't say what actually went wrong. Throwing a clear error with the actual text that failed to match saves a lot of time debugging a flaky test later."

### Comparing JSON responses between two environments (a real testing use case)

```js
function compareResponses(staging, production, ignoreFields = []) {
  const clean = (obj) => {
    const copy = structuredClone(obj);
    ignoreFields.forEach((field) => delete copy[field]);
    return copy;
  };

  return JSON.stringify(clean(staging)) === JSON.stringify(clean(production));
  // NOTE: still has the key-order caveat from section 2 —
  // fine here only if both responses come from the same consistent serializer
}
```

**Worth saying if you use this pattern:** "This is a quick way to diff two environment responses after stripping fields expected to differ, like timestamps or request IDs. I'd call out that it still has the key-order caveat from before — it's reliable here mainly because both responses are typically serialized the same way by the same API, so key order is consistent between them. For anything where that's not guaranteed, I'd use a real deep-equality comparison instead."

---

## 5. Interview Q&A Script

**Q: Write a regex to validate an email.**
> *(Write `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`, explain each piece out loud, then add the caveat about RFC complexity and that real validation ultimately needs a confirmation email, not just a regex — this caveat is what separates a senior answer from a junior one.)*

**Q: How do you compare two JSON objects for equality?**
> "Not with `JSON.stringify(a) === JSON.stringify(b)` — that compares serialized text, and `JSON.stringify` preserves key order, so two objects with identical data but different key insertion order would incorrectly come back as not equal. The right approach is a real deep-equality check — recursively comparing keys and values regardless of order — which is exactly what `toEqual` in Jest or `assert.deepStrictEqual` in Node already do. I'd reach for those rather than hand-roll it in real code, but I can write a basic recursive version if needed to show the mechanics."

**Q: What does `JSON.stringify` do with `undefined`, functions, or symbols in an object?**
> "It silently drops them — they just don't appear in the output at all, rather than causing an error or serializing as `null`. That's worth knowing because a missing field in a serialized object might mean the source value was `undefined` or a function, not that the serialization logic itself is broken."

**Q: How would you validate that a date string, like `"2024-02-30"`, is actually a real date?**
> "A regex can validate the *format* — four digits, a valid-looking month and day range — but it can't know real calendar rules, like February having 28 or 29 days. I'd combine the format regex with an actual date check — parse it with `Date` and confirm it round-trips back to the exact same string, which catches invalid dates a regex alone would miss."

**Q: How would you validate an API response's structure in a test?**
> "For a handful of fields, straightforward manual checks are fine — type checks, a regex for an email, an allowed-values list for an enum-like field. For anything larger or used across many tests, I'd use a schema validation library like `zod` or `ajv` instead — one declarative schema, consistent and detailed error messages, and it handles nested structures properly without hand-written recursive checks."

---

## 6. One-Page Cheat Sheet

- **`JSON.stringify`** silently drops `undefined`, functions, and symbols. Second argument is a replacer (filter/transform fields); third is indentation for pretty-printing.
- **`JSON.parse`** throws on invalid input — always wrap it, and give the error a clear message, since a non-JSON response (like an HTML error page) is a very common real-world cause.
- **Dates don't round-trip automatically** — `stringify` turns a `Date` into a string, `parse` doesn't turn it back; use a reviver function or convert manually.
- **Never compare JSON objects with `JSON.stringify(a) === JSON.stringify(b)`** — key order breaks it. Use `toEqual`/`deepStrictEqual`, or a real recursive deep-equality function.
- **Email regex:** `/^[^\s@]+@[^\s@]+\.[^\s@]+$/` — good enough for "reasonably well-formed," not a full RFC validator. Always mention that real validation needs a confirmation email, not just a pattern.
- **Regex validates format, not meaning** — a date regex can check digit shape and ranges but can't know February has 28/29 days; combine with real date parsing for true validity.
- **Testing rule:** hand-rolled validation is fine for a few fields; reach for `zod`/`ajv`/`joi` for anything larger. Always check `match !== null` before using a captured regex group, and throw a clear error including the actual text that failed to match. Wrap file-read and `JSON.parse` separately in config loaders so failures point to the real problem.
