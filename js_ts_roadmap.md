# JavaScript + TypeScript Roadmap for SDET/QA Interviews
### One topic a day, interview-focused

**How to use this:** Each day = ~1-2 hrs. Learn the concept → write 3-4 small code snippets → answer the "Interview Angle" questions out loud without looking. SDET interviews test JS fundamentals + how you'd apply them in test automation (Playwright/Cypress/WebdriverIO), so each day also has a "Test Automation Tie-in."

---

## PHASE 1: JavaScript Fundamentals (Days 1–12)

### Day 1 — Variables, Scope & Hoisting
- `var` vs `let` vs `const`, block scope vs function scope, temporal dead zone
- **Interview angle:** "What happens if you access a `let` variable before declaration?" "Why avoid `var` in modern code?"
- **Test tie-in:** Why loop variables in `for` loops with async callbacks (e.g., looping over test data) behave differently with `var` vs `let`.

### Day 2 — Data Types & Type Coercion
- Primitives vs reference types, `==` vs `===`, truthy/falsy, `NaN`, `typeof`
- **Interview angle:** "Explain `[] == false`." "Why is `typeof null === 'object'`?"
- **Test tie-in:** Assertion pitfalls — comparing objects/arrays with `===` in test assertions (deep equality libraries exist for a reason).

### Day 3 — Functions Deep Dive
- Function declarations vs expressions, arrow functions vs regular (this-binding), default/rest params, IIFEs
- **Interview angle:** "Why doesn't `this` work the same in arrow functions?" "When would an arrow function break a test helper?"
- **Test tie-in:** Writing reusable test utility/helper functions correctly.

### Day 4 — Objects & Arrays (Deep Dive)
- Object/array destructuring, spread/rest, shallow vs deep copy, `Object.freeze`
- **Interview angle:** "How do you deep clone an object without a library?" "Difference between shallow and deep copy — why does it matter for test data setup?"
- **Test tie-in:** Cloning fixtures/test data objects so tests don't mutate shared state.

### Day 5 — Array Methods (map/filter/reduce/find/etc.)
- `map`, `filter`, `reduce`, `find`, `some`, `every`, `forEach` vs `map`
- **Interview angle:** "Implement `reduce` from scratch." "When would you use `find` vs `filter` in test data validation?"
- **Test tie-in:** Filtering API response arrays, extracting specific fields to assert on.

### Day 6 — Closures
- Lexical scope, closures, private variables via closures
- **Interview angle:** "What is a closure? Give a real example." "Classic loop + `setTimeout` closure bug — walk through it."
- **Test tie-in:** Module pattern for page objects / test config that need private state.

### Day 7 — Prototypes & `this`
- Prototype chain, `Object.create`, `call`/`apply`/`bind`, `this` in different contexts
- **Interview angle:** "How does prototypal inheritance differ from classical inheritance?" "What does `bind` do and when would you use it?"
- **Test tie-in:** Understanding `this` inside Mocha/Jest hooks (`function() {}` vs arrow functions in `beforeEach`).

### Day 8 — Classes & OOP in JS
- `class`, constructors, `extends`, `super`, static methods, getters/setters
- **Interview angle:** "How is a JS class different from a Java/C# class under the hood?"
- **Test tie-in:** Building Page Object Model (POM) classes for Selenium/Playwright — base class + inheritance for common actions.

### Day 9 — Error Handling
- `try/catch/finally`, custom errors, `throw`, error propagation
- **Interview angle:** "How do you handle errors in async code vs sync code?" "Design a custom error class."
- **Test tie-in:** Handling flaky element-not-found errors, custom assertion error messages.

### Day 10 — Modules (ESM vs CommonJS)
- `import/export` vs `require/module.exports`, default vs named exports
- **Interview angle:** "Difference between CommonJS and ES Modules?" "Why might a Node test project need `"type": "module"`?"
- **Test tie-in:** Structuring a test framework (page objects, utils, config) into reusable modules.

