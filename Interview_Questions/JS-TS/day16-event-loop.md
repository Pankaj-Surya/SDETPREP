# Day 16 — Event Loop Deep Dive (Microtasks vs Macrotasks)

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. The Two Queues — Quick Recap From Day 12

| | Macrotask queue | Microtask queue |
|---|---|---|
| What lands here | `setTimeout`, `setInterval`, DOM/UI events, I/O callbacks, `setImmediate` (Node) | `.then/.catch/.finally` callbacks, code after an `await`, `queueMicrotask()`, `MutationObserver` |
| Priority | Lower | **Higher** |
| How the loop treats it | Takes **one** task per turn | Drains **everything**, including microtasks added while draining |

**The loop in four lines:**

1. Run the current script (all synchronous code) until the call stack is empty.
2. Drain the **entire** microtask queue — if a microtask schedules another microtask, that one runs too, before moving on.
3. Take **one** macrotask, run it to completion.
4. Drain microtasks again. Repeat forever.

That "drain after *every* macrotask" detail is what most puzzle answers hinge on.

---

## 2. A Reliable Method for Any "Predict the Output" Puzzle

Don't guess — run this checklist every time, writing the queues down on paper:

1. **Run all synchronous code first, top to bottom.** Remember two things that look async but are sync:
   - The function passed to `new Promise(executor)` runs **immediately**.
   - An `async` function runs synchronously **up to its first `await`**.
2. **As you go, register callbacks:**
   - `setTimeout(fn, 0)` → fn goes to the **macrotask** queue.
   - `.then(fn)` on an already-resolved promise → fn goes to the **microtask** queue.
   - Code after `await x` → continuation goes to the **microtask** queue.
3. **When the stack is empty:** drain microtasks, in the order they were queued. New microtasks added during this drain run in the same drain.
4. **Take one macrotask**, run it, then drain microtasks again.
5. Repeat until both queues are empty.

---

## 3. Puzzle 1 — The Classic (warm-up)

```js
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve().then(() => console.log("3"));

console.log("4");
```

*Pause and predict before reading on.*

**Output:**
```
1
4
3
2
```

**Walkthrough:**
- Sync pass: logs `1`. `setTimeout` registers "2" as a macrotask. `.then` registers "3" as a microtask. Logs `4`.
- Stack empty → drain microtasks: logs `3`.
- Next macrotask: logs `2`.

**Say it like this:** "Sync code first, then all microtasks, then the next macrotask — Promises beat `setTimeout(fn, 0)` every time."

---

## 4. Puzzle 2 — Nested Microtasks

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve()
  .then(() => {
    console.log("C");
    Promise.resolve().then(() => console.log("D"));
  })
  .then(() => console.log("E"));

console.log("F");
```

**Output:**
```
A
F
C
D
E
B
```

**Walkthrough:**
- Sync pass: `A`, register timeout "B" (macrotask), register first `.then` (microtask), `F`.
- Drain microtasks: first callback runs → logs `C`, and **while running** queues the inner `.then` ("D"). When that first callback returns, its promise resolves, which queues the second `.then` ("E"). So the queue is now `[D, E]`.
- Drain continues: `D`, then `E`.
- Microtask queue empty → macrotask: `B`.

**The lesson:** microtasks queued *during* a microtask drain still run before any macrotask. `B` waits for the whole microtask chain, however long.

---

## 5. Puzzle 3 — `async/await`

```js
async function foo() {
  console.log("1");
  await bar();
  console.log("2");
}

async function bar() {
  console.log("3");
}

