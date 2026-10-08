# Day 17 — Fetch API & HTTP Basics in JS

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. HTTP Basics — the 60-Second Refresher

You know this already, so just the parts that show up in interview answers and test assertions.

**Methods and what they mean:**

| Method | Purpose | Safe? | Idempotent? |
|---|---|---|---|
| `GET` | Read a resource | Yes | Yes |
| `POST` | Create / trigger an action | No | **No** |
| `PUT` | Replace a resource entirely | No | Yes |
| `PATCH` | Partially update a resource | No | Not guaranteed |
| `DELETE` | Remove a resource | No | Yes |

*Idempotent* = calling it ten times leaves the server in the same state as calling it once. That's why retrying a `GET` or `PUT` is generally safe, but blindly retrying a `POST` can create duplicates — worth saying out loud when retries come up.

**Status code families:**

- `2xx` success — `200 OK`, `201 Created`, `204 No Content`
- `3xx` redirect — `301`, `302`, `304 Not Modified`
- `4xx` the **client** did something wrong — `400 Bad Request`, `401 Unauthorized` (not authenticated), `403 Forbidden` (authenticated, not allowed), `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests`
- `5xx` the **server** failed — `500`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout`

**Headers that matter in tests:** `Content-Type` (what the body is), `Accept` (what the client wants back), `Authorization` (credentials), plus `Cache-Control` and `Location` (often returned with a `201`).

---

## 2. `fetch` — the Basics

`fetch` is built into modern browsers and into Node 18+ (no import needed).

```js
const response = await fetch("https://jsonplaceholder.typicode.com/users/1");

console.log(response.status);     // 200
console.log(response.ok);         // true — status is in the 200–299 range
console.log(response.headers.get("content-type")); // "application/json; charset=utf-8"

const user = await response.json(); // a SECOND promise — reading the body is async too
console.log(user.name);
```

**Two things that surprise people:**

**1. There are two awaits.** The first `await fetch(...)` resolves as soon as the **headers** arrive. Reading the body (`.json()`, `.text()`, `.blob()`) is a separate async step.

**2. The body can only be read once.**

```js
const response = await fetch(url);
await response.json();
await response.json(); // ❌ TypeError: body stream already read

// If you need it twice, clone first:
const copy = response.clone();
const a = await response.json();
const b = await copy.json();
```

### Sending data — a `POST` with JSON

```js
const response = await fetch("https://api.example.com/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json", // you MUST set this yourself with fetch
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({ name: "Alice", email: "alice@test.com" }), // must be a string
});
```

**The classic mistake:** forgetting `JSON.stringify` (the body becomes `"[object Object]"`) or forgetting the `Content-Type` header (server can't parse the body and returns a `400`). `fetch` does **not** do either for you — a point where axios is more convenient (see section 6).

---

## 3. `fetch` Throwing vs Returning `ok: false` — the Core Interview Question

This is *the* question for this topic, and the answer is short but precise:

> **`fetch` only rejects when the request couldn't complete at all** — no network, DNS failure, connection refused, request aborted, or (in browsers) a CORS block. **If the server responds with *any* HTTP status — including 404 or 500 — the promise *resolves*** and you get a `Response` with `ok: false`.

```js
// Server responds with 404 → fetch does NOT throw
const response = await fetch("https://api.example.com/users/99999");
console.log(response.ok);     // false
console.log(response.status); // 404
// No exception. The code just keeps running.

// No connection at all → fetch DOES throw
try {
  await fetch("https://this-domain-does-not-exist.invalid");
} catch (error) {
  console.log(error.name); // "TypeError" (Node: "fetch failed")
}
```

**Why it was designed this way:** from HTTP's point of view, a 404 is a *successful* round trip — the server received the request and answered. `fetch` treats "did we get a response?" and "was the response what we wanted?" as two different questions. The consequence is that **you must check `response.ok` yourself**, every time.

**The bug this causes in real code:**

```js
// ❌ Looks safe, isn't — a 500 sails straight through
async function getUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    return await response.json(); // 500 with an HTML error page → .json() throws a confusing SyntaxError
  } catch (error) {
    console.error(error);
  }
}
```

If the server returns a `404` with `{ "error": "not found" }`, this code happily returns that error object *as if it were a user*. If it returns a `502` with an HTML page, `.json()` throws `Unexpected token <` — the real problem (a 502) is hidden behind a parsing error (Day 11's classic).

### The two failure modes, side by side

| Situation | `fetch` promise | What you check |
|---|---|---|
| Network down, DNS failure, connection refused, aborted | **Rejects** | `catch` block |
| Server returns 4xx / 5xx | **Resolves** with `ok: false` | `response.ok` / `response.status` |
| Server returns 2xx but the body isn't valid JSON | Resolves; **`.json()` rejects** | `try/catch` around the body read |

---

## 4. How Do You Handle a Failed `fetch` Request? — a Production-Quality Answer

Build this up in layers. Each layer is a point worth saying in an interview.

### Layer 1 — check `response.ok` and throw a meaningful error

Using the custom error pattern from Day 9:

```js
class HttpError extends Error {
  constructor(message, status, body) {
    super(message);
    this.name = "HttpError";
    this.status = status;
    this.body = body;
  }
}

