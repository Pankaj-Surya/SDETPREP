# Day 18 — TS Basics & Why TypeScript

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. What TypeScript Actually Is

TypeScript is **JavaScript plus a static type system**. Every valid JS file is already valid TS; TS just lets you describe what shape your data and functions are supposed to have, and checks that description **before the code runs**.

```ts
function add(a: number, b: number): number {
  return a + b;
}

add(2, 3);     // ✅
add("2", 3);   // ❌ Compile error: Argument of type 'string' is not assignable to parameter of type 'number'
```

The same mistake in plain JS would silently produce `"23"` (Day 2's coercion) and surface much later, somewhere unrelated.

**Static vs dynamic typing, in one line:** in JS, types are checked **at runtime**, when the line actually executes. In TS, types are checked **at compile time**, before anything runs — so a whole category of bugs gets caught in your editor or in CI, not in production or at minute 40 of a regression run.

---

## 2. Compiling TS → JS — and the Point People Miss

Browsers and Node don't run TypeScript. A compile step turns `.ts` into `.js`:

```ts
// input.ts
function greet(name: string): string {
  return `Hello, ${name}`;
}
```

```js
// output.js — the types are simply gone
function greet(name) {
  return `Hello, ${name}`;
}
```

This is called **type erasure**, and it's the single most important thing to understand about TS:

> **Types exist only at compile time. At runtime there are no types, no checks — just JavaScript.**

**Consequences worth saying out loud in an interview:**

- TS **cannot protect you from bad data arriving at runtime.** If an API returns `{ id: "abc" }` but you typed it as `{ id: number }`, nothing stops it. The type was a promise you made, not a check the program performs.
- You can't do `typeof SomeInterface` or `if (x instanceof SomeInterface)` — interfaces and type aliases don't exist in the output.
- Type errors and emit are separate: by default `tsc` still produces JS even when there are type errors (unless `noEmitOnError` is set).

### The commands

```bash
npx tsc              # compile according to tsconfig.json, emit .js files
npx tsc --noEmit     # type-check only, produce no files (very common in CI)
npx tsc --watch      # recompile on change
```

Many tools **strip types without checking them** — `tsx`, esbuild/SWC-based runners, Vitest, and Playwright's own TypeScript transform all do this for speed. That means a test run can succeed even when the project has type errors. The standard fix:

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "test": "playwright test"
  }
}
```

and run `typecheck` as a separate CI step. **Worth saying in an interview:** "Running tests and type-checking are two different things — I make sure CI does both."

### A minimal `tsconfig.json` for a test project

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "noEmit": true,
    "types": ["node"]
  },
  "include": ["tests/**/*.ts", "pages/**/*.ts", "utils/**/*.ts"]
}
```

**`"strict": true` is the line that matters.** It turns on a bundle of checks — notably `strictNullChecks`, `noImplicitAny`, and `useUnknownInCatchVariables`. Without it, TS is much more permissive and you lose most of the value. Always start new projects strict.

---

## 3. The Basic Types

```ts
const name: string = "Alice";
const age: number = 30;          // one number type — no int/float split
const isActive: boolean = true;

const tags: string[] = ["qa", "automation"];       // array of strings
const scores: Array<number> = [90, 85];            // same thing, different syntax
const pair: [string, number] = ["Alice", 30];      // tuple — fixed length and position types
```

### Type inference — you don't annotate everything

```ts
const count = 5;              // TS infers: number
const items = ["a", "b"];     // TS infers: string[]

function double(n: number) {  // return type inferred as number
  return n * 2;
}
```

**Senior-level habit:** let inference do the work for local variables, and **explicitly annotate function parameters and public return types** — that's where the contract lives.

### Union types and literal types — where TS gets really useful

```ts
let id: string | number;       // either one
id = "abc";
id = 123;

type Status = "pending" | "passed" | "failed";   // only these exact strings allowed
let result: Status = "passed";
result = "done";               // ❌ Type '"done"' is not assignable to type 'Status'
```

Compare that with JS, where `"Passed"` vs `"passed"` is a silent bug waiting to happen.

### `null`, `undefined`, and `strictNullChecks`

With `strict` on, `null` and `undefined` are **not** assignable to other types — you have to say so explicitly:

```ts
let name: string = null;            // ❌ error under strict
let maybe: string | null = null;    // ✅ explicitly allowed

const users = [{ id: 1, name: "Alice" }];
const user = users.find((u) => u.id === 99);   // type: { id: number; name: string } | undefined

console.log(user.name);        // ❌ 'user' is possibly 'undefined'
console.log(user?.name);       // ✅ optional chaining
if (user) console.log(user.name); // ✅ narrowed
```