### Day 11 — JSON & Regex
- `JSON.parse/stringify`, common regex patterns (email, phone, date validation)
- **Interview angle:** "Write a regex to validate an email." "How do you compare two JSON objects for equality?"
- **Test tie-in:** Validating API response schemas, parsing config/test data files.

### Day 12 — Timers & Event Loop (Intro)
- `setTimeout`, `setInterval`, call stack, task queue (basic mental model)
- **Interview angle:** "Explain the JS event loop in your own words." "Why doesn't `setTimeout(fn, 0)` run immediately?"
- **Test tie-in:** Why hard-coded `sleep()`/waits are bad practice vs explicit/implicit waits.

---

## PHASE 2: Asynchronous JavaScript (Days 13–17)
*(This is the single most-tested area for SDETs — Playwright/Cypress are async-heavy.)*

### Day 13 — Callbacks & Callback Hell
- Callback pattern, why it gets messy, error-first callbacks
- **Interview angle:** "What's callback hell and how do you avoid it?"
- **Test tie-in:** Legacy WebdriverIO/Selenium callback-style code you may encounter.

### Day 14 — Promises
- Promise states, `.then/.catch/.finally`, chaining, `Promise.all/allSettled/race/any`
- **Interview angle:** "Difference between `Promise.all` and `Promise.allSettled`?" "What happens if one promise in `Promise.all` rejects?"
- **Test tie-in:** Running multiple independent API validations in parallel, waiting on multiple element states.

### Day 15 — Async/Await
- `async/await` syntax, error handling with try/catch, sequential vs parallel awaits
- **Interview angle:** "Why can awaiting in a loop be a performance problem?" "Convert this promise chain to async/await."
- **Test tie-in:** This IS Playwright syntax — `await page.click()`, `await expect(locator).toBeVisible()`. Know it cold.

### Day 16 — Event Loop Deep Dive (Microtasks vs Macrotasks)
- Microtask queue (Promises) vs macrotask queue (setTimeout), execution order puzzles
- **Interview angle:** "Predict the console.log output order" (classic setTimeout + Promise + sync code puzzle — practice 3-4 of these)
- **Test tie-in:** Debugging why an assertion runs before a UI update finishes.

### Day 17 — Fetch API & HTTP Basics in JS
- `fetch`, async API calls, handling responses/errors, intro to axios
- **Interview angle:** "How do you handle a failed fetch request?" "Difference between `fetch` throwing and returning `ok: false`?"
- **Test tie-in:** Writing API tests directly in JS/TS (supertest, axios) — a very common SDET interview task.

---

## PHASE 3: TypeScript (Days 18–26)

### Day 18 — TS Basics & Why TypeScript
- Static typing, compiling TS→JS, basic types (`string`, `number`, `boolean`, `any`, `unknown`)
- **Interview angle:** "Why would a QA team choose TS over JS?" "Difference between `any` and `unknown`?"

### Day 19 — Interfaces & Type Aliases
- `interface` vs `type`, optional properties, readonly, extending interfaces
- **Interview angle:** "When would you use `type` over `interface`?" "Can you extend a `type` like an `interface`?"
- **Test tie-in:** Typing test data fixtures / API response shapes.

### Day 20 — Functions & Generics
- Typed function params/returns, optional/default params, generics basics (`<T>`)
- **Interview angle:** "Write a generic function that returns the first element of any array." "Why use generics instead of `any`?"
- **Test tie-in:** Generic wait/retry helper functions, generic API response wrappers.

### Day 21 — Enums, Tuples & Union/Intersection Types
- `enum`, tuples, union (`|`) and intersection (`&`) types, literal types
- **Interview angle:** "When would you use an enum vs a union of string literals?" "What's a discriminated union?"
- **Test tie-in:** Modeling test environments (`enum Env { DEV, QA, PROD }`), typing different response states.