async function request(url, options = {}) {
  const response = await fetch(url, options);

  if (!response.ok) {
    // Try to read the error body, but don't let a non-JSON body hide the real status
    const body = await response.text().catch(() => "");
    throw new HttpError(`${options.method ?? "GET"} ${url} failed: ${response.status}`, response.status, body);
  }

  if (response.status === 204) return null; // No Content — there's no body to parse
  return response.json();
}
```

Notice two small details that signal experience: reading the error body as **text** (so a non-JSON error page can't cause a second failure), and handling **`204 No Content`**, where calling `.json()` would throw.

### Layer 2 — a timeout, because `fetch` has none by default

`fetch` will wait a very long time for a hung server. Add a timeout with `AbortSignal`:

```js
async function requestWithTimeout(url, options = {}, timeoutMs = 5000) {
  try {
    return await request(url, { ...options, signal: AbortSignal.timeout(timeoutMs) });
  } catch (error) {
    if (error.name === "TimeoutError") {
      throw new Error(`Request to ${url} timed out after ${timeoutMs}ms`);
    }
    throw error;
  }
}
```

(`AbortSignal.timeout` is available in Node 18+ and modern browsers. The older equivalent is creating an `AbortController` and calling `controller.abort()` from a `setTimeout`.)

### Layer 3 — retry only what's worth retrying

```js
async function requestWithRetry(url, options = {}, { retries = 3, delayMs = 500 } = {}) {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      return await requestWithTimeout(url, options);
    } catch (error) {
      const isNetworkError = !(error instanceof HttpError);
      const isRetryableStatus = error instanceof HttpError && (error.status >= 500 || error.status === 429);

      // 4xx errors (except 429) mean OUR request is wrong — retrying won't fix it
      if (attempt === retries || !(isNetworkError || isRetryableStatus)) throw error;

      await new Promise((r) => setTimeout(r, delayMs * 2 ** (attempt - 1))); // exponential backoff
    }
  }
}
```

**The reasoning to say aloud:**

- **Retry:** network failures, `5xx`, `429` — these are *transient*; the same request may succeed a moment later.
- **Don't retry:** `400`, `401`, `403`, `404`, `422` — the request itself is wrong, so retrying just repeats the failure.
- **Be careful with non-idempotent methods:** retrying a `POST` that timed out can create a duplicate, because you don't know whether the server processed the first attempt. Use idempotency keys, or only auto-retry `GET`/`PUT`/`DELETE`.
- **Use backoff**, not a tight loop, so you don't hammer a server that's already struggling.

### The one-paragraph interview answer

> "I check `response.ok`, because `fetch` only rejects on network-level failures — a 404 or 500 still resolves. If it's not ok, I throw a typed error carrying the status and body, so callers can branch on `error.status` instead of parsing messages. I add a timeout with `AbortSignal.timeout`, since `fetch` has none by default. For retries, I only retry transient failures — network errors, 5xx, and 429 — with exponential backoff, and I don't blindly retry non-idempotent calls like `POST`. And I read error bodies defensively, because a gateway might return HTML instead of JSON."

---

## 5. Handling Responses — a Few More Details Worth Knowing

```js
const response = await fetch(url);

await response.json();        // parse JSON body
await response.text();        // raw text — safest when you're unsure what came back
await response.blob();        // files/images
await response.arrayBuffer(); // binary data
```

**Check the content type before assuming JSON** when the server's behavior is unpredictable:

```js
const contentType = response.headers.get("content-type") ?? "";
const data = contentType.includes("application/json") ? await response.json() : await response.text();
```

**Parallel requests** — same lesson as Days 14–15:

```js
const [users, posts] = await Promise.all([
  fetch("/api/users").then((r) => r.json()),
  fetch("/api/posts").then((r) => r.json()),
]);
```

---

## 6. Intro to Axios — and How It Differs From `fetch`

Axios is a popular HTTP client library (`npm install axios`) that works in Node and the browser. Interviewers ask about it because the differences map exactly onto the `fetch` gotchas above.

```js
import axios from "axios";