**This is a big deal in test code.** `find`, `match` (Day 11), `Map.get`, and optional fields all return "might not exist" types — TS forces you to handle the missing case instead of crashing with `Cannot read properties of undefined`.

### Functions — optional params, defaults, and `void`

```ts
function createUser(name: string, role: string = "guest", age?: number): { name: string; role: string; age?: number } {
  return { name, role, age };
}

function logMessage(message: string): void {
  console.log(message);   // void = returns nothing useful
}

async function fetchUser(id: number): Promise<{ id: number; name: string }> {
  // an async function's return type is always Promise<...>
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}
```

### Objects and interfaces (a first look)

```ts
interface User {
  id: number;
  name: string;
  email: string;
  nickname?: string;     // optional
  readonly createdAt: Date; // can't be reassigned
}

function sendWelcome(user: User) { /* ... */ }

sendWelcome({ id: 1, name: "Alice" });
// ❌ Property 'email' is missing in type '{ id: number; name: string; }'
```

---

## 4. `any` vs `unknown` — the Core Interview Question

Both can hold *any value*. The difference is what you're allowed to **do** with it.

### `any` — switches the type checker off

```ts
let a: any = "hello";

a.toFixed(2);          // ✅ compiles — but crashes at runtime: a.toFixed is not a function
a.foo.bar.baz();       // ✅ compiles — crashes at runtime
const n: number = a;   // ✅ compiles — n is actually a string
```

`any` is **contagious**: anything you derive from an `any` is also `any`, so one careless `any` can quietly disable checking across a whole chain of code.

### `unknown` — "I don't know yet, so make me prove it"

```ts
let u: unknown = "hello";

u.toUpperCase();        // ❌ 'u' is of type 'unknown'
const n: number = u;    // ❌ Type 'unknown' is not assignable to type 'number'

// You must NARROW it first:
if (typeof u === "string") {
  u.toUpperCase();      // ✅ here TS knows u is a string
}
```

**The one-sentence answer:** "`any` opts out of type checking — you can do anything with it and TS won't object, so errors surface at runtime. `unknown` is the type-safe counterpart: it can also hold anything, but TS won't let you use it until you've narrowed it with a check, so the checking is forced rather than skipped."

### Where `unknown` shows up for real

**1. `catch` clauses.** Under `strict`, the caught value is `unknown` — because in JS anyone can `throw` anything (Day 9):

```ts
try {
  await page.click("#submit");
} catch (error) {                      // error: unknown
  console.log(error.message);          // ❌ can't assume it has a message

  if (error instanceof Error) {
    console.log(error.message);        // ✅ narrowed
  } else {
    console.log("Non-Error thrown:", String(error));
  }
}
```

**2. Parsed JSON and API responses.** `JSON.parse()` and `response.json()` return **`any`** — an easy way for unchecked data to leak in. Treat them as `unknown`:

```ts
const raw: unknown = JSON.parse(text);   // annotating as unknown forces you to check

function isUser(value: unknown): value is User {   // a "type guard"
  return (
    typeof value === "object" &&
    value !== null &&
    typeof (value as User).id === "number" &&
    typeof (value as User).email === "string"
  );
}

if (isUser(raw)) {
  console.log(raw.email);   // ✅ safely typed as User here
}
```

For anything beyond a couple of fields, a schema library like `zod` (Day 11) does this validation *and* produces the type for you — the proper bridge between "types vanish at runtime" and "I need runtime safety":

```ts
import { z } from "zod";

const UserSchema = z.object({ id: z.number(), email: z.string().email() });
type User = z.infer<typeof UserSchema>;     // the type is derived from the schema

const user: User = UserSchema.parse(await res.json());   // throws if the data doesn't match
```

### Related traps worth naming

- **Type assertions (`as`) are not checks.** `const user = data as User` tells TS "trust me" — if you're wrong, nothing catches it at runtime. It's a controlled use of `any` in disguise.
- **Generics on HTTP clients are assertions too.** `api.get<User>("/users/1")` *claims* the response is a `User`; it does not verify it.
- **The non-null assertion `!`** (`user!.name`) silences "possibly undefined" without handling it. Fine occasionally, a smell when it's everywhere.
- **`@ts-ignore` / `@ts-expect-error`:** prefer `@ts-expect-error` — it fails if the error goes away, so stale suppressions get cleaned up.

**Practical rule to state:** "I treat `any` as a last resort and turn on `noImplicitAny` so it can't sneak in by accident. When I genuinely don't know a type, I use `unknown` and narrow."

---

## 5. Why Would a QA Team Choose TypeScript Over JavaScript?

