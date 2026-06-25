By default in Playwright, the answer is: **The 2 test files will run in parallel, but the test cases *inside* each file will run sequentially.** Let's break down exactly how Playwright will distribute your 20 tests across those 4 workers under the hood.

---

## 1. The Default Playwright Behavior

Playwright’s fundamental rule of execution is **file-level parallelization**. It allocates entire test files to workers, not individual test cases.

With 2 files, 10 tests each, and `workers: 4`:

* **Worker 1** takes `file1.spec.ts` and starts executing its 10 tests **one after the other (sequentially)**.
* **Worker 2** takes `file2.spec.ts` and starts executing its 10 tests **one after the other (sequentially)**.
* **Worker 3 & 4** sit idle initially because there are no more files to distribute.

---

## 2. How to Make Test Cases Inside a File Run in Parallel

If you want those 10 test cases inside the same file to run concurrently across your available workers, you have to explicitly opt-in. You do this by adding `test.describe.configure({ mode: 'parallel' })` at the top of your test files.

### If you configure them to be parallel:

```typescript
import { test } from '@playwright/test';

// Add this at the top of both files
test.describe.configure({ mode: 'parallel' });

test('Test 1', async ({ page }) => { ... });
test('Test 2', async ({ page }) => { ... });

```

### The New Behavior with `workers: 4`:

Now, Playwright breaks the file barrier. It will dump all 20 test cases into a single pool and utilize all 4 workers simultaneously:

* **Worker 1:** Runs `File 1 - Test 1`, then `File 1 - Test 5`, etc.
* **Worker 2:** Runs `File 1 - Test 2`, then `File 2 - Test 2`, etc.
* **Worker 3:** Runs `File 1 - Test 3`, then `File 2 - Test 1`, etc.
* **Worker 4:** Runs `File 1 - Test 4`, then `File 2 - Test 3`, etc.

---

## Summary for Your Framework

* **Without changes (Default):** 2 files run in parallel (on 2 workers). The 10 tests *inside* each file run sequentially. 2 workers will sit completely unused.
* **With `mode: 'parallel' `:** All 4 workers will actively grab individual test cases from both files and execute everything completely in parallel, cutting your total execution time significantly down.