const response = await axios.get("https://jsonplaceholder.typicode.com/users/1");
console.log(response.status); // 200
console.log(response.data);   // already parsed — no .json() step
```

| | `fetch` | `axios` |
|---|---|---|
| Installed | Built in (browsers, Node 18+) | Library (`npm install axios`) |
| Non-2xx status | **Resolves** (`ok: false`) | **Rejects by default** |
| JSON parsing | Manual `await response.json()` | Automatic → `response.data` |
| Sending JSON | `JSON.stringify` + `Content-Type` header yourself | Pass an object; sets header automatically |
| Timeout | None by default (use `AbortSignal`) | `timeout: 5000` option |
| Interceptors (shared request/response hooks) | No (write wrappers) | Built in |
| Base URL / shared config | Manual | `axios.create({ baseURL, headers })` |

### Axios rejects on non-2xx — and the error has structure

```js
try {
  await axios.get("https://api.example.com/users/99999");
} catch (error) {
  if (axios.isAxiosError(error)) {
    if (error.response) {
      // Server responded, but with a non-2xx status
      console.log(error.response.status); // 404
      console.log(error.response.data);   // the error body, already parsed
    } else if (error.request) {
      // Request was sent, but no response arrived (network failure, timeout)
      console.log("No response received:", error.message);
    } else {
      // Something went wrong building the request
      console.log("Request setup error:", error.message);
    }
  }
}
```

**Interview line:** "The key behavioral difference is that axios treats non-2xx as an error by default, while `fetch` treats it as a normal response. With axios, the error object tells me *which kind* of failure it was — a server response, no response at all, or a setup problem — which is handy. And you can change the rule with the `validateStatus` option."

### A shared client with `axios.create` and an interceptor

```js
const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
  headers: { "Content-Type": "application/json" },
});

// Attach the auth token to every request in one place
api.interceptors.request.use((config) => {
  config.headers.Authorization = `Bearer ${process.env.API_TOKEN}`;
  return config;
});

// Centralized handling for every response
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) console.warn("Token expired");
    return Promise.reject(error); // always pass the error on
  }
);
```

**When would you still choose `fetch`?** No dependency, built into the platform, works in edge/serverless runtimes, and is perfectly fine once you wrap it with a small helper like the one in section 4. Axios earns its place when you want interceptors, built-in timeouts, and less boilerplate across a large codebase.

---

## 7. Test Tie-In: Writing API Tests in JS/TS — a Very Common SDET Interview Task

The typical prompt: *"Here's a REST API. Write automated tests for it."* The interviewer is watching **what you choose to cover** as much as the code you write.

### First, the checklist to say out loud

For every endpoint, think through:

1. **Status code** — exactly what's expected (`201` for create, `204` for delete, not just "2xx").
2. **Response body** — correct fields, correct types, correct values (schema validation, Day 11).
3. **Headers** — `Content-Type`, `Location` on creation, caching headers where relevant.
4. **Negative cases** — missing required field (`400`/`422`), nonexistent ID (`404`), duplicate (`409`), wrong HTTP method (`405`).
5. **Auth** — no token (`401`), valid token but insufficient role (`403`), expired token.
6. **Data-driven variants** — boundary values, empty strings, very long input, special characters.
7. **State and cleanup** — a created resource should be fetchable afterward and cleaned up after the test.
8. **Performance sanity** — response time under a reasonable threshold.

### Option A: axios + Jest — hitting a running API

The key design choice: set **`validateStatus: () => true`** so axios never throws on 4xx/5xx. In tests, a `404` isn't an exception — it's a **result you want to assert on**.

```js
import axios from "axios";

const api = axios.create({
  baseURL: process.env.API_URL ?? "https://jsonplaceholder.typicode.com",
  timeout: 5000,
  validateStatus: () => true, // never throw — let the test assert on the status itself
});

describe("Users API", () => {
  test("GET /users/1 returns the user", async () => {
    const res = await api.get("/users/1");

    expect(res.status).toBe(200);
    expect(res.headers["content-type"]).toMatch(/application\/json/);
    expect(res.data).toMatchObject({
      id: 1,
      name: expect.any(String),
      email: expect.stringMatching(/^[^\s@]+@[^\s@]+\.[^\s@]+$/), // regex from Day 11
    });
  });

  test("GET /users/99999 returns 404", async () => {
    const res = await api.get("/users/99999");
    expect(res.status).toBe(404);
  });

  test("responds within 2 seconds", async () => {
    const start = Date.now();
    await api.get("/users");
    expect(Date.now() - start).toBeLessThan(2000);
  });
});