### Day 22 — Classes in TypeScript
- Access modifiers (`public/private/protected`), abstract classes, implementing interfaces
- **Interview angle:** "Difference between `private` in TS vs `#private` in JS?" "What's an abstract class used for?"
- **Test tie-in:** Building a strongly-typed Page Object base class / BasePage pattern.

### Day 23 — Type Narrowing & Guards
- `typeof`/`instanceof` guards, custom type guards, `as` assertions, non-null assertion (`!`)
- **Interview angle:** "How do you narrow a union type safely?" "Why is overusing `as` risky?"
- **Test tie-in:** Safely handling API responses that could be success or error shapes.

### Day 24 — Utility Types
- `Partial`, `Pick`, `Omit`, `Record`, `Readonly`
- **Interview angle:** "What does `Partial<T>` do and when would you use it?" "Implement `Pick` conceptually."
- **Test tie-in:** Creating partial test data objects for negative test cases, typing config objects with `Record`.

### Day 25 — Modules, Namespaces & Config
- `tsconfig.json` key options (`strict`, `target`, `module`), module resolution
- **Interview angle:** "What does `strict: true` actually enable?" "Why might a test project set `esModuleInterop: true`?"
- **Test tie-in:** Setting up a Playwright/TS project from scratch — expect this as a live-coding task.

### Day 26 — Async Code + Generics in TS (Combining Everything)
- Typing Promises (`Promise<T>`), async function return types, generic API wrapper functions
- **Interview angle:** "Type a function that fetches and returns a typed API response." (Very common live-coding prompt.)
- **Test tie-in:** Writing a typed `apiClient.get<T>(url): Promise<T>` helper — a realistic take-home task.

---

## PHASE 4: Testing-Specific JS/TS (Days 27–32)

### Day 27 — Testing Framework Fundamentals (Jest/Mocha syntax)
- `describe/it/test`, hooks (`beforeEach/afterEach/beforeAll/afterAll`), assertions
- **Interview angle:** "Order of execution across nested `describe` blocks with hooks?"

### Day 28 — Mocking & Spies
- `jest.fn()`, `jest.spyOn()`, mocking modules/API calls, stubs vs mocks vs spies (terminology)
- **Interview angle:** "Difference between a mock, a stub, and a spy?" "How do you mock a fetch call in Jest?"

### Day 29 — Async Testing Patterns
- Testing promises/async functions, `waitFor`, handling flaky async assertions
- **Interview angle:** "How do you test a function that returns a rejected promise?"
- **Test tie-in:** This maps directly to Playwright's auto-waiting `expect` assertions.

### Day 30 — Debugging & Common JS/TS Pitfalls (Review Day)
- Common gotchas: `NaN !== NaN`, floating point math, mutation bugs, `===` vs `==`, `this` traps
- **Interview angle:** Rapid-fire "what's wrong with this code" snippets — do 10-15 of these.

### Day 31 — Design Patterns for Test Automation
- Page Object Model, Singleton (for driver/config), Factory (for test data)
- **Interview angle:** "Why use POM?" "How would you implement a Singleton for a browser instance in TS?"

### Day 32 — Mock Interview / Full Review
- Pick 5 questions from earlier days at random, answer + code them live under time pressure
- Review any topic that felt shaky

---

## Quick Reference: Most Commonly Asked SDET Interview Questions
- `var` vs `let` vs `const` and hoisting
- `==` vs `===` and coercion examples
- Explain closures with a real-world example
- Explain the event loop / microtask vs macrotask
- `Promise.all` vs `Promise.allSettled` vs `Promise.race`
- `async/await` error handling
- `interface` vs `type` in TypeScript
- Generics — write one from scratch
- Deep vs shallow copy
- Mock vs stub vs spy
- Predict-the-output code puzzles (async ordering)
- Live-code: type an API client function, or fix a broken async test

---

**Tip:** Don't just read — for every day, write the code in a scratch file and run it. SDET interviews are heavy on "predict this output" and live small coding, not just theory.
