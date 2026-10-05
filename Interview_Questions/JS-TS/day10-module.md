# Day 10 — Modules (ESM vs CommonJS)

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. The Two Systems, Side by Side

```js
// CommonJS (CJS) — the original Node module system
const fs = require("fs");

function readConfig(path) {
  return fs.readFileSync(path, "utf-8");
}

module.exports = { readConfig };
```

```js
// ES Modules (ESM) — the standard JS module system, now native in Node too
import fs from "fs";

export function readConfig(path) {
  return fs.readFileSync(path, "utf-8");
}
```

They do the same basic job — split code into files and share things between them — but they work differently enough under the hood that mixing them carelessly causes real, confusing bugs. Worth knowing both cold, because most real codebases (especially test frameworks with a few years of history) have at least some of each.

---

## 2. `require`/`module.exports` vs `import`/`export`

**CommonJS:** `require()` is a regular function call, evaluated at **runtime**, synchronously, wherever it appears in the code.

```js
if (process.env.NODE_ENV === "test") {
  const mockDb = require("./mockDb"); // totally fine — conditional require works in CJS
}
```

**ESM:** `import` is not a function call — it's a declaration, processed at **parse time**, before any code actually runs. This is why ESM imports must be at the top level and can't be conditional the same way.

```js
if (process.env.NODE_ENV === "test") {
  import mockDb from "./mockDb"; // ❌ SyntaxError — import must be top-level
}

// If you genuinely need conditional loading in ESM, you need dynamic import instead:
if (process.env.NODE_ENV === "test") {
  const mockDb = await import("./mockDb"); // ✅ returns a Promise, resolved at runtime
}
```

**Interview line:** "`require` is just a function, so you can call it anywhere — inside an `if`, inside a function body, even in a loop. `import` is a static declaration the engine processes before execution starts, which is actually a feature, not a limitation — it lets tools statically analyze your dependency graph for things like tree-shaking, without having to run your code first. If you need the equivalent of a conditional `require` in ESM, you reach for dynamic `import()`, which returns a Promise instead of the value directly."

---

## 3. Default vs Named Exports

```js
// Named exports — export multiple specific things, import them by exact name
export function login(username, password) { /* ... */ }
export function logout() { /* ... */ }
export const BASE_URL = "https://app.com";

import { login, logout, BASE_URL } from "./authUtils.js";
```

```js
// Default export — one "main" thing per file
export default class LoginPage {
  // ...
}

import LoginPage from "./LoginPage.js"; // the name here is up to you, no curly braces needed
```

**The practical difference worth stating clearly:**

- Named exports force the importer to use the exact name you exported (or explicitly rename it with `as`), which makes refactoring safer — your editor can reliably find every usage.
- Default exports let the importer call it whatever they want, which is flexible but means there's no single canonical name tying an import back to its source — two files could `import Page from "./LoginPage.js"` and `import LP from "./LoginPage.js"` and both are "correct."

```js
import { login as doLogin } from "./authUtils.js"; // renaming a named import
```

**A real-world rule worth mentioning if asked which to prefer:** a lot of experienced teams default to named exports almost everywhere, and reserve default exports for files that genuinely export exactly one thing — like a single class representing a page object or a single React component. The main reason: named exports give you better auto-import and refactor support in editors, and they make it obvious at the `import` line exactly what's being pulled in, instead of an arbitrary name the importer made up.

You can mix both in the same file:

```js
export const timeout = 5000;
export default class ApiClient { /* ... */ }

import ApiClient, { timeout } from "./ApiClient.js";
```

---

## 4. CommonJS vs ES Modules — the Core Interview Question

Say these points in roughly this order — it shows you understand the mechanics, not just the syntax difference.