describe("Posts API", () => {
  test("POST /posts creates a post", async () => {
    const payload = { title: "Hello", body: "World", userId: 1 };
    const res = await api.post("/posts", payload);

    expect(res.status).toBe(201);
    expect(res.data).toMatchObject(payload);        // echoes what we sent
    expect(res.data.id).toEqual(expect.any(Number)); // server assigned an id
  });
});
```

*(JSONPlaceholder is a free fake REST API — handy for practice. It accepts writes but doesn't actually persist them, so a create-then-read chain needs a real API.)*

### A create → read → delete flow against a real API, with cleanup

```js
describe("Order lifecycle", () => {
  let createdId;

  afterAll(async () => {
    // Clean up even if an assertion failed midway
    if (createdId) await api.delete(`/orders/${createdId}`);
  });

  test("creates an order", async () => {
    const res = await api.post("/orders", { item: "Laptop", qty: 1 });
    expect(res.status).toBe(201);
    createdId = res.data.id;
  });

  test("the created order can be fetched", async () => {
    const res = await api.get(`/orders/${createdId}`);
    expect(res.status).toBe(200);
    expect(res.data.item).toBe("Laptop");
  });

  test("rejects an invalid order", async () => {
    const res = await api.post("/orders", { qty: -5 });
    expect(res.status).toBe(400);
    expect(res.data).toHaveProperty("error");
  });
});
```

**Points worth saying aloud:** tests in a flow share state, so they must run in order and clean up after themselves; and cleanup goes in `afterAll` so a failed test doesn't leave junk data behind (Day 4's shared-state lesson, applied to the server).

### Option B: supertest — testing your own Express app, no server needed

`supertest` lets you test an HTTP app **in-process**. It spins the app up on an ephemeral port for the duration of the test, so there's no need to start a server separately, and no port conflicts.

```js
// app.js — export the app, but do NOT call listen() here
import express from "express";

export const app = express();
app.use(express.json());

app.get("/health", (req, res) => res.json({ status: "ok" }));

app.post("/users", (req, res) => {
  if (!req.body.email) return res.status(400).json({ error: "email is required" });
  res.status(201).json({ id: 1, ...req.body });
});

// server.js — the only file that calls app.listen(...)
```

```js
// users.test.js
import request from "supertest";
import { app } from "./app.js";