console.log("4");
foo();
console.log("5");
```

**Output:**
```
4
1
3
5
2
```

**Walkthrough:**
- `4` logs (sync).
- `foo()` starts running **synchronously**: logs `1`, then calls `bar()`, which also runs synchronously and logs `3`.
- `await bar()` — `bar` returned an already-resolved promise, but `await` still pauses `foo` and queues the rest of `foo` as a **microtask**. Control returns to the caller.
- `5` logs (sync).
- Stack empty → drain microtasks: the continuation of `foo` runs, logs `2`.

**The lesson:** `await` doesn't make the code *before* it asynchronous. Everything up to the first `await` — including calls to other async functions — runs synchronously. Only the code *after* the `await` is deferred.

---

## 6. Puzzle 4 — Microtasks Inside Macrotasks

```js
setTimeout(() => {
  console.log("timeout 1");
  Promise.resolve().then(() => console.log("promise inside timeout 1"));
}, 0);

setTimeout(() => console.log("timeout 2"), 0);

Promise.resolve().then(() => console.log("promise 1"));

console.log("sync");
```

**Output:**
```
sync
promise 1
timeout 1
promise inside timeout 1
timeout 2
```

**Walkthrough:**
- Sync pass: register both timeouts (two macrotasks), register `promise 1` (microtask), log `sync`.
- Drain microtasks: `promise 1`.
- Macrotask #1 runs: logs `timeout 1`, queues a new microtask.
- **Microtasks are drained again right after that macrotask finishes** — before macrotask #2: `promise inside timeout 1`.
- Macrotask #2: `timeout 2`.

**The lesson:** the microtask drain happens after *every* macrotask, not just once at startup. Many people predict `timeout 1, timeout 2, promise inside timeout 1` — that's the wrong mental model.

*(Note: this is the behavior in browsers and Node 11+. Very old Node versions batched timers differently — worth a one-line mention only if asked.)*

---

## 7. Puzzle 5 — The Full Classic (everything at once)

This one appears in interviews almost verbatim. Take your time.

```js
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}

async function async2() {
  console.log("async2");
}

console.log("script start");

setTimeout(() => console.log("setTimeout"), 0);

async1();

new Promise((resolve) => {
  console.log("promise1");
  resolve();
}).then(() => console.log("promise2"));

console.log("script end");
```

**Output:**
```
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
```

**Walkthrough, step by step:**
1. `script start` — sync.
2. `setTimeout` registers a macrotask.
3. `async1()` runs synchronously: logs `async1 start`, calls `async2()` which logs `async2`. Then `await` pauses `async1` and queues its continuation (`async1 end`) as a **microtask — queued first**.
4. `new Promise(executor)` — the executor runs **synchronously**, so `promise1` logs now. `resolve()` is called, and `.then` queues `promise2` as a microtask — queued **second**.
5. `script end` — sync.
6. Drain microtasks in queue order: `async1 end`, then `promise2`.
7. Macrotask: `setTimeout`.

**Why `async1 end` comes before `promise2`:** it was queued first (step 3 happens before step 4). Order of microtasks is simply the order they were queued.

*(One accuracy note for follow-ups: in older engines — before roughly Node 12 / Chrome 73 — `await` took extra ticks, so `async1 end` appeared after `promise2`. Modern engines behave as shown. If an interviewer's answer key differs, this is usually why.)*

---

## 8. Extras Worth Knowing

### Microtasks can starve macrotasks

```js
function loop() {
  Promise.resolve().then(loop); // each microtask schedules another one
}
loop();

setTimeout(() => console.log("never runs"), 0); // the microtask queue never empties
```

Because the loop drains microtasks *completely* before moving on, an endlessly self-scheduling microtask freezes timers, rendering, and input. Worth a one-liner: "microtasks run to exhaustion, so a runaway promise chain can block the page."

### `queueMicrotask` and `process.nextTick` (Node)

```js
queueMicrotask(() => console.log("microtask"));

