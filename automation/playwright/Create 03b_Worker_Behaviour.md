Yes, you hit the nail on the head! Playwright determines the order of file execution **alphabetically by default**.

When you trigger a test run, Playwright scans your test directories, gathers all matching test files, and sorts them alphabetically by their file path/name before passing them to the worker queue.

Because `file1.spec.ts` comes before `file2.spec.ts` alphabetically, Playwright processes `file1` first.

---

## The Hidden Catch: Why It Feels Random Sometimes

While the queue is sorted alphabetically, you might notice that in a massive test suite, the execution order doesn't look strictly alphabetical. This happens because of how workers operate:

1. **Alphabetical Feeding:** Playwright feeds files into the worker pool alphabetically.
2. **Worker Racing:** If `file1` is parallelized across all 4 workers, those workers will chew through `file1`'s individual tests incredibly fast.
3. **First-Come, First-Served:** The very first worker to finish its assigned test from `file1` will immediately look at the alphabetical queue, see `file2` waiting next, and grab it.

---

## What if you want to control the order?

Relying on alphabetical naming (like naming files `01_login.spec.ts`, `02_checkout.spec.ts`) is a common workaround, but Playwright actually provides a built-in way to control order via a **Test Dependencies** feature in your `playwright.config.ts`.

If you absolutely need `file2` to run before `file1`, you can configure your projects like this:

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'Setup/Pre-requisites',
      testMatch: 'file2.spec.ts',
    },
    {
      name: 'Main Tests',
      testMatch: 'file1.spec.ts',
      dependencies: ['Setup/Pre-requisites'], // Forces file2 to finish completely first!
    },
  ],
});

```

### Summary for your framework

Your deduction is 100% correct. Playwright reads and queues your files **alphabetically**. Since `file1` is first in line and set to `parallel`, it floods your available workers instantly, and `file2` waits patiently for the first available opening.