Give a **balanced** answer — naming the costs alongside the benefits is exactly what separates a senior answer from a pitch.

### The benefits, in order of real-world impact

**1. Failures move from runtime to edit/compile time.**
In JS, a typo in a page-object method only blows up when that specific test runs — possibly deep into a long nightly run.

```js
// JS — fine until the test executes, then: TypeError: loginPage.logn is not a function
await loginPage.logn("alice", "secret");
```
```ts
// TS — red squiggle immediately in the editor, and tsc fails in CI before any browser launches
await loginPage.logn("alice", "secret");
// ❌ Property 'logn' does not exist on type 'LoginPage'. Did you mean 'login'?
```

**2. Safe refactoring across a big suite.** Rename a method, change a function's parameters, or restructure test data, and the compiler lists *every* call site that broke. In a 500-test JS suite, the same refactor is find-and-replace plus hope.

**3. Autocomplete and self-documenting code.** Page objects, fixtures, and API clients become discoverable: type `loginPage.` and see every action with its parameter types. New team members learn the framework from the editor instead of from reading source.

**4. Typed contracts for test data and API responses.**

```ts
interface Order { id: number; status: "pending" | "shipped" | "cancelled"; total: number }

function createOrder(overrides: Partial<Order> = {}): Order {
  return { id: 1, status: "pending", total: 100, ...overrides };
}

createOrder({ status: "shiped" });   // ❌ caught — typo'd status can't reach the test run
```

**5. It forces you to handle the "might be missing" case.** `strictNullChecks` surfaces the `undefined` that would otherwise appear as a flaky `Cannot read properties of undefined` in CI.

**6. Ecosystem fit.** Playwright is TypeScript-first, with typed fixtures and locators; Cypress, WebdriverIO, and most modern test tooling ship their own type definitions. Using TS means better editor support from the tools you already use.

**7. Better static analysis.** With type information, linters like `typescript-eslint` can enforce `no-floating-promises` — catching the missing `await` bug from Day 15, which plain JS linting can't reliably do.

### The honest costs

- **A build/config step and a learning curve.** `tsconfig`, type definitions, and generics take time, especially for team members new to typed languages.
- **Types are not runtime validation.** An API can still return something that doesn't match your interface — you still need schema validation for real safety.
- **Third-party libraries sometimes lack good types** (you install `@types/...` packages, or occasionally write declarations yourself).
- **`any` erodes the benefit.** A codebase littered with `any` and `as` gets the overhead of TS with little of the protection — it needs team discipline and strict settings.
- **Slower feedback if misconfigured** — a heavyweight type-check in the hot path can annoy people; usually solved by running `tsc --noEmit` separately from test execution.

### The answer to give

> "I'd pick TypeScript for a test framework that's going to live and grow. The main win is moving failures earlier — a typo in a page-object method or a wrong argument type shows up in the editor and in CI before any browser launches, instead of halfway through a nightly run. It makes refactoring a large suite safe because the compiler lists every broken call site, and typed page objects, fixtures, and API models give you autocomplete and self-documenting contracts, which helps onboarding. It also fits the ecosystem — Playwright is TypeScript-first. The trade-offs are the setup and learning curve, and the fact that types disappear at runtime, so I'd still validate API responses with something like `zod` and keep `strict` on and `any` out. For a tiny throwaway script, plain JS is perfectly fine; for a shared framework, TS pays for itself."

---

## 6. Test Tie-In: A Small Typed Slice of a Framework

Everything from the last few days, with types added.

```ts
// types.ts
export interface TestUser {
  username: string;
  password: string;
  role: "admin" | "editor" | "viewer";
}

// pages/LoginPage.ts
import type { Page } from "@playwright/test";
import type { TestUser } from "../types";

export class LoginPage {
  constructor(private readonly page: Page) {}   // "private readonly" declares and assigns the field in one go

  async login(user: Pick<TestUser, "username" | "password">): Promise<void> {
    await this.page.fill("#username", user.username);
    await this.page.fill("#password", user.password);
    await this.page.click("#submit");
  }

  async getErrorMessage(): Promise<string | null> {
    return this.page.textContent("#error-message");   // may be null — the type says so
  }
}
```

```ts
// utils/dataFactory.ts
import type { TestUser } from "../types";

export function createUser(overrides: Partial<TestUser> = {}): TestUser {
  return { username: "test.user", password: "Passw0rd!", role: "viewer", ...overrides };
}
```

