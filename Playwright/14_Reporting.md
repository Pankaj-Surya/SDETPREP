# Playwright — Reporting — Notes with Real-Time Examples

Reference: https://playwright.dev/docs/test-reporters · https://playwright.dev/docs/api/class-reporter

Simple language, real test-automation scenarios, all in TypeScript.

---

## 1. Built-in HTML reporter

**What it does:** Playwright's default, most-used reporter — a self-contained, interactive HTML report with a pass/fail overview, per-test timelines, screenshots, and embedded traces you can open right in the browser.

**Real-time example — a QA engineer wants to review last night's overnight regression run, including screenshots of anything that failed:**

```ts
// playwright.config.ts
export default defineConfig({
  reporter: [['html', { open: 'never' }]], // don't auto-open on CI; open manually when needed
});
```

```bash
npx playwright show-report   # opens the last HTML report in a browser
```

Clicking into a failed test shows the exact step that failed, a screenshot at that moment, and (if `trace` was enabled) the full trace timeline — usually enough to diagnose the failure without re-running anything.

---

## 2. JUnit XML (for CI)

**What it does:** Produces a `.xml` file in the JUnit format that virtually every CI system (Jenkins, Azure DevOps, GitLab CI, CircleCI) understands natively — so failures show up directly in the CI UI (not just as a pass/fail build status).

**Real-time example — a Jenkins pipeline that needs to show individual test failures in its own test-results tab, not just "build failed":**

```ts
// playwright.config.ts
export default defineConfig({
  reporter: [['junit', { outputFile: 'results/junit-report.xml' }]],
});
```

```groovy
// Jenkinsfile snippet
post {
  always {
    junit 'results/junit-report.xml'
  }
}
```

Jenkins then shows a per-test breakdown, historical trend graphs, and flaky-test tracking, all sourced from this one file.

---

## 3. JSON reporter

**What it does:** Dumps the entire test run (every test, status, duration, errors, attachments) as structured JSON — meant for **machines to consume**, not for humans to read directly. Perfect for building custom dashboards or piping results elsewhere (Slack, a database).

**Real-time example — posting a Slack message summarizing pass/fail counts after every CI run:**

```ts
// playwright.config.ts
export default defineConfig({
  reporter: [['json', { outputFile: 'results/results.json' }]],
});
```

```ts
// post-run-slack-notify.ts
import fs from 'fs';

const results = JSON.parse(fs.readFileSync('results/results.json', 'utf-8'));
const failed = results.suites.flatMap((s: any) => s.specs).filter((t: any) => !t.ok);

await fetch(process.env.SLACK_WEBHOOK_URL!, {
  method: 'POST',
  body: JSON.stringify({ text: `Test run finished: ${failed.length} failed out of ${results.suites.length}` }),
});
```

---

## 4. Allure integration

**What it does:** Allure is a popular third-party reporting tool with rich, filterable dashboards (by tag/severity/owner), historical trend charts across runs, and nicer visual grouping than the built-in HTML report — common in enterprise QA teams already standardized on Allure across multiple frameworks.

**Real-time example — a company running Selenium, Playwright, and API tests across different teams, wanting **one consistent reporting dashboard** for all of them:**

```bash
npm install -D allure-playwright
```

```ts
// playwright.config.ts
export default defineConfig({
  reporter: [['allure-playwright']],
});
```

```bash
npx playwright test
npx allure generate ./allure-results --clean -o ./allure-report
npx allure open ./allure-report
```

You can also enrich the report with real business context, using Allure's own annotations inside a test:
```ts
import { test } from '@playwright/test';
import { allure } from 'allure-playwright';

test('checkout applies discount correctly', async ({ page }) => {
  await allure.severity('critical');
  await allure.tag('checkout', 'pricing');
  // ...
});
```

---

## 5. Custom reporter (`Reporter` interface)