// Node only:
process.nextTick(() => console.log("nextTick")); // runs BEFORE promise microtasks
```

In Node, the `process.nextTick` queue is processed before the promise microtask queue. In practice: `nextTick` → promise callbacks → then macrotasks. Mention it if the role is Node-heavy, but don't lead with it.

### `setTimeout` vs `setImmediate` (Node)

Called from the main module, the order of `setTimeout(fn, 0)` and `setImmediate(fn)` is not guaranteed. Inside an I/O callback, `setImmediate` always fires first. Only worth bringing up if asked — the safe answer is "don't depend on their relative order."

---

## 9. Test Tie-In: Why an Assertion Runs Before the UI Update Finishes

Almost every "it passes locally, fails in CI" or "it needs a sleep to work" test bug is an event-loop ordering bug in disguise.

### The core problem

`await` only waits for **the promise you gave it**. It knows nothing about what the application does *afterwards* — a state update, a re-render, a debounced handler, a follow-up network call. Those happen in later microtasks or macrotasks, and your next line of test code may run first.

```js
// App code (simplified)
button.addEventListener("click", async () => {
  const data = await fetch("/api/profile").then((r) => r.json()); // takes time
  label.textContent = data.name;                                    // UI update happens LATER
});
```

```js
// ❌ Test code
await page.click("#load");                       // resolves once the click is dispatched — NOT when the handler finishes
const text = await page.textContent("#label");   // reads immediately — label is still empty
expect(text).toBe("Alice");                      // fails (or passes only if the server is fast)
```

**Why it fails, in event-loop terms:** the click handler's `fetch` is still in flight when the test reads the DOM. The UI update is queued for a future turn; the test's read happened first.

### The fix: wait on the *condition*, not on the action

```js
// ✅ Playwright — web-first assertion retries until true or timeout
await page.click("#load");
await expect(page.locator("#label")).toHaveText("Alice");
```

### Same bug in unit/component tests (Jest + Testing Library)

```js
// ❌ Reads DOM right after triggering an async state update
fireEvent.click(screen.getByText("Load"));
expect(screen.getByText("Alice")).toBeInTheDocument(); // throws — update hasn't happened yet

// ✅ Wait for the UI to catch up
fireEvent.click(screen.getByText("Load"));
expect(await screen.findByText("Alice")).toBeInTheDocument(); // findBy* = built-in retry

// ✅ Or wrap an assertion that needs retrying
await waitFor(() => expect(screen.getByText("Alice")).toBeInTheDocument());
```

### Flushing pending promises, explained through the event loop

You'll see this helper in older Jest code:

```js
const flushPromises = () => new Promise((resolve) => setTimeout(resolve, 0));

it("updates after the API resolves", async () => {
  render(<Profile />);
  await flushPromises(); // lets all pending promise callbacks run first
  expect(screen.getByText("Alice")).toBeInTheDocument();
});
```

**Why this works — and it's a nice interview point:** `setTimeout(resolve, 0)` schedules a *macrotask*. Macrotasks only run after the microtask queue is fully drained, so by the time `resolve` fires, every already-queued `.then` callback has run. You're using the event loop's ordering rules to guarantee "everything pending in microtasks is done."

**Caveat worth stating:** it only flushes work that's already queued. If the app schedules a *real* delay (an actual 300ms debounce or a network call that hasn't returned), `flushPromises` won't wait for it — you need a condition-based wait. With Jest's fake timers, `setTimeout` is faked too, so you'd advance the clock instead (for example `jest.advanceTimersByTime(...)`, or the async variant on recent Jest/Vitest versions) and let microtasks flush with an `await`.

### Debounce example — a classic

```js
// App: search runs 300ms after the user stops typing
input.addEventListener("input", debounce(runSearch, 300));
```

```js
// ❌ Asserts before the debounce timer (a macrotask) has even fired
await page.fill("#search", "laptop");
expect(await page.locator(".result").count()).toBeGreaterThan(0); // 0 — search hasn't run yet