**1. Loading: synchronous vs asynchronous.**
CommonJS loads modules synchronously — `require()` blocks until the file is read and executed. ESM is designed around asynchronous loading from the start, which is part of why it works natively in browsers (where you can't block on a network request the way `require` blocks on a local file read).

**2. Exports: a copied value vs a live binding.**
This is the one that actually surprises people in practice.

```js
// CommonJS — exports a snapshot/copy of the value at export time
// counter.js
let count = 0;
function increment() { count++; }
module.exports = { count, increment };

// main.js
const { count, increment } = require("./counter");
increment();
console.log(count); // 0 — still the OLD copied value, not updated!
```

```js
// ESM — exports a LIVE binding, not a copy
// counter.mjs
export let count = 0;
export function increment() { count++; }

// main.mjs
import { count, increment } from "./counter.mjs";
increment();
console.log(count); // 1 — reflects the live, current value
```

**Interview line:** "In CommonJS, when you destructure a value out of `require()`, you get a copy taken at that exact moment — if the module later changes that value internally, your copy doesn't update. ESM exports are live bindings — the imported name is actually linked to the exporting module's variable, so if it changes there, you see the updated value on your side too. This is a real, practical difference, not just trivia — it's caught people off guard debugging why a counter or a mutable config object 'wasn't updating' when it actually was, just not on their copy."

**3. `this` at the top level.**
In CommonJS, `this` at the top of a file refers to `module.exports`. In ESM, top-level `this` is `undefined`. Minor, but a real gotcha if old CJS code relies on it.

**4. `__dirname`/`__filename`.**
Available automatically in CommonJS. Not available in ESM — you have to reconstruct them:

```js
import { fileURLToPath } from "url";
import { dirname } from "path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

**5. File extensions matter more in ESM.**
Node requires either a `.mjs` extension, or `"type": "module"` in `package.json`, to treat `.js` files as ESM at all — by default, Node still assumes CommonJS.

**One-line summary if asked to be brief:** "CommonJS is Node's original, synchronous module system using `require`/`module.exports`, exporting copied values. ESM is the language-standard module system, statically analyzed, asynchronous by design, and exports live bindings instead of copies. Node supports both today, but you need to be explicit about which one a given file or project is using."

---

## 5. Why a Node Test Project Might Need `"type": "module"`

By default, Node treats every `.js` file as CommonJS. Adding `"type": "module"` to `package.json` flips that default — now every `.js` file in the project is treated as ESM instead, and you use `import`/`export` directly without renaming files to `.mjs`.

```json
{
  "name": "my-test-framework",
  "type": "module",
  "scripts": {
    "test": "node --test"
  }
}
```

**Real reasons this actually comes up in test automation projects specifically:**

- **A lot of modern tooling ships as ESM-only now** — some newer versions of testing libraries, HTTP clients, or utility packages publish only an ESM build. If your project is still CommonJS, importing them requires the clunkier dynamic `import()` instead of a plain `require()`, or sometimes doesn't work cleanly at all. Setting `"type": "module"` lets you `import` them normally.
- **Top-level `await`.** ESM allows `await` outside of an `async function`, right at the top level of a module. This is genuinely useful in test setup/config files — loading environment config, fetching a token, or initializing a test database connection — without wrapping the whole file in an async IIFE.

```js
// ESM only — top-level await, no wrapper function needed
export const authToken = await fetchAuthToken();
```

```js
// CommonJS equivalent needs an IIFE wrapper
let authToken;
(async () => {
  authToken = await fetchAuthToken();
})();
```

- **Consistency with the rest of the JS ecosystem.** Frontend code, most modern frameworks, and newer Node APIs are converging on ESM as the default going forward. Teams often set `"type": "module"` in a test framework specifically to keep shared code (like validation logic or types also used in the frontend) importable without a build step translating between module systems.

**The gotcha worth mentioning:** once you set `"type": "module"`, *every* `.js` file in that project is ESM — including any older CommonJS-style utility files you haven't migrated yet, which will now throw syntax errors on `require()`/`module.exports`. Either migrate them to `import`/`export`, or explicitly rename files you want to keep as CommonJS to `.cjs`.

---

## 6. Test Tie-In: Structuring a Test Framework Into Modules

This is where this topic is genuinely practical — not an abstract syntax choice, but something that shapes how maintainable a test framework actually is a year or two in.

### A real folder structure

```
test-framework/
├── pages/
│   ├── BasePage.js
│   ├── LoginPage.js
│   └── DashboardPage.js
├── utils/
│   ├── apiClient.js
│   └── dataFactory.js
├── config/
│   └── testConfig.js
└── tests/
    └── login.test.js
```

### `config/testConfig.js` — one default export, since it's a single cohesive thing

```js
const env = process.env.TEST_ENV || "staging";

const configs = {
  staging: { baseUrl: "https://staging.app.com", timeout: 5000 },
  production: { baseUrl: "https://app.com", timeout: 10000 },
};

export default configs[env];
```

### `utils/dataFactory.js` — named exports, since these are independent, pickable helpers

```js
export function createUser(overrides = {}) {
  return { id: 1, name: "Test User", active: true, ...overrides };
}

export function createOrder(overrides = {}) {
  return { id: 100, items: [], total: 0, ...overrides };
}
```

### `pages/BasePage.js` — default export, one class per file

```js
export default class BasePage {
  constructor(page) {
    this.page = page;
  }

  async navigateTo(url) {
    await this.page.goto(url);
  }
}
```

### `pages/LoginPage.js` — imports the base, extends it, exports itself the same way

```js
import BasePage from "./BasePage.js";

export default class LoginPage extends BasePage {
  async login(username, password) {
    await this.page.fill("#username", username);
    await this.page.fill("#password", password);
    await this.page.click("#submit");
  }
}
```

### `tests/login.test.js` — pulls everything together

```js
import LoginPage from "../pages/LoginPage.js";
import { createUser } from "../utils/dataFactory.js";
import config from "../config/testConfig.js";

test("user can log in", async ({ page }) => {
  const loginPage = new LoginPage(page);
  const testUser = createUser({ name: "Alice" });

  await loginPage.navigateTo(config.baseUrl);
  await loginPage.login(testUser.name, "password123");
});
```

**The design decisions worth explaining if asked:**

- **Page objects use default exports** — each file represents exactly one page/class, so there's no ambiguity, and the import reads cleanly (`import LoginPage from "./LoginPage.js"`).
- **Utility files use named exports** — `dataFactory.js` has several independent helper functions, and named exports let a test file import only what it actually needs (`import { createUser } from ...`), which also makes it immediately obvious in any test file exactly which helpers are in play.
- **Config uses a single default export** — it's one cohesive object representing "the current environment's settings," so a default export fits naturally, and there's no need to destructure individual named pieces out of it everywhere.

**A senior-level point worth raising if asked how you'd structure a larger framework:** keep a consistent convention across the whole project — don't let some page objects use default exports and others use named exports arbitrarily. Consistency here matters more than which specific choice you make, because it's what lets a new team member predict how to import something without having to open the file first.

---

## 7. Interview Q&A Script

**Q: What's the difference between CommonJS and ES Modules?**
> "CommonJS is Node's original module system — `require`/`module.exports`, loaded synchronously, and importing something gives you a copy of the exported value at that moment. ESM is the language-standard system — `import`/`export`, statically analyzed before execution, asynchronous by design, and exports are live bindings, so changes to the exporting module's variable are reflected on the importing side too. Node supports both, but a project or file needs to declare which one it's using — via `"type": "module"`, file extensions, or just defaulting to CommonJS."

**Q: Why might a Node test project need `"type": "module"`?**
> "A few real reasons: some modern libraries or HTTP clients now ship ESM-only, so importing them cleanly in a CommonJS project gets awkward. Top-level `await` is genuinely useful in test setup and config files — loading env config or an auth token without wrapping everything in an async IIFE. And some teams want consistency with frontend code or newer tooling that's standardized on ESM. The trade-off is that once you flip it on, every `.js` file in the project is treated as ESM, so any remaining CommonJS files need to be migrated or renamed to `.cjs`."

**Q: What's the real difference between a default export and a named export?**
> "A named export has to be imported using its exact exported name, which makes tooling and refactors more reliable. A default export can be imported under any name the importer chooses, which is flexible but means there's no single canonical name tied to it. In practice, I use default exports for files that represent exactly one thing — a single class, like a page object — and named exports for utility files with several independent helpers someone might want to import individually."

**Q: Is there a real practical difference between CJS copying exports and ESM using live bindings, or is that just trivia?**
> "It's real — I've seen it cause confusion when a module exports a mutable value, like a counter or a config object, and someone destructures it with `require`. In CommonJS, that destructured value is frozen at the moment of import; it won't reflect later changes inside the module. In ESM, the imported binding stays live and does reflect those changes. It's not something you hit daily, but it's exactly the kind of thing that causes a confusing, hard-to-explain bug if you don't know it's happening."

---

## 8. One-Page Cheat Sheet

- **CommonJS:** `require()`/`module.exports`, synchronous, can be called conditionally/anywhere in code, exports are **copied values**.
- **ESM:** `import`/`export`, statically analyzed at parse time (top-level only; use dynamic `import()` for conditional loading), exports are **live bindings**.
- **Default export:** one per file, importer can name it anything — best for files representing one cohesive thing (a class, a config object).
- **Named exports:** exact name required (or explicit `as` rename) — best for utility files with several independent, individually-importable helpers.
- **`"type": "module"` in `package.json`:** makes Node treat `.js` files as ESM by default. Needed for: ESM-only dependencies, top-level `await` in setup/config files, and ecosystem consistency. Gotcha: every `.js` file in the project becomes ESM — old CommonJS files need migrating or renaming to `.cjs`.
- **Other real differences:** `this` at top level (`module.exports` in CJS vs `undefined` in ESM); `__dirname`/`__filename` exist automatically in CJS but must be reconstructed in ESM via `import.meta.url`.
- **Framework structuring rule:** default exports for one-class-per-file page objects, named exports for utility/helper modules, one default export for a cohesive config object — and keep the convention consistent across the whole project so imports are predictable without opening each file.