**What it does:** Lets you write your **own** reporter class hooking into the test lifecycle — for anything the built-in/third-party reporters don't do out of the box (custom dashboards, a specific internal format, real-time Slack alerts on failure).

**Key lifecycle methods (you only implement the ones you need):**

| Method | Called when |
|---|---|
| `onBegin(config, suite)` | Once, before any test runs |
| `onTestBegin(test, result)` | Right before each test starts |
| `onStepBegin` / `onStepEnd` | Around each `test.step()` |
| `onTestEnd(test, result)` | Right after each test finishes (incl. retries) |
| `onEnd(result)` | Once, after the whole run finishes |

**Real-time example — a lightweight custom reporter that posts a Slack alert immediately whenever a test fails (rather than waiting for a full-run summary):**

```ts
// slack-failure-reporter.ts
import type { Reporter, TestCase, TestResult } from '@playwright/test/reporter';

class SlackFailureReporter implements Reporter {
  onTestEnd(test: TestCase, result: TestResult) {
    if (result.status === 'failed') {
      fetch(process.env.SLACK_WEBHOOK_URL!, {
        method: 'POST',
        body: JSON.stringify({ text: `❌ ${test.title} failed: ${result.error?.message}` }),
      });
    }
  }

  onEnd() {
    console.log('Test run finished.');
  }
}

export default SlackFailureReporter;
```

```ts
// playwright.config.ts
export default defineConfig({
  reporter: [['./slack-failure-reporter.ts']],
});
```

---

## 6. TestRail / Xray integration

**What it does:** Syncs Playwright test **results** back into a test-case-management tool (TestRail, Xray for Jira) so manual QA and automated results live in one place — useful when a company tracks test cases/requirements traceability in Jira/TestRail rather than purely in code.

**How it's usually done:** Tag each Playwright test with the corresponding TestRail case ID / Jira issue key, then use a custom reporter (or an existing community plugin) to push each result to that tool's API after the run.

**Real-time example — tagging a test with its TestRail case ID, then pushing results after the run:**

```ts
test('login shows an error for an invalid password', {
  tag: '@C1024', // TestRail case ID convention
}, async ({ page }) => {
  // ...
});
```

```ts
// testrail-reporter.ts — simplified custom reporter pushing results to TestRail's API
import type { Reporter, TestCase, TestResult } from '@playwright/test/reporter';

class TestRailReporter implements Reporter {
  private results: { caseId: string; status: number }[] = [];

  onTestEnd(test: TestCase, result: TestResult) {
    const match = test.tags.find(t => t.startsWith('@C'));
    if (match) {
      this.results.push({ caseId: match.slice(2), status: result.status === 'passed' ? 1 : 5 }); // TestRail status IDs
    }
  }

  async onEnd() {
    await fetch(`${process.env.TESTRAIL_URL}/api/v2/add_results_for_cases/R123`, {
      method: 'POST',
      headers: { Authorization: `Basic ${process.env.TESTRAIL_AUTH}` },
      body: JSON.stringify({ results: this.results.map(r => ({ case_id: r.caseId, status_id: r.status })) }),
    });
  }
}

export default TestRailReporter;
```

For Jira/Xray, the same pattern applies — tag tests with the Xray test key (e.g. `@XRAY-T101`), then push a JUnit or JSON result file to Xray's "import execution results" API endpoint (Xray natively accepts standard JUnit XML, which is why the JUnit reporter above is often reused here too).

---

## Quick summary

| Reporter | One-line real-world use |
|---|---|
| HTML (built-in) | Visual pass/fail report with screenshots and traces, for humans |
| JUnit XML | Feed CI systems (Jenkins, Azure DevOps) their native test-results view |
| JSON | Machine-readable output for custom dashboards, Slack bots, DB storage |
| Allure | Rich, filterable, cross-framework enterprise reporting dashboard |
| Custom (`Reporter` interface) | Any bespoke need — instant Slack alerts, internal formats |
| TestRail / Xray | Sync automated results back into a test-case-management/Jira workflow |