// ✅ Wait for the observable result
await page.fill("#search", "laptop");
await expect(page.locator(".result").first()).toBeVisible();
```

### A debugging checklist for "assertion ran too early"

1. **Is the thing I'm asserting on produced by something async I didn't wait for?** (fetch, timer, debounce, animation, state batching)
2. **Did I `await` the *action* but not the *effect*?** `await click()` finishes when the click is dispatched, not when the app finishes reacting.
3. **Am I doing a one-shot read and then asserting?** Replace with a retrying assertion (`toHaveText`, `findBy*`, `waitFor`).
4. **Is there a missing `await` on a promise in the test itself?** (Day 15's #1 bug.)
5. **Does adding a `sleep` make it pass?** That confirms it's a timing issue — then replace the sleep with a condition. Never leave the sleep.

---

## 10. Interview Q&A Script

**Q: Predict the output order of this code (sync + `setTimeout` + Promise).**
> *(Don't blurt an answer. Say the method aloud: "Sync first, then drain microtasks, then one macrotask, then drain microtasks again." Write the two queues as you trace. Mention the two traps proactively: Promise executors run synchronously, and an `async` function runs synchronously up to its first `await`.)*

**Q: What's the difference between the microtask and macrotask queues?**
> "Macrotasks are things like `setTimeout`, I/O, and UI events — the event loop takes one per turn. Microtasks are Promise callbacks and the code after an `await` — they have higher priority, and the loop drains the entire microtask queue, including newly added microtasks, before it takes the next macrotask. It also drains microtasks after every macrotask, not just once at the start."

**Q: Why does a resolved Promise's `.then` run before `setTimeout(fn, 0)`?**
> "Because after the current synchronous code finishes, the event loop clears all microtasks before it touches the macrotask queue. The `.then` callback is a microtask; the timer callback is a macrotask. The timer's `0` delay doesn't give it any priority — it only says how soon it becomes *eligible*."

**Q: Can microtasks block the page or starve timers?**
> "Yes. Since the loop runs microtasks until the queue is completely empty, a chain of promises that keeps scheduling new promises can monopolize the thread and prevent timers, rendering, and input from running."

**Q: Why does an assertion sometimes run before the UI update finishes?**
> "Because `await` only waits for the specific promise I awaited — for example the click being dispatched — not for the app's follow-up work, which happens in later microtasks or macrotasks: a fetch, a state update, a debounce timer. My next line runs first. The fix is to wait on the observable condition with a retrying assertion — `toHaveText`, `findBy*`, `waitFor` — instead of reading a value once or adding a fixed sleep."

**Q: Why does `await new Promise(r => setTimeout(r, 0))` flush pending promise callbacks?**
> "It schedules a macrotask, and macrotasks only run after the microtask queue is drained. So by the time that timer fires, every already-queued `.then` callback has executed. It only helps with work that's already queued — it won't wait for a real network call or a real debounce delay."

---

## 11. One-Page Cheat Sheet

- **Loop order:** run sync code → drain *all* microtasks → run *one* macrotask → drain microtasks again → repeat.
- **Microtasks:** `.then/.catch/.finally`, code after `await`, `queueMicrotask`. **Macrotasks:** `setTimeout`, `setInterval`, I/O, UI events, `setImmediate` (Node).
- **Microtasks added during a drain run in the same drain** — they all finish before any macrotask.
- **Sync traps:** the Promise executor runs immediately; an `async` function runs synchronously up to its first `await`; only code *after* `await` is deferred.
- **Queue order = scheduling order** within the microtask queue. In modern engines, `await` on an already-resolved promise costs one tick.
- **Runaway microtasks** (a promise that keeps re-scheduling itself) starve timers, rendering, and input.
- **Node extra:** `process.nextTick` runs before promise microtasks.
- **Puzzle answers to remember:** Puzzle 1 → `1 4 3 2`. Puzzle 2 → `A F C D E B`. Puzzle 3 → `4 1 3 5 2`. Puzzle 4 → `sync, promise 1, timeout 1, promise inside timeout 1, timeout 2`. Puzzle 5 → `script start, async1 start, async2, promise1, script end, async1 end, promise2, setTimeout`.
- **Testing rule:** `await action` ≠ `await effect`. Assert on the observable result with a retrying assertion (`expect(locator).toHaveText`, `findBy*`, `waitFor`); never add a fixed `sleep` to hide a timing bug. `flushPromises` (`setTimeout(…, 0)`) only drains already-queued microtasks — it won't wait for real timers or network calls.