describe("POST /users", () => {
  test("creates a user", async () => {
    const res = await request(app)
      .post("/users")
      .send({ email: "alice@test.com" })
      .expect(201)
      .expect("Content-Type", /json/);

    expect(res.body).toMatchObject({ id: expect.any(Number), email: "alice@test.com" });
  });

  test("returns 400 when email is missing", async () => {
    const res = await request(app).post("/users").send({}).expect(400);
    expect(res.body.error).toBe("email is required");
  });
});
```

**The design point to make:** exporting `app` separately from `app.listen()` is what makes the app testable. If `listen()` is called at import time, tests would start a real server, hold ports open, and Jest would hang.

You can also point `supertest` at an already-running server: `request("http://localhost:3000").get("/health")`.

### Option C: plain `fetch` in a test (Node 18+)

```js
test("GET /health returns ok", async () => {
  const res = await fetch(`${process.env.API_URL}/health`);

  expect(res.status).toBe(200);          // note: fetch won't throw on 4xx/5xx, so asserting status is natural here
  expect(await res.json()).toEqual({ status: "ok" });
});
```

Since `fetch` already resolves on any status, it needs no `validateStatus`-style workaround for tests — a small point in its favor.

### Making the suite maintainable (senior-level touches)

- **One API client module** (the `axios.create` instance) imported by every test — base URL, timeout, and auth in one place.
- **Environment config** from variables, never hardcoded URLs (Day 10's config module).
- **Typed responses in TypeScript:** `const res = await api.get<User>("/users/1");` gives `res.data` a real type, so a renamed field is caught at compile time.
- **Schema validation** with `zod`/`ajv` for the response shape instead of dozens of individual `expect` lines.
- **A clear failure message** — include the response body when asserting status, so CI logs show *why* it failed:

```js
expect(res.status, `Unexpected response: ${JSON.stringify(res.data)}`).toBe(201);
```

- **Don't mock what you're testing.** For an API test the point is exercising the real service; mocking belongs in *unit* tests of code that *calls* APIs.

---

## 8. Interview Q&A Script

**Q: How do you handle a failed `fetch` request?**
> *(Use the one-paragraph answer from section 4: check `response.ok` since `fetch` doesn't reject on 4xx/5xx, throw a typed error with status and body, add a timeout via `AbortSignal.timeout`, retry only transient failures with backoff, and read error bodies defensively.)*

**Q: What's the difference between `fetch` throwing and returning `ok: false`?**
> "`fetch` rejects only when the request couldn't complete — no network, DNS failure, connection refused, an abort, or a CORS block in the browser. If the server answers with any status at all, including 404 or 500, the promise resolves with a `Response` whose `ok` is false. So 'did I get an answer?' and 'was the answer good?' are separate checks, and I have to test `response.ok` myself. A third failure mode is a 2xx response whose body isn't valid JSON — there `fetch` resolves, but `.json()` rejects."

**Q: How is axios different from `fetch`?**
> "Axios rejects on non-2xx by default, parses JSON automatically into `response.data`, sets the JSON content type when I pass an object, supports timeouts and interceptors natively, and lets me create a preconfigured instance with a base URL. `fetch` is built in and dependency-free but leaves all of that to me. In tests I usually configure axios with `validateStatus: () => true` so error statuses become values to assert on instead of exceptions."

**Q: Why wouldn't you retry a failed `POST` automatically?**
> "Because a `POST` isn't idempotent. If it timed out, I don't know whether the server processed it, so a retry could create a duplicate order or charge. I'd retry safely for `GET`, `PUT`, and `DELETE`, or use an idempotency key so the server can recognize repeats."

**Q: What's the difference between a 401 and a 403?**
> "`401` means the caller isn't authenticated — missing, invalid, or expired credentials. `403` means they're authenticated, but not permitted to do that. In tests I cover both: no token should give 401; a valid token with the wrong role should give 403."

**Q: You're given an API and asked to test it. How do you approach it?**
> "I start with the contract — endpoints, methods, schemas, auth — then cover the happy path for each endpoint with exact status codes, headers, and a schema-validated body. Then negative cases: missing or invalid fields, nonexistent IDs, duplicates, wrong methods. Then auth — no token, wrong role. Then flows that chain calls, like create, read, update, delete, with cleanup in `afterAll` so failed runs don't leave data behind. I'd keep a single configured client, pull URLs and credentials from environment variables, and add a basic response-time check."

**Q: Why does `supertest` need the app exported separately from `listen()`?**
> "`supertest` starts the app itself on an ephemeral port for each test run. If the module called `listen()` at import time, importing it in a test would start a real server on a fixed port, causing port conflicts and leaving open handles that keep Jest from exiting. Keeping `listen()` in a separate entry file makes the app a pure, importable object."

---

## 9. One-Page Cheat Sheet

- **HTTP:** `GET/PUT/DELETE` are idempotent, `POST` isn't (careful with retries). `4xx` = client's fault, `5xx` = server's fault. `401` = not authenticated, `403` = not allowed.
- **`fetch` basics:** two awaits (response, then body); body readable once (`clone()` if needed); you must set `Content-Type` and `JSON.stringify` the body yourself; `204` has no body, so don't call `.json()`.
- **The core answer:** `fetch` **rejects only on network-level failure** (offline, DNS, refused, aborted, CORS). **Any HTTP status resolves** with `ok: false`. Always check `response.ok`. Valid 2xx with non-JSON body → `.json()` rejects.
- **Robust handling:** check `ok` → throw a typed `HttpError` (status + body) → add a timeout (`AbortSignal.timeout`) → retry only network errors, 5xx, and 429 with exponential backoff → never blindly retry non-idempotent `POST` → read error bodies as text, defensively.
- **Axios vs fetch:** axios rejects on non-2xx (configurable via `validateStatus`), auto-parses to `response.data`, auto-sets JSON header, has `timeout`, interceptors, and `axios.create` for shared config. Error shape: `error.response` (server replied), `error.request` (no reply), neither (setup problem).
- **API test approach:** cover status, headers, body schema, negative cases, auth, data-driven variants, state/cleanup, response time.
- **Tooling:** axios + Jest with `validateStatus: () => true` for external APIs; `supertest` for in-process testing of your own Express app (export `app` separately from `listen()`); plain `fetch` works in Node 18+.
- **Maintainability:** one shared client, env-driven config, typed responses in TypeScript, schema validation (`zod`/`ajv`), failure messages that include the response body, cleanup in `afterAll`, and don't mock the service you're testing.