```ts
// tests/login.spec.ts
import { test, expect } from "@playwright/test";
import { LoginPage } from "../pages/LoginPage";
import { createUser } from "../utils/dataFactory";

test("invalid password shows an error", async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.login(createUser({ password: "wrong" }));

  const message = await loginPage.getErrorMessage();   // string | null
  expect(message).toContain("Invalid credentials");

  createUser({ role: "superuser" });   // ❌ compile error — not one of the allowed roles
});
```

**What to point out:** `Partial<TestUser>` makes every field optional for overrides; `Pick<...>` narrows a type to just the fields a function needs; the `string | null` return type forces callers to consider `null`; and the invalid role is rejected before the test ever runs.

---

## 7. Interview Q&A Script

**Q: Why would a QA team choose TS over JS?**
> *(Use the balanced answer from section 5 — lead with "failures move earlier", then refactoring safety, typed contracts/autocomplete, and ecosystem fit; then volunteer the costs: setup, types vanish at runtime so still validate responses, and `any` erodes the value.)*

**Q: What's the difference between `any` and `unknown`?**
> "`any` disables type checking — you can call anything on it and TS stays quiet, so mistakes surface at runtime, and it spreads to anything derived from it. `unknown` also accepts any value, but I can't use it until I narrow it with a check like `typeof`, `instanceof`, or a type guard. So `unknown` is the safe way to say 'I don't know the type yet'. It's what `catch` variables are under `strict`, and how I'd treat parsed JSON."

**Q: Does TypeScript check types at runtime?**
> "No. Types are erased when it compiles to JavaScript, so at runtime there's nothing left to check. That means TS can't validate data coming from an API or a file — for that I need runtime validation like `zod`. It also means interfaces can't be tested with `instanceof` or `typeof`."

**Q: What does `strict: true` give you?**
> "It enables a group of stricter checks — importantly `strictNullChecks`, so `null`/`undefined` must be handled explicitly; `noImplicitAny`, so untyped values don't silently become `any`; and `useUnknownInCatchVariables`, so caught errors are `unknown` and must be narrowed. Without it TS is far more permissive, and you lose most of the value."

**Q: Is `response as User` or `api.get<User>()` safe?**
> "No — both are assertions, not validation. They tell the compiler to trust me, and nothing verifies the data at runtime. If the API shape drifts, tests could pass or fail for confusing reasons. For real safety I'd parse the response through a schema, like `zod`, and derive the type from it."

**Q: Why run `tsc --noEmit` separately from tests?**
> "Many test runners, including Playwright, transpile TypeScript by stripping types without type-checking, for speed. So tests can run fine even when the project has type errors. A separate `tsc --noEmit` step in CI catches those errors."

**Q: How does TS help with the 'maybe undefined' bugs common in tests?**
> "With `strictNullChecks`, methods like `find`, `Map.get`, and `match` are typed as possibly returning `undefined` or `null`, and the compiler won't let me use the result until I handle it with optional chaining or a check. That turns a flaky `Cannot read properties of undefined` in CI into an error I see while writing the code."

---

## 8. One-Page Cheat Sheet

- **TS = JS + static types**, checked at **compile time**; then compiled to plain JS. **Types are erased** — no runtime checks, no `instanceof` on interfaces.
- **Commands:** `tsc` (emit), `tsc --noEmit` (type-check only — run in CI), `tsc --watch`. Fast runners (Playwright, Vitest, `tsx`) strip types **without checking**, so type-check separately.
- **`tsconfig`:** always `"strict": true` (enables `strictNullChecks`, `noImplicitAny`, `useUnknownInCatchVariables`, …).
- **Basics:** `string`, `number`, `boolean`, arrays (`string[]`), tuples, union types (`string | number`), literal types (`"passed" | "failed"`), optional (`age?: number`), `void`, `Promise<T>`. Let inference handle locals; annotate function params and public returns.
- **`strictNullChecks`:** `null`/`undefined` aren't assignable to other types; `find()` returns `T | undefined` — handle with `?.` or a check.
- **`any`** = checker off, contagious, errors at runtime. **`unknown`** = accepts anything, but must be narrowed (`typeof`, `instanceof`, type guard, schema) before use. Prefer `unknown`; `catch` variables are `unknown` under strict.
- **Not checks:** `as` assertions, generics like `api.get<User>()`, and `!` just tell the compiler to trust you. Use `zod` for real runtime validation.
- **Why QA picks TS:** failures caught at edit/compile time, safe refactoring across big suites, autocomplete + self-documenting page objects/fixtures/API models, forced handling of `undefined`, ecosystem fit (Playwright is TS-first), and lint rules like `no-floating-promises`.
- **Honest costs:** setup and learning curve, types ≠ runtime validation, patchy third-party types, `any` erodes benefits. Answer with both sides.
