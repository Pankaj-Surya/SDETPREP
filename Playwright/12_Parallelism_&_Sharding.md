# Playwright — Parallelism & Sharding — Notes with Real-Time Examples

Reference: https://playwright.dev/docs/test-parallel · https://playwright.dev/docs/test-sharding

Simple language, real test-automation scenarios, all in TypeScript.

---

## 1. `fullyParallel` mode

**What it does:** By default, Playwright parallelizes across **test files** (each file runs in its own worker), but tests **within** the same file run one after another. `fullyParallel: true` makes every individual test run in parallel, even ones in the same file.

**Real-time example — a large regression suite where each test is fully independent (its own fresh page/data), so there's no reason to run them one-by-one:**

```ts
// playwright.config.ts
export default defineConfig({
  fullyParallel: true,
});
```

```ts
// product-search.spec.ts — with fullyParallel: true, these 3 tests can now
// run at the same time across different workers, instead of sequentially
test('search by name', async ({ page }) => { /* ... */ });
test('search by category', async ({ page }) => { /* ... */ });
test('search by price range', async ({ page }) => { /* ... */ });
```

> Only turn this on if tests are truly independent — no shared mutable state (like the same DB record) between them, or you'll get race conditions.

---

## 2. `workers` config

**What it does:** Controls **how many tests run at the same time** (how many parallel worker processes Playwright spins up).

**Real-time example — using more workers locally (fast dev machine) but fewer on a resource-constrained CI runner to avoid flakiness from CPU contention:**

```ts
// playwright.config.ts
export default defineConfig({
  workers: process.env.CI ? 4 : '75%', // 4 fixed workers on CI, 75% of local CPU cores otherwise
});
```

```bash
# override just for one run, e.g. debugging a suspected race condition
npx playwright test --workers=1
```

---

## 3. Worker-scoped fixtures

**What it does:** A fixture created **once per worker process** and reused by every test file that worker happens to run — instead of recreating an expensive resource for every single test.

**Real-time example — one authenticated API client shared by every test in that worker, instead of logging in again and again:**

```ts
type WorkerFixtures = { apiToken: string };

export const test = base.extend<{}, WorkerFixtures>({
  apiToken: [async ({}, use) => {
    const res = await fetch('https://api.example.com/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email: 'user@example.com', password: 'pass' }),
    });
    const { token } = await res.json();
    await use(token); // shared by ALL tests that run in this worker
  }, { scope: 'worker' }],
});
```

```ts
test('order history loads', async ({ page, apiToken }) => {
  await page.setExtraHTTPHeaders({ Authorization: `Bearer ${apiToken}` });
  await page.goto('/orders');
});
```

---

## 4. Serial mode (`test.describe.serial`)

**What it does:** Forces tests inside a describe block to run **in order, on the same worker**, and — critically — if one test in the chain fails, the rest are **skipped** (since they depend on the previous one's state).

**Real-time example — a multi-step wizard where each step genuinely depends on the previous one succeeding (e.g. account creation → email verification → first login):**

```ts
test.describe.serial('New account onboarding', () => {
  test('step 1: create account', async ({ page }) => {
    await page.goto('/signup');
    // ...
  });

  test('step 2: verify email', async ({ page }) => {
    // relies on the account created in step 1 still existing
  });

  test('step 3: first login shows welcome tour', async ({ page }) => {
    // relies on steps 1 and 2 having succeeded
  });
});
```

> Use sparingly — serial tests can't be parallelized and one failure cascades. Prefer independent tests (with API-based setup) wherever possible.

---

## 5. Sharding (`--shard N/M`)

**What it does:** Splits your **entire test suite** across multiple machines/CI jobs (not just multiple workers on one machine) — each shard runs only its slice of the tests, all shards running at the same time.

**Real-time example — running a 1,000-test suite across 4 CI machines simultaneously instead of one machine running everything sequentially:**

```bash
# Machine 1
npx playwright test --shard=1/4
# Machine 2
npx playwright test --shard=2/4
# Machine 3
npx playwright test --shard=3/4
# Machine 4
npx playwright test --shard=4/4
```

**GitHub Actions matrix example — 4 shards running in parallel jobs:**

```yaml
strategy:
  matrix:
    shardIndex: [1, 2, 3, 4]
    shardTotal: [4]
steps:
  - run: npx playwright test --shard=${{ matrix.shardIndex }}/${{ matrix.shardTotal }}
```

---

## 6. `merge-reports` CLI

**What it does:** Since each shard produces its **own separate** report, `merge-reports` combines all of them back into a single, unified report — so you don't have to check 4 different HTML reports to see the whole run's results.

**How it works:** Set the reporter to `'blob'` on CI (a mergeable intermediate format), then merge afterward.

```ts
// playwright.config.ts
export default defineConfig({
  reporter: process.env.CI ? 'blob' : 'html',
});
```

**Real-time example — CI pipeline: run 4 shards (each producing a blob report), collect them into one folder, then merge into a single HTML report:**

```yaml
# after all 4 shard jobs finish and their blob-report/*.zip files are collected into ./all-blob-reports
- run: npx playwright merge-reports --reporter html ./all-blob-reports
```

```bash
# locally, after copying blob reports from each shard into one folder
npx playwright merge-reports --reporter html ./all-blob-reports
```

The result is one `playwright-report/index.html` showing every test from every shard, as if it had all run on one machine.

---

## Quick summary

| Feature | One-line real-world use |
|---|---|
| `fullyParallel` | Run every independent test concurrently, not just per-file |
| `workers` | Tune concurrency: more locally, fewer/fixed on CI |
| Worker-scoped fixtures | Share one expensive resource (auth token, DB connection) across a worker's tests |
| `test.describe.serial` | Force ordered, dependent steps (e.g. onboarding wizard) to run in sequence |
| Sharding (`--shard N/M`) | Split the whole suite across multiple CI machines for speed |
| `merge-reports` | Combine each shard's blob report into one unified HTML report |
