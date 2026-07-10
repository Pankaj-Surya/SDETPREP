Act as a **Principal SDET, Test Architect, and Engineering Manager** with 20+ years of experience at FAANG, FinTech, Insurance, and SaaS product companies. You have deep hands-on expertise in test strategy, framework architecture, cloud platforms, AI-driven quality engineering, and developer-level backend engineering. You have personally mentored 50+ engineers from junior to staff level and have led QA org-wide transformations.

---

### MY PROFILE

- **Total experience:** 10+ years in QA/SDET
- **Current expertise:**
  - UI Automation: Selenium Java, Playwright TypeScript
  - API Testing: REST Assured, Postman, Playwright APIRequestContext
  - CI/CD: GitHub Actions, Jenkins, Azure DevOps
  - Databases: SQL (MySQL, PostgreSQL), basic NoSQL awareness
  - Design Patterns: Page Object Model
  - Scripting: Basic Bash, JavaScript/TypeScript
- **Target roles:** Senior SDET, Staff SDET, Principal SDET, Test Architect
- **Target companies:** FAANG, FinTech (Stripe, Razorpay, PayPal), Healthcare, SaaS (Salesforce, Atlassian), Startups

---

### TOPIC: Playwright TypeScript — Full Mastery

**FOCUS:** Interview Prep + Project-Only + Architecture + Quick Reference
**DEPTH:** Exhaustive
**LANGUAGE:** TypeScript (strict mode throughout)
**TIME:** 30 days
**INTEGRATE WITH:** Playwright, k6, AWS/Azure, AI/GenAI, Docker/Kubernetes, Observability, Monitoring
**OUTPUT:** Full + Sections + Code + Interview
**OUTPUT LANGUAGE:** Plain English — no fancy words, no fluff, just clear and direct

---

### WHAT I WANT YOU TO GENERATE

For **Playwright TypeScript**, produce a complete, expert-level technical deep-dive structured exactly as follows. Do not skip any section. Be specific, exhaustive, use real tools and real TypeScript code examples. Write everything in plain English.

---

## SECTION 1 — CONCEPT MASTERY

### 1.1 What Is Playwright? (First Principles)
- Explain what Playwright is and why Microsoft built it
- What problem does it solve that Selenium and Cypress did not?
- How does it fit into a modern SDET's daily work?
- Give a real-world analogy

### 1.2 Full Topic Hierarchy — Complete Sub-Topic Tree

Provide a complete, exhaustive nested topic tree covering every area a Principal SDET must know:

```
Playwright TypeScript
├── 1. Core Architecture
│   ├── Browser Engine Layer (Chromium, Firefox, WebKit)
│   ├── Protocol Layer (CDP, WebDriver BiDi)
│   ├── Node.js Runtime
│   └── TypeScript Strict Mode Setup
│
├── 2. Installation & Configuration
│   ├── npm init playwright@latest
│   ├── playwright.config.ts
│   │   ├── projects (multi-browser)
│   │   ├── use (baseURL, viewport, trace, video, screenshot)
│   │   ├── reporter (list, html, json, junit, allure)
│   │   ├── globalSetup / globalTeardown
│   │   ├── testDir, outputDir, timeout, retries
│   │   └── webServer (auto-start app under test)
│   ├── tsconfig.json (strict, paths, moduleResolution)
│   └── .env + dotenv config
│
├── 3. Locator Strategies
│   ├── Role-based (getByRole)
│   ├── Label-based (getByLabel)
│   ├── Placeholder (getByPlaceholder)
│   ├── Text (getByText)
│   ├── TestId (getByTestId)
│   ├── CSS / XPath (legacy — when to avoid)
│   ├── nth(), filter(), and(), or()
│   ├── Chained locators
│   └── Inside / has / hasText
│
├── 4. Actions & Interactions
│   ├── click, dblclick, hover, focus, blur
│   ├── fill, type, press, clear
│   ├── check, uncheck, selectOption
│   ├── dragAndDrop
│   ├── tap (mobile)
│   ├── keyboard (Keyboard API)
│   ├── mouse (Mouse API)
│   ├── scroll (scrollIntoViewIfNeeded, wheel)
│   └── File upload (setInputFiles)
│
├── 5. Assertions (expect)
│   ├── Web-first assertions (auto-retry)
│   │   ├── toBeVisible / toBeHidden
│   │   ├── toBeEnabled / toBeDisabled
│   │   ├── toHaveText / toContainText
│   │   ├── toHaveValue / toHaveCount
│   │   ├── toHaveAttribute / toHaveClass
│   │   ├── toHaveURL / toHaveTitle
│   │   └── toBeChecked / toBeFocused
│   ├── Snapshot assertions
│   │   ├── toMatchSnapshot (text)
│   │   └── toHaveScreenshot (visual)
│   ├── Soft assertions (expect.soft)
│   └── Custom matchers (expect.extend)
│
├── 6. Page & Browser APIs
│   ├── page.goto / page.reload / page.goBack
│   ├── page.waitForLoadState
│   ├── page.waitForSelector / waitForFunction
│   ├── page.evaluate / page.evaluateHandle
│   ├── page.exposeFunction
│   ├── page.addInitScript
│   ├── page.route / page.unroute
│   ├── page.on (events: request, response, console, dialog)
│   ├── Browser context (newContext, newPage)
│   ├── Cookies (add, clear, get)
│   └── LocalStorage / SessionStorage manipulation
│
├── 7. Network Layer
│   ├── Request interception (route)
│   ├── Response mocking (fulfill)
│   ├── Request modification (continue)
│   ├── HAR recording and replay
│   ├── Network throttling (via CDP)
│   ├── Abort requests
│   └── WebSocket interception
│
├── 8. API Testing (APIRequestContext)
│   ├── request.get / post / put / patch / delete
│   ├── request.newContext (baseURL, headers, auth)
│   ├── JSON body / form data
│   ├── File upload via API
│   ├── Response assertions
│   ├── API + UI hybrid tests
│   └── Auth flows (Bearer, Basic, OAuth2, API Key)
│
├── 9. Test Organisation
│   ├── test() / test.describe() / test.describe.parallel()
│   ├── test.beforeAll / afterAll / beforeEach / afterEach
│   ├── test.skip / test.only / test.fixme / test.fail
│   ├── test.step (named steps for reporting)
│   ├── Annotations (tag, slow, retries per test)
│   └── test.use (override config per describe)
│
├── 10. Fixtures
│   ├── Built-in fixtures (page, browser, context, request)
│   ├── Custom fixtures (extend test)
│   ├── Fixture scope (test / worker)
│   ├── Fixture composition
│   ├── Auto-use fixtures
│   └── Shared fixtures across files
│
├── 11. Design Patterns
│   ├── Page Object Model (POM)
│   ├── Service Object Model
│   ├── Component Object Model
│   ├── Fixture-First architecture
│   ├── Singleton (browser manager)
│   ├── Factory (page / data factory)
│   ├── Builder (test data builder)
│   ├── Strategy (login strategy)
│   ├── Facade (app facade)
│   └── Repository (locator repo)
│
├── 12. SOLID Principles in Playwright
│   ├── Single Responsibility
│   ├── Open/Closed
│   ├── Liskov Substitution
│   ├── Interface Segregation
│   └── Dependency Inversion
│
├── 13. Parallelism & Sharding
│   ├── fullyParallel mode
│   ├── workers config
│   ├── Worker-scoped fixtures
│   ├── Serial mode (test.describe.serial)
│   ├── Sharding (--shard N/M)
│   └── merge-reports CLI
│
├── 14. Debugging & Tooling
│   ├── Trace Viewer (trace: on/retain-on-failure)
│   ├── Playwright Inspector (PWDEBUG=1)
│   ├── Codegen (npx playwright codegen)
│   ├── VS Code extension
│   ├── --ui mode (interactive test runner)
│   └── Step-through debugging (breakpoints)
│
├── 15. Reporting
│   ├── Built-in HTML reporter
│   ├── JUnit XML (for CI)
│   ├── JSON reporter
│   ├── Allure integration
│   ├── Custom reporter (Reporter interface)
│   └── TestRail / Xray integration
│
├── 16. Advanced Topics
│   ├── Shadow DOM
│   ├── Iframes (frameLocator)
│   ├── Multiple tabs / popup handling
│   ├── Downloads
│   ├── File chooser dialog
│   ├── Authentication (storageState, globalSetup)
│   ├── Visual regression (toHaveScreenshot)
│   ├── Accessibility testing (axe-core + getByRole)
│   ├── Mobile emulation (devices)
│   └── Geolocation / permissions
│
├── 17. Data-Driven Testing
│   ├── test.each (inline data)
│   ├── CSV with papaparse
│   ├── Excel with xlsx / exceljs
│   ├── JSON fixtures
│   └── Faker.js for dynamic data
│
├── 18. RBAC Testing
│   ├── Role-specific storageState
│   ├── Fixture-per-role
│   ├── Permission matrix design
│   └── Multi-user concurrent tests
│
├── 19. Database Integration
│   ├── pg (PostgreSQL)
│   ├── mysql2
│   ├── mongoose (MongoDB)
│   ├── Seed in beforeAll, teardown in afterAll
│   └── Transaction rollback pattern
│
├── 20. CI/CD Integration
│   ├── GitHub Actions (matrix, sharding, artifacts)
│   ├── Azure DevOps Pipelines
│   ├── Jenkins
│   ├── Docker (mcr.microsoft.com/playwright)
│   ├── Kubernetes test pods
│   └── Quality gates (SonarQube, coverage)
│
├── 21. Cloud Testing
│   ├── AWS Device Farm
│   ├── Azure Container Instances
│   ├── BrowserStack Automate
│   └── LambdaTest
│
├── 22. Playwright MCP (2025)
│   ├── MCP server setup
│   ├── AI agent browser control
│   ├── Accessibility snapshot mode
│   └── Claude Desktop / Cursor integration
│
├── 23. Playwright CLI (2025)
│   ├── @playwright/cli install
│   ├── Skill-based workflows
│   ├── Token-efficient automation
│   └── Claude Code / Copilot integration
│
├── 24. AI/GenAI Integration
│   ├── LLM-generated test cases
│   ├── Self-healing locators
│   ├── AI test data generation
│   ├── CrewAI agent for test orchestration
│   └── n8n / LangFlow pipelines
│
├── 25. Observability & Monitoring
│   ├── Trace attachment in CI
│   ├── Log aggregation (CloudWatch, ELK)
│   ├── Custom metrics from test runs
│   ├── Datadog / New Relic APM correlation
│   └── Alerting on test failure trends
│
└── 26. Anti-Patterns
    ├── Hardcoded waits (page.waitForTimeout)
    ├── CSS/XPath selectors over role-based
    ├── No fixture isolation
    ├── Global state leaking between tests
    ├── No retry / flakiness strategy
    ├── Ignoring trace on failure
    └── No teardown / resource leaks
```

### 1.3 How Playwright Works Internally
- Playwright connects to browsers using the Chrome DevTools Protocol (CDP) for Chromium, and its own protocol bridges for Firefox and WebKit
- When you call `page.click()`, Playwright sends a protocol message to the browser, waits for the element to be stable, scrolls it into view, and fires the click — no artificial waits needed
- Each `BrowserContext` is isolated: separate cookies, localStorage, and network sessions — like opening a fresh incognito window
- `page.route()` intercepts at the network protocol level before the browser even sends the request
- Auto-waiting: every action queries the element's state (visible, enabled, stable, not obscured) in a retry loop until timeout — this eliminates 80% of flakiness from Selenium-style tests

### 1.4 Key Terminology Glossary

| Term | Definition | Why It Matters |
|------|-----------|----------------|
| BrowserContext | Isolated browser session (cookies, storage, auth) | Use one per test for full isolation |
| Locator | Lazy reference to an element — re-queries on every action | Prevents stale element errors |
| Auto-waiting | Playwright retries actions until element is actionable | Eliminates most artificial waits |
| Fixture | Dependency injection for test setup/teardown | Keeps test code clean and reusable |
| storageState | Saved cookies + localStorage snapshot | Re-use logged-in sessions across tests without re-logging in |
| route() | Intercept and mock HTTP requests | Isolate tests from backend or third-party APIs |
| Sharding | Split test suite across N machines | Reduce CI time from 30 mins to 5 mins |
| Worker | Playwright process running tests | Each worker gets its own browser — fully parallel |
| Trace | Recorded test execution (DOM, network, screenshots, timeline) | Debug failures without re-running |
| MCP | Model Context Protocol — AI agent browser control via Playwright | Connect AI agents to real browsers |

---

## SECTION 2 — PRACTICAL IMPLEMENTATION

### 2.1 Level 1 — Basic Test

```typescript
// tests/login.spec.ts
// Basic Playwright test — login flow
// Run: npx playwright test tests/login.spec.ts

import { test, expect } from '@playwright/test';

test('user can log in with valid credentials', async ({ page }) => {
  // Navigate to the app
  await page.goto('https://example.com/login');

  // Fill login form using role/label locators — NOT CSS selectors
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('Password123!');

  // Click the submit button by its role and name
  await page.getByRole('button', { name: 'Sign in' }).click();

  // Assert we landed on the dashboard
  // toHaveURL auto-retries — no explicit wait needed
  await expect(page).toHaveURL(/dashboard/);
  await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
});
```

### 2.2 Level 2 — Intermediate with Fixtures and Error Handling

```typescript
// fixtures/base.fixture.ts
// Custom fixture for authenticated page + API context

import { test as base, expect, APIRequestContext } from '@playwright/test';

type TestFixtures = {
  authenticatedPage: { page: import('@playwright/test').Page; userId: string };
  apiClient: APIRequestContext;
};

// Extend base test with custom fixtures
export const test = base.extend<TestFixtures>({

  // API client fixture — reusable across all tests
  apiClient: async ({ playwright }, use) => {
    const client = await playwright.request.newContext({
      baseURL: process.env.API_BASE_URL ?? 'https://api.example.com',
      extraHTTPHeaders: {
        Authorization: `Bearer ${process.env.API_TOKEN}`,
        'Content-Type': 'application/json',
      },
    });
    await use(client);
    await client.dispose(); // always clean up
  },

  // Authenticated page — login once, reuse session
  authenticatedPage: async ({ page, apiClient }, use) => {
    // Create user via API — faster than UI login for setup
    const response = await apiClient.post('/test/users', {
      data: { email: `test+${Date.now()}@example.com`, role: 'user' },
    });
    expect(response.ok()).toBeTruthy();
    const user = await response.json();

    // Set auth cookie directly — skip UI login entirely
    await page.context().addCookies([{
      name: 'auth_token',
      value: user.token,
      domain: 'example.com',
      path: '/',
    }]);

    await page.goto('/dashboard');
    await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();

    await use({ page, userId: user.id });

    // Teardown: delete test user after the test
    await apiClient.delete(`/test/users/${user.id}`);
  },
});

export { expect };
```

```typescript
// tests/dashboard.spec.ts
import { test, expect } from '../fixtures/base.fixture';

test.describe('Dashboard — authenticated user', () => {
  test('shows correct user data', async ({ authenticatedPage }) => {
    const { page } = authenticatedPage;
    await expect(page.getByTestId('user-email')).toContainText('@example.com');
  });

  test('can create a new item', async ({ authenticatedPage, apiClient }) => {
    const { page, userId } = authenticatedPage;
    await page.getByRole('button', { name: 'New item' }).click();
    await page.getByLabel('Item name').fill('My test item');
    await page.getByRole('button', { name: 'Save' }).click();
    await expect(page.getByText('My test item')).toBeVisible();
  });
});
```

### 2.3 Level 3 — Advanced with Page Object Model + Service Object

```typescript
// pages/login.page.ts
// Page Object — UI interactions only, no assertions

import { type Page, type Locator } from '@playwright/test';

export class LoginPage {
  private readonly emailInput: Locator;
  private readonly passwordInput: Locator;
  private readonly submitButton: Locator;
  private readonly errorMessage: Locator;

  constructor(private page: Page) {
    // All locators defined in constructor — single place to update
    this.emailInput    = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton  = page.getByRole('button', { name: 'Sign in' });
    this.errorMessage  = page.getByRole('alert');
  }

  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async login(email: string, password: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorText(): Promise<string> {
    return this.errorMessage.textContent() ?? '';
  }
}
```

```typescript
// services/user.service.ts
// Service Object — all API calls for user operations

import { type APIRequestContext } from '@playwright/test';

interface User {
  id: string;
  email: string;
  role: string;
  token: string;
}

export class UserService {
  constructor(private request: APIRequestContext) {}

  async createUser(email: string, role = 'user'): Promise<User> {
    const response = await this.request.post('/api/users', {
      data: { email, role },
    });
    if (!response.ok()) {
      throw new Error(`Failed to create user: ${response.status()}`);
    }
    return response.json();
  }

  async deleteUser(id: string): Promise<void> {
    await this.request.delete(`/api/users/${id}`);
  }

  async getUserById(id: string): Promise<User> {
    const response = await this.request.get(`/api/users/${id}`);
    return response.json();
  }
}
```

```typescript
// tests/login.spec.ts — using POM + Service Object
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/login.page';
import { UserService } from '../services/user.service';

test.describe('Login — POM + Service Object pattern', () => {
  let testUser: { id: string; email: string; token: string };

  test.beforeAll(async ({ request }) => {
    const userService = new UserService(request);
    testUser = await userService.createUser(`sdet+${Date.now()}@test.com`);
  });

  test.afterAll(async ({ request }) => {
    const userService = new UserService(request);
    await userService.deleteUser(testUser.id);
  });

  test('successful login redirects to dashboard', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login(testUser.email, 'Password123!');
    await expect(page).toHaveURL(/dashboard/);
  });

  test('wrong password shows error message', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login(testUser.email, 'wrong-password');
    const error = await loginPage.getErrorText();
    expect(error).toContain('Invalid credentials');
  });
});
```

### 2.4 Level 4 — Expert / Architect Grade

```typescript
// playwright.config.ts — production-ready, multi-env, sharded

import { defineConfig, devices } from '@playwright/test';
import dotenv from 'dotenv';

dotenv.config({ path: `.env.${process.env.ENV ?? 'local'}` });

export default defineConfig({
  testDir: './tests',
  outputDir: './test-results',
  timeout: 30_000,
  expect: { timeout: 10_000 },
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : undefined,
  fullyParallel: true,

  reporter: [
    ['list'],
    ['html', { outputFolder: 'playwright-report', open: 'never' }],
    ['junit', { outputFile: 'test-results/results.xml' }],
    ['./reporters/custom.reporter.ts'],
    // Allure: install allure-playwright first
    // ['allure-playwright', { detail: true, outputFolder: 'allure-results' }],
  ],

  use: {
    baseURL: process.env.BASE_URL ?? 'http://localhost:3000',
    headless: true,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure',
    actionTimeout: 15_000,
    navigationTimeout: 30_000,
  },

  projects: [
    // Setup project — runs before all others
    { name: 'setup', testMatch: '**/global.setup.ts' },

    // Browser matrix
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
      dependencies: ['setup'],
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
      dependencies: ['setup'],
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
      dependencies: ['setup'],
    },

    // Mobile
    {
      name: 'mobile-chrome',
      use: { ...devices['Pixel 5'] },
      dependencies: ['setup'],
    },

    // API-only tests — no browser overhead
    {
      name: 'api',
      testMatch: '**/api/**/*.spec.ts',
      use: { baseURL: process.env.API_BASE_URL },
    },
  ],

  webServer: process.env.CI ? undefined : {
    command: 'npm run start:test',
    port: 3000,
    reuseExistingServer: true,
  },
});
```

```typescript
// tests/global.setup.ts — run once before entire suite

import { test as setup, expect } from '@playwright/test';
import path from 'path';

// Auth state file — shared across all workers
const authFile = path.join(__dirname, '../.auth/user.json');
const adminFile = path.join(__dirname, '../.auth/admin.json');

setup('authenticate as regular user', async ({ page, request }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.USER_EMAIL!);
  await page.getByLabel('Password').fill(process.env.USER_PASSWORD!);
  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page).toHaveURL(/dashboard/);
  // Save auth state to file — reused by all test workers
  await page.context().storageState({ path: authFile });
});

setup('authenticate as admin', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.ADMIN_EMAIL!);
  await page.getByLabel('Password').fill(process.env.ADMIN_PASSWORD!);
  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page).toHaveURL(/admin/);
  await page.context().storageState({ path: adminFile });
});
```

```typescript
// fixtures/rbac.fixture.ts — role-based fixtures for RBAC testing

import { test as base } from '@playwright/test';
import path from 'path';

type RbacFixtures = {
  userPage: import('@playwright/test').Page;
  adminPage: import('@playwright/test').Page;
};

export const test = base.extend<RbacFixtures>({
  userPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: path.join(__dirname, '../.auth/user.json'),
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  },

  adminPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: path.join(__dirname, '../.auth/admin.json'),
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  },
});
```

```typescript
// tests/data-driven/payments.spec.ts — CSV + Excel driven tests

import { test, expect } from '@playwright/test';
import Papa from 'papaparse';
import fs from 'fs';
import path from 'path';

// Load CSV test data at file load time
const csvRaw = fs.readFileSync(
  path.join(__dirname, '../../test-data/payments.csv'),
  'utf8'
);
const { data: paymentCases } = Papa.parse<{
  amount: string;
  currency: string;
  expectedResult: string;
}>(csvRaw, { header: true, skipEmptyLines: true });

// Run one test per CSV row
for (const testCase of paymentCases) {
  test(`payment: ${testCase.amount} ${testCase.currency} → ${testCase.expectedResult}`, async ({ page }) => {
    await page.goto('/payments/new');
    await page.getByLabel('Amount').fill(testCase.amount);
    await page.getByLabel('Currency').selectOption(testCase.currency);
    await page.getByRole('button', { name: 'Pay' }).click();

    if (testCase.expectedResult === 'success') {
      await expect(page.getByText('Payment successful')).toBeVisible();
    } else {
      await expect(page.getByRole('alert')).toContainText('failed');
    }
  });
}
```

```typescript
// reporters/custom.reporter.ts — custom reporter for Slack + test metrics

import type {
  Reporter,
  TestCase,
  TestResult,
  FullConfig,
  Suite,
  FullResult,
} from '@playwright/test/reporter';

interface TestMetrics {
  passed: number;
  failed: number;
  skipped: number;
  totalDuration: number;
  failures: { title: string; error: string }[];
}

export default class CustomReporter implements Reporter {
  private metrics: TestMetrics = {
    passed: 0, failed: 0, skipped: 0, totalDuration: 0, failures: [],
  };

  onBegin(config: FullConfig, suite: Suite): void {
    console.log(`\nStarting ${suite.allTests().length} tests...`);
  }

  onTestEnd(test: TestCase, result: TestResult): void {
    this.metrics.totalDuration += result.duration;
    if (result.status === 'passed') this.metrics.passed++;
    else if (result.status === 'failed') {
      this.metrics.failed++;
      this.metrics.failures.push({
        title: test.title,
        error: result.error?.message ?? 'Unknown error',
      });
    } else {
      this.metrics.skipped++;
    }
  }

  async onEnd(result: FullResult): Promise<void> {
    const summary = `Tests: ${this.metrics.passed} passed, ${this.metrics.failed} failed, ${this.metrics.skipped} skipped | Duration: ${(this.metrics.totalDuration / 1000).toFixed(1)}s`;
    console.log(summary);

    // Post to Slack if failures exist and SLACK_WEBHOOK is set
    if (this.metrics.failed > 0 && process.env.SLACK_WEBHOOK) {
      const message = {
        text: `:red_circle: Playwright tests failed\n${summary}`,
        attachments: this.metrics.failures.map(f => ({
          color: 'danger',
          title: f.title,
          text: f.error.slice(0, 300),
        })),
      };
      await fetch(process.env.SLACK_WEBHOOK, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(message),
      });
    }
  }
}
```

### 2.5 Folder Structure

```
playwright-enterprise/
├── .auth/                    # storageState files per role (gitignored)
│   ├── user.json
│   └── admin.json
├── .env.local                # local env vars (gitignored)
├── .env.staging              # staging env vars
├── .env.ci                   # CI env vars (secrets injected by pipeline)
├── fixtures/
│   ├── base.fixture.ts       # API client + authenticated page fixtures
│   ├── rbac.fixture.ts       # Role-based fixtures (user, admin, viewer)
│   └── db.fixture.ts         # Database connection + seed/teardown
├── pages/
│   ├── base.page.ts          # Base class with shared helpers
│   ├── login.page.ts
│   ├── dashboard.page.ts
│   └── components/
│       ├── nav.component.ts  # Reusable UI component objects
│       └── modal.component.ts
├── services/
│   ├── user.service.ts       # User API calls
│   ├── payment.service.ts    # Payment API calls
│   └── auth.service.ts       # Auth API calls
├── tests/
│   ├── global.setup.ts       # Runs once before all tests
│   ├── global.teardown.ts    # Runs once after all tests
│   ├── e2e/                  # Full end-to-end flows
│   ├── api/                  # API-only tests
│   ├── visual/               # Screenshot regression tests
│   ├── performance/          # Playwright + k6 hybrid
│   └── security/             # Auth, RBAC, injection tests
├── test-data/
│   ├── payments.csv          # CSV test data
│   ├── users.xlsx            # Excel test data
│   └── products.json         # JSON fixture data
├── reporters/
│   └── custom.reporter.ts    # Custom Slack + metrics reporter
├── utils/
│   ├── db.helper.ts          # PostgreSQL / MySQL helpers
│   ├── file.helper.ts        # CSV / Excel reader helpers
│   ├── retry.helper.ts       # Custom retry logic
│   └── logger.ts             # Structured logger
├── playwright.config.ts      # Main config
├── tsconfig.json
├── package.json
└── docker-compose.yml        # App + DB + mock server for local dev
```

---

## SECTION 3 — DESIGN PATTERNS & ARCHITECTURE

### 3.1 Design Patterns

**Page Object Model** — one class per page, UI interactions only, no assertions inside

```typescript
// pages/base.page.ts
import { type Page } from '@playwright/test';

export abstract class BasePage {
  constructor(protected page: Page) {}

  // Shared helpers all pages can use
  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }

  async getPageTitle(): Promise<string> {
    return this.page.title();
  }
}
```

**Factory Pattern** — create page objects or test data without new() scattered everywhere

```typescript
// factories/page.factory.ts
import { type Page } from '@playwright/test';
import { LoginPage } from '../pages/login.page';
import { DashboardPage } from '../pages/dashboard.page';

export class PageFactory {
  constructor(private page: Page) {}

  login(): LoginPage          { return new LoginPage(this.page); }
  dashboard(): DashboardPage  { return new DashboardPage(this.page); }
}
```

**Builder Pattern** — build complex test data objects cleanly

```typescript
// builders/user.builder.ts
interface UserPayload {
  email: string;
  firstName: string;
  lastName: string;
  role: string;
  active: boolean;
}

export class UserBuilder {
  private user: UserPayload = {
    email: `test+${Date.now()}@example.com`,
    firstName: 'Test',
    lastName: 'User',
    role: 'viewer',
    active: true,
  };

  withEmail(email: string): this      { this.user.email = email; return this; }
  withRole(role: string): this         { this.user.role = role; return this; }
  withFirstName(name: string): this    { this.user.firstName = name; return this; }
  asInactive(): this                   { this.user.active = false; return this; }

  build(): UserPayload { return { ...this.user }; }
}

// Usage in tests:
// const admin = new UserBuilder().withRole('admin').withEmail('admin@test.com').build();
```

**Strategy Pattern** — swappable login methods

```typescript
// strategies/login.strategy.ts
import { type Page } from '@playwright/test';

interface LoginStrategy {
  login(page: Page, credentials: { email: string; password: string }): Promise<void>;
}

// Standard form login
export class FormLoginStrategy implements LoginStrategy {
  async login(page: Page, credentials: { email: string; password: string }): Promise<void> {
    await page.goto('/login');
    await page.getByLabel('Email').fill(credentials.email);
    await page.getByLabel('Password').fill(credentials.password);
    await page.getByRole('button', { name: 'Sign in' }).click();
  }
}

// SSO login (skip form, use token directly)
export class SSOLoginStrategy implements LoginStrategy {
  async login(page: Page, credentials: { email: string; password: string }): Promise<void> {
    const token = await fetch('/api/auth/sso-token', {
      method: 'POST',
      body: JSON.stringify(credentials),
    }).then(r => r.json()).then(d => d.token);

    await page.context().addCookies([{
      name: 'sso_token', value: token,
      domain: new URL(page.url()).hostname, path: '/',
    }]);
    await page.goto('/dashboard');
  }
}
```

### 3.2 SOLID Principles

**Single Responsibility:** `LoginPage` only handles login UI. `UserService` only handles user API calls. Never mix them.

**Open/Closed:** Add new login strategies by creating a new class that implements `LoginStrategy` — never edit the existing strategies.

**Liskov Substitution:** `AdminPage extends DashboardPage` — any test using `DashboardPage` should work with `AdminPage` without breaking.

**Interface Segregation:** Don't create one giant `IPageActions` interface. Create small focused ones: `INavigable`, `IFormFillable`, `IAssertable`.

**Dependency Inversion:** Tests depend on `LoginStrategy` interface, not `FormLoginStrategy` directly. Inject the strategy via fixture.

### 3.3 Architecture Diagram

```
╔══════════════════════════════════════════════════════════════╗
║                   TEST EXECUTION LAYER                       ║
║  GitHub Actions / Azure DevOps / Jenkins                     ║
║  ┌────────────┐  ┌─────────────┐  ┌────────────────────┐    ║
║  │  Shard 1   │  │  Shard 2    │  │    Shard 3         │    ║
║  │ (Worker 1) │  │ (Worker 2)  │  │  (Worker 3)        │    ║
║  └─────┬──────┘  └──────┬──────┘  └────────┬───────────┘    ║
╚════════╪════════════════╪══════════════════╪════════════════╝
         │                │                  │
╔════════╪════════════════╪══════════════════╪════════════════╗
║        ▼                ▼                  ▼                 ║
║              PLAYWRIGHT TEST RUNNER                          ║
║  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   ║
║  │  Fixtures    │  │  Page Objs   │  │  Service Objects │   ║
║  │  (setup/     │  │  (UI layer)  │  │  (API layer)     │   ║
║  │   teardown)  │  │              │  │                  │   ║
║  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘   ║
╚═════════╪════════════════╪═══════════════════╪═════════════╝
          │                │                   │
    ┌─────▼──────┐  ┌──────▼──────┐  ┌─────────▼───────┐
    │  Browser   │  │  App Under  │  │   Backend API   │
    │  (Chromium │  │   Test      │  │   + Database    │
    │  Firefox   │  │  (Docker)   │  │   (Docker)      │
    │  WebKit)   │  │             │  │                 │
    └─────┬──────┘  └─────────────┘  └─────────────────┘
          │
    ┌─────▼──────────────────────────────────┐
    │           OBSERVABILITY                │
    │  Allure Report │ Trace Viewer          │
    │  CloudWatch    │ Slack Notifications   │
    │  Custom Metrics│ TestRail Sync         │
    └────────────────────────────────────────┘
```

### 3.4 Architecture Decision Records

```
ADR-001: Use Fixture-First Architecture Over Test-Level Setup
Status: Accepted
Context: Teams write setup code inside test.beforeEach causing duplication and coupling
Decision: All setup/teardown goes into typed fixtures using test.extend()
Consequences: Tests become 3-5 lines. Setup is reusable. Errors in setup fail fast.
Alternatives: beforeEach hooks — rejected because they cause hidden coupling

ADR-002: Use storageState for Auth, Not UI Login in Every Test
Status: Accepted
Context: Login UI adds 3-5 seconds per test. 500 tests = 40 mins just on login.
Decision: globalSetup logs in once, saves storageState to .auth/*.json, tests reuse it
Consequences: Suite runs 60% faster. Auth tested separately in dedicated spec.
Alternatives: Cookie injection per test — rejected, still requires login endpoint calls

ADR-003: Role-Based Locators Over CSS/XPath
Status: Accepted
Context: CSS selectors break when developers rename classes or restructure DOM
Decision: Use getByRole, getByLabel, getByTestId as primary locator strategy
Consequences: Tests survive most UI refactors. Better accessibility coverage.
Alternatives: data-testid only — rejected, misses accessibility validation
```

---

## SECTION 4 — ENTERPRISE USE CASES

### 4.1 Use Case Matrix

| Industry | Use Case | Specific Challenge | Approach |
|----------|----------|-------------------|----------|
| FinTech | Payment flow testing | Non-deterministic transaction IDs, 3DS auth popups | Mock payment gateway via route(), handle popups with page.waitForEvent('popup') |
| FinTech | KYC document upload | File upload + async OCR processing | setInputFiles + poll API for status via waitForFunction |
| Healthcare | Patient data HIPAA compliance | PII in logs, screenshots must be scrubbed | Custom reporter to redact PII, disable screenshot on auth pages |
| Healthcare | EHR form with 50+ fields | Complex form interactions, date pickers, dropdowns | Builder pattern for form data, component objects per section |
| SaaS | Multi-tenant isolation | Test user A cannot see tenant B data | RBAC fixture matrix with assertion on 403 responses |
| SaaS | Bulk import (10,000 records) | Async processing, progress bar, completion webhook | API seed + waitForResponse + WebSocket listener |
| FAANG | A/B test coverage | Same URL renders different variants | Detect variant in DOM, conditional test branching, flag both |
| Startup | Cross-browser visual parity | CSS renders differently in Safari | toHaveScreenshot with per-browser golden files |

### 4.2 Hands-On Project — FinTech Payment Automation Suite

**Project Title:** Stripe-Style Payment Gateway Test Suite

**Business Context:** You are the lead SDET at a FinTech startup processing $10M/day. A bug in the payment flow costs $50K per minute. You need a test suite that runs in under 5 minutes on every PR, covering happy path, edge cases, and failure scenarios.

**Tech Stack:**
- Playwright 1.48+ with TypeScript 5.x
- Node.js 20
- PostgreSQL (via Docker)
- WireMock (API mock server via Docker)
- GitHub Actions
- k6 (performance smoke test post-deploy)

**Step-by-Step:**

```bash
# 1. Clone and install
git clone https://github.com/your-org/payment-suite
cd payment-suite
npm install

# 2. Install Playwright browsers
npx playwright install

# 3. Start the test environment
docker-compose up -d

# 4. Verify services are running
curl http://localhost:8080/__admin/  # WireMock
curl http://localhost:3000/health    # App

# 5. Run the full suite
npx playwright test

# 6. View report
npx playwright show-report
```

```typescript
// tests/e2e/payment-flow.spec.ts

import { test, expect } from '../../fixtures/base.fixture';
import Papa from 'papaparse';
import fs from 'fs';
import path from 'path';

// Load payment scenarios from CSV
const csv = fs.readFileSync(path.join(__dirname, '../../test-data/payment-cases.csv'), 'utf8');
const { data } = Papa.parse<{ card: string; amount: string; expectedStatus: string }>(
  csv, { header: true, skipEmptyLines: true }
);

// Intercept and mock Stripe API
test.beforeEach(async ({ page }) => {
  await page.route('**/api/payments/process', async route => {
    const body = await route.request().postDataJSON();
    // Return success for valid cards, failure for test cards starting with 4000
    const isDeclined = body.cardNumber.startsWith('4000');
    await route.fulfill({
      status: isDeclined ? 402 : 200,
      contentType: 'application/json',
      body: JSON.stringify({
        status: isDeclined ? 'declined' : 'success',
        transactionId: `txn_${Date.now()}`,
      }),
    });
  });
});

for (const scenario of data) {
  test(`payment: ${scenario.card} amount ${scenario.amount} → ${scenario.expectedStatus}`, async ({ page, authenticatedPage }) => {
    const { page: authPage } = authenticatedPage;
    
    await authPage.goto('/checkout');
    await authPage.getByLabel('Card number').fill(scenario.card);
    await authPage.getByLabel('Amount').fill(scenario.amount);
    await authPage.getByRole('button', { name: 'Pay now' }).click();

    if (scenario.expectedStatus === 'success') {
      await expect(authPage.getByTestId('payment-success')).toBeVisible();
      await expect(authPage.getByTestId('transaction-id')).toHaveText(/txn_/);
    } else {
      await expect(authPage.getByRole('alert')).toContainText('declined');
    }
  });
}
```

```yaml
# test-data/payment-cases.csv
card,amount,expectedStatus
4111111111111111,100.00,success
4000000000000002,50.00,declined
4111111111111111,0.00,validation-error
4111111111111111,999999.99,limit-exceeded
```

**CI/CD — GitHub Actions:**

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]  # Run 4 shards in parallel

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium

      - name: Start test environment
        run: docker-compose up -d --wait

      - name: Run Playwright tests (shard ${{ matrix.shard }}/4)
        run: npx playwright test --shard=${{ matrix.shard }}/4
        env:
          CI: true
          BASE_URL: http://localhost:3000
          API_BASE_URL: http://localhost:3001
          USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-results-shard-${{ matrix.shard }}
          path: |
            playwright-report/
            test-results/
          retention-days: 7

  merge-reports:
    needs: test
    runs-on: ubuntu-latest
    if: always()
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci

      - name: Download all shard reports
        uses: actions/download-artifact@v4
        with:
          path: all-results/

      - name: Merge reports
        run: npx playwright merge-reports --reporter html ./all-results

      - name: Upload merged report
        uses: actions/upload-artifact@v4
        with:
          name: playwright-merged-report
          path: playwright-report/

      - name: Notify Slack on failure
        if: failure()
        uses: slackapi/slack-github-action@v1.26.0
        with:
          payload: '{"text":"Playwright tests failed on ${{ github.ref }} - ${{ github.run_url }}"}'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

**Extension Challenges:**
1. Add k6 smoke test that runs after deployment and asserts p95 latency < 200ms on the payment API
2. Add visual regression tests for the checkout page across 3 browsers
3. Add a Pact consumer test that validates the payment API response schema matches the provider

### 4.3 Anti-Patterns to Avoid

**Anti-Pattern 1: Hardcoded waits**
```typescript
// BAD — arbitrary timeout, flaky on slow machines, slow on fast ones
await page.waitForTimeout(3000);
await page.click('#submit');

// GOOD — wait for the actual condition
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
await page.getByRole('button', { name: 'Submit' }).click();
```

**Anti-Pattern 2: CSS selectors everywhere**
```typescript
// BAD — breaks when developer renames class
await page.click('.btn-primary.submit-btn');

// GOOD — tied to what the user actually sees
await page.getByRole('button', { name: 'Submit' }).click();
```

**Anti-Pattern 3: Assertions inside Page Objects**
```typescript
// BAD — page objects should not assert
class LoginPage {
  async login(email: string, pass: string) {
    await this.page.fill('#email', email);
    await this.page.fill('#pass', pass);
    await this.page.click('#login');
    await expect(this.page).toHaveURL('/dashboard'); // WRONG — assertion in POM
  }
}

// GOOD — assert in the test, POM just performs actions
class LoginPage {
  async login(email: string, pass: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(pass);
    await this.submitButton.click();
  }
}
// In the test:
await loginPage.login(email, pass);
await expect(page).toHaveURL('/dashboard'); // assertion where it belongs
```

**Anti-Pattern 4: Shared browser context between tests**
```typescript
// BAD — test 2 inherits cookies/state from test 1
let page: Page;
test.beforeAll(async ({ browser }) => { page = await browser.newPage(); });

// GOOD — each test gets its own context (Playwright default)
test('test 1', async ({ page }) => { /* fresh context */ });
test('test 2', async ({ page }) => { /* fresh context */ });
```

**Anti-Pattern 5: No cleanup of test data**
```typescript
// BAD — test data piles up in the database, breaking other tests
test('create user', async ({ request }) => {
  await request.post('/api/users', { data: { email: 'test@example.com' } });
  // no cleanup — next run fails because email already exists
});

// GOOD — always clean up
test('create user', async ({ request }) => {
  const res = await request.post('/api/users', { data: { email: `t+${Date.now()}@x.com` } });
  const { id } = await res.json();
  test.afterEach(async () => { await request.delete(`/api/users/${id}`); });
});
```

---

## SECTION 5 — CI/CD & PIPELINE INTEGRATION

### 5.1 Azure DevOps Pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include: [main, develop, release/*]

pool:
  vmImage: ubuntu-latest

variables:
  - group: playwright-secrets        # Key Vault linked variable group
  - name: NODE_VERSION
    value: '20'

stages:
  - stage: Test
    displayName: Playwright E2E Tests
    jobs:
      - job: RunTests
        strategy:
          parallel: 4               # 4 agents running shards
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(NODE_VERSION)

          - script: npm ci
            displayName: Install dependencies

          - script: npx playwright install --with-deps chromium
            displayName: Install browsers

          - script: docker-compose up -d --wait
            displayName: Start test services

          - script: |
              SHARD_INDEX=$(($(System.JobPositionInPhase)))
              npx playwright test --shard=$SHARD_INDEX/4
            displayName: Run tests (shard $(System.JobPositionInPhase)/4)
            env:
              CI: true
              BASE_URL: $(BASE_URL)
              USER_EMAIL: $(TEST_USER_EMAIL)
              USER_PASSWORD: $(TEST_USER_PASSWORD)

          - task: PublishTestResults@2
            condition: always()
            inputs:
              testResultsFormat: JUnit
              testResultsFiles: test-results/results.xml
              testRunTitle: Playwright Shard $(System.JobPositionInPhase)

          - task: PublishPipelineArtifact@1
            condition: always()
            inputs:
              targetPath: playwright-report
              artifact: playwright-report-$(System.JobPositionInPhase)

  - stage: QualityGate
    dependsOn: Test
    jobs:
      - job: Gate
        steps:
          - script: npx playwright merge-reports --reporter html ./all-results
            displayName: Merge shard reports
          - script: |
              FAILED=$(cat test-results/results.xml | grep -c 'failure' || true)
              if [ "$FAILED" -gt "0" ]; then exit 1; fi
            displayName: Fail if any test failed
```

### 5.2 Docker Setup

```dockerfile
# Dockerfile
FROM mcr.microsoft.com/playwright:v1.48.0-jammy

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

# Run as non-root for security
RUN groupadd -r playwright && useradd -r -g playwright playwright
RUN chown -R playwright:playwright /app
USER playwright

CMD ["npx", "playwright", "test", "--reporter=junit"]
```

```yaml
# docker-compose.yml
services:
  app:
    image: your-app:latest
    ports: ['3000:3000']
    environment:
      DATABASE_URL: postgres://test:test@db:5432/testdb
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: testdb
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U test']
      interval: 5s
      retries: 5

  wiremock:
    image: wiremock/wiremock:3.3.1
    ports: ['8080:8080']
    volumes:
      - ./wiremock:/home/wiremock

  playwright:
    build: .
    depends_on: [app, db, wiremock]
    environment:
      BASE_URL: http://app:3000
      API_BASE_URL: http://wiremock:8080
    volumes:
      - ./playwright-report:/app/playwright-report
      - ./test-results:/app/test-results
```

---

## SECTION 6 — DATABASE INTEGRATION

```typescript
// fixtures/db.fixture.ts
import { test as base } from '@playwright/test';
import { Pool, type PoolClient } from 'pg';

type DbFixtures = {
  dbClient: PoolClient;
  seedUser: { id: string; email: string };
};

// Worker-scoped pool — one connection pool per worker process
const pool = new Pool({
  connectionString: process.env.DATABASE_URL ?? 'postgres://test:test@localhost:5432/testdb',
  max: 5,
});

export const test = base.extend<DbFixtures, { dbPool: Pool }>({
  // Worker-scoped pool — created once per worker
  dbPool: [async ({}, use) => {
    await use(pool);
    await pool.end();
  }, { scope: 'worker' }],

  // Test-scoped client — each test gets its own client + transaction
  dbClient: async ({ dbPool }, use) => {
    const client = await dbPool.connect();
    await client.query('BEGIN');  // wrap test in transaction
    await use(client);
    await client.query('ROLLBACK');  // always rollback — no test data left behind
    client.release();
  },

  // Seed a test user in the DB before the test
  seedUser: async ({ dbClient }, use) => {
    const email = `seed+${Date.now()}@test.com`;
    const { rows } = await dbClient.query<{ id: string; email: string }>(
      `INSERT INTO users (email, role, created_at)
       VALUES ($1, 'user', NOW())
       RETURNING id, email`,
      [email]
    );
    await use(rows[0]);
    // No explicit delete — transaction ROLLBACK handles cleanup
  },
});
```

---

## SECTION 7 — CLOUD INTEGRATION

### 7.1 AWS — Run Playwright in ECS Fargate

```yaml
# terraform/playwright-runner.tf (simplified)
resource "aws_ecs_task_definition" "playwright" {
  family                   = "playwright-tests"
  requires_compatibilities = ["FARGATE"]
  network_mode             = "awsvpc"
  cpu                      = "2048"
  memory                   = "4096"
  execution_role_arn       = aws_iam_role.ecs_task_exec.arn

  container_definitions = jsonencode([{
    name  = "playwright"
    image = "${aws_ecr_repository.playwright.repository_url}:latest"
    environment = [
      { name = "BASE_URL", value = var.app_url },
      { name = "CI",       value = "true" }
    ]
    secrets = [
      { name = "USER_EMAIL",    valueFrom = aws_ssm_parameter.test_user_email.arn },
      { name = "USER_PASSWORD", valueFrom = aws_ssm_parameter.test_user_pass.arn }
    ]
    logConfiguration = {
      logDriver = "awslogs"
      options = {
        "awslogs-group"  = "/ecs/playwright"
        "awslogs-region" = var.aws_region
      }
    }
  }])
}
```

### 7.2 Azure Container Instances

```bash
# Run Playwright tests in ACI — useful for on-demand test execution
az container create \
  --resource-group rg-testing \
  --name playwright-runner \
  --image yourregistry.azurecr.io/playwright:latest \
  --cpu 2 --memory 4 \
  --environment-variables \
    BASE_URL=https://staging.example.com \
    CI=true \
  --secure-environment-variables \
    USER_EMAIL="$TEST_USER_EMAIL" \
    USER_PASSWORD="$TEST_USER_PASSWORD" \
  --restart-policy Never \
  --azure-log-analytics-workspace-id "$WORKSPACE_ID" \
  --azure-log-analytics-workspace-key "$WORKSPACE_KEY"

# Wait and get exit code
az container wait --name playwright-runner --resource-group rg-testing --for-exit-code 0
```

---

## SECTION 8 — AI & ADVANCED TOOLING

### 8.1 Playwright MCP Integration

```bash
# Install and run Playwright MCP server
npx @playwright/mcp@latest --port 8931

# In Claude Desktop config (claude_desktop_config.json):
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

Now Claude can control a real browser. Use cases for SDETs:
- "Navigate to the checkout page and tell me what accessibility issues exist"
- "Fill the payment form with test card 4111111111111111 and take a screenshot"
- "Run the login flow and generate a Playwright test from the steps you took"

### 8.2 Playwright CLI (2025 Agent Mode)

```bash
# Install CLI
npm i -g @playwright/cli@latest

# Run a skill — token-efficient, purpose-built for coding agents
playwright run-skill "fill-and-submit-form" --url https://example.com/login
```

### 8.3 AI-Augmented Testing with Claude

Prompt template for generating Playwright tests with Claude:

```
You are a Senior SDET. Generate a Playwright TypeScript test for the following user story:
[paste user story / acceptance criteria]

Requirements:
- Use getByRole / getByLabel locators only
- Use fixture-based setup with test.extend()
- Include API setup via APIRequestContext to seed test data
- Include both happy path and at least 2 failure scenarios
- Use the Builder pattern for test data construction
- Follow this folder structure: [paste your structure]
- Use strict TypeScript, no 'any' types
```

---

## SECTION 9 — PERFORMANCE & SECURITY

### 9.1 k6 Smoke Test Post-Deploy

```javascript
// k6/smoke.js — run after every deployment
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 10,
  duration: '1m',
  thresholds: {
    http_req_duration: ['p95<500', 'p99<1000'],  // 95% under 500ms
    http_req_failed: ['rate<0.01'],               // less than 1% errors
  },
};

export default function () {
  const res = http.get(`${__ENV.BASE_URL}/api/health`);
  check(res, {
    'status is 200': r => r.status === 200,
    'response time OK': r => r.timings.duration < 500,
  });
  sleep(1);
}
```

### 9.2 Security Testing with Playwright

```typescript
// tests/security/auth-bypass.spec.ts

import { test, expect } from '@playwright/test';

test.describe('Security — auth bypass attempts', () => {
  test('unauthenticated user cannot access protected routes', async ({ page }) => {
    // Try accessing protected page without auth
    await page.goto('/admin/users');
    // Should redirect to login or return 401
    await expect(page).toHaveURL(/login/);
  });

  test('regular user cannot access admin endpoints via API', async ({ request }) => {
    // Using a user token, try to hit admin API
    const response = await request.get('/api/admin/users', {
      headers: { Authorization: `Bearer ${process.env.USER_TOKEN}` },
    });
    expect(response.status()).toBe(403);
  });

  test('SQL injection attempt in search field is rejected', async ({ page }) => {
    await page.goto('/search');
    // Attempt SQL injection
    await page.getByLabel('Search').fill("'; DROP TABLE users; --");
    await page.getByRole('button', { name: 'Search' }).click();
    // App should return empty results or validation error, not 500
    await expect(page.getByRole('alert')).not.toBeVisible();
    await expect(page).not.toHaveURL(/error/);
  });
});
```

---

## SECTION 10 — OBSERVABILITY & MONITORING

```typescript
// utils/logger.ts — structured logger for test runs

import fs from 'fs';
import path from 'path';

type LogLevel = 'INFO' | 'WARN' | 'ERROR' | 'DEBUG';

interface LogEntry {
  timestamp: string;
  level: LogLevel;
  testId?: string;
  message: string;
  data?: unknown;
}

export class TestLogger {
  private logFile: fs.WriteStream;

  constructor(private testId: string) {
    const logDir = path.join(process.cwd(), 'logs');
    if (!fs.existsSync(logDir)) fs.mkdirSync(logDir, { recursive: true });
    this.logFile = fs.createWriteStream(
      path.join(logDir, `${this.testId}-${Date.now()}.log`),
      { flags: 'a' }
    );
  }

  log(level: LogLevel, message: string, data?: unknown): void {
    const entry: LogEntry = {
      timestamp: new Date().toISOString(),
      level,
      testId: this.testId,
      message,
      data,
    };
    // Write structured JSON log — easy to parse in CloudWatch / ELK
    this.logFile.write(JSON.stringify(entry) + '\n');
    if (level === 'ERROR') console.error(`[${level}] ${message}`, data ?? '');
  }

  info(msg: string, data?: unknown)  { this.log('INFO',  msg, data); }
  warn(msg: string, data?: unknown)  { this.log('WARN',  msg, data); }
  error(msg: string, data?: unknown) { this.log('ERROR', msg, data); }
  debug(msg: string, data?: unknown) { this.log('DEBUG', msg, data); }

  close(): void { this.logFile.end(); }
}
```

---

## SECTION 11 — INTERVIEW PREPARATION

### 11.1 Senior SDET Questions (5–8 YOE)

**Q1: What is the difference between a Locator and an ElementHandle in Playwright?**
A: A Locator is a lazy, re-queryable reference — every time you act on it, it queries the DOM fresh. An ElementHandle is a live reference to a specific DOM node at a point in time — it goes stale if the page changes. Always use Locators. ElementHandles exist for edge cases like evaluating complex expressions.

**Q2: How does auto-waiting work in Playwright?**
A: Before every action (click, fill, etc.), Playwright checks a set of actionability conditions: the element must be visible, enabled, stable (not moving), and not obscured. It retries this check in a loop until the timeout. This replaces explicit waits with smart waiting that is both faster and more reliable.

**Q3: How do you handle authentication in Playwright so you don't log in for every test?**
A: Use `globalSetup` to log in once and save `storageState` (cookies + localStorage) to a file. Configure each test project to use that file via `use: { storageState: '.auth/user.json' }`. Tests start already authenticated — no login round-trip needed.

**Q4: What is the difference between `test.describe.parallel()` and `fullyParallel: true`?**
A: `fullyParallel: true` runs all tests across all files in parallel — each test gets its own worker. `test.describe.parallel()` runs tests inside a specific describe block in parallel but keeps other describe blocks sequential. Use `fullyParallel` for most suites; use `serial` or `describe.serial()` for tests that share state.

**Q5: How do you run Playwright tests across multiple machines to reduce CI time?**
A: Use sharding: `npx playwright test --shard=1/4` runs the first quarter of tests. Run 4 instances in parallel (in CI matrix), each with a different shard index. After all shards finish, use `npx playwright merge-reports` to combine results into one HTML report.

**Q6: When should you use `route()` vs WireMock for API mocking?**
A: Use `route()` for fast, simple per-test mocking — great for one-off scenarios and response simulation. Use WireMock when you need persistent mocks, stateful responses, stub management via API, or when multiple services all need mocking and you want a central mock server.

**Q7: What is storageState and how is it different from cookies?**
A: `storageState` saves the entire browser session: cookies, localStorage, and sessionStorage. It is more complete than just saving cookies. You can create a storageState file per user role and reload it to get an authenticated context instantly.

**Q8: How do you test a file download in Playwright?**
A: Use `page.waitForEvent('download')` triggered alongside the action that starts the download. Then assert the filename and optionally read the file content from the disk path.

**Q9: What is a worker-scoped fixture and when do you use it?**
A: A worker-scoped fixture is set up once per worker process and shared across all tests in that worker. Use it for expensive setup that is safe to share — like a database connection pool. Test-scoped fixtures are set up and torn down for each test.

**Q10: How do you handle a test that is inherently flaky due to external dependencies?**
A: First, mock the dependency with `route()` to remove external flakiness. If mocking is not possible: set `retries: 2` in config, add `test.slow()` for the specific test, and tag it `@flaky` for monitoring. Track the flakiness rate over time and prioritise fixing it.

---

### 11.2 Staff SDET Questions (8–12 YOE)

**Q1: How would you design a test framework for a monorepo with 10 microservices?**
A: Shared `packages/test-base` library with common fixtures, page objects base classes, and service clients. Each service has its own `tests/` folder. Single `playwright.config.ts` at root with projects per service. Shared GitHub Actions reusable workflow for running any service's tests. Central test data management with builders per service. Shared reporting setup with Allure.

**Q2: How do you measure and improve test suite reliability (flakiness)?**
A: Track per-test pass/fail rate over last N runs. Flag any test with failure rate > 5% as flaky. Root cause categories: timing issues (fix with better locators/waits), data isolation issues (fix with per-test data builders + cleanup), environment issues (fix with health checks + retry on infra failure). Report flakiness rate as a team metric alongside test coverage.

**Q3: A developer says "your tests are slowing down our CI pipeline." How do you respond?**
A: Audit the pipeline first. Identify what is slow: are tests themselves slow (optimize with sharding), or is setup slow (optimize with globalSetup/storageState), or is CI infrastructure slow (upgrade to faster runners). Present data: current time vs projected time after optimizations. Implement sharding, parallel execution, and remove redundant tests. Target: full E2E suite under 10 minutes.

**Q4: How do you handle test data management across environments?**
A: Three strategies: (1) Dynamic creation — create data via API in beforeAll, clean up in afterAll, use timestamps in emails/names for uniqueness. (2) Seed scripts — run once per environment, mark data as `test_data: true` for easy cleanup. (3) Snapshot — take DB snapshots after seeding, restore before test runs. Choose based on data complexity and test speed requirements.

**Q5: How do you decide between E2E tests and component/integration tests?**
A: Use the test trophy / pyramid. Unit tests: fast, cheap, cover business logic in isolation. Integration tests: cover service boundaries, contract validation. E2E tests: cover critical user journeys only — login, checkout, key workflows. E2E for confidence, not coverage. If a unit test can cover it, it should. E2E should be expensive-to-fail, not expensive-to-run.

---

### 11.3 Principal SDET / Test Architect Questions (12+ YOE)

**Q1: How do you build a quality engineering culture in an organization where testing is an afterthought?**
A: Start with data — show defect escape rates, MTTR, production incidents caused by missing tests. Pick one high-visibility team and partner closely: embed with them, deliver fast results, make them advocates. Build shared tooling that makes testing easy, not a burden. Create a testing guild — cross-team community of practice. Measure and celebrate improvements. Gradually shift from testing-as-gating to quality-as-shared-responsibility.

**Q2: Build vs buy: when do you build a custom test framework vs adopt an existing one?**
A: Almost always adopt. Playwright, Jest, k6 are maintained by large teams with dedicated resources. Only build custom tooling when: (1) a specific gap exists that no tool fills, (2) the gap is business-critical, (3) you have the team to maintain it long-term. Custom reporters, fixtures, and helpers: always build. Custom assertion libraries or test runners: rarely justified.

**Q3: How do you approach testability review in system design?**
A: Get SDETs into design review as early as requirements. Ask: how will we verify this behaviour? Can we trigger edge cases in lower environments? Are there observable outputs for every testable behaviour? Flag designs that make testing hard: tight coupling, no environment override points, no test data APIs, no event hooks. Write testability requirements into the design doc as non-negotiables.

**Q4: How do you scale a test infrastructure from 1 team to 50 teams?**
A: Golden path: one recommended stack with documented standards. Shared platform team owns the tooling, frameworks, and CI templates. Each product team owns their own tests. Inner-source model: anyone can contribute to shared libraries. Platform abstracts away infra complexity — teams focus on test logic, not Docker configs. Metrics dashboard showing test health per team.

**Q5: A release is blocked by a failing E2E test. The business wants to override it. What do you do?**
A: Treat it like a production incident. Triage immediately: is this a real regression or a test environment issue? If real regression: block the release, fix the code, this is exactly what the gate is for. If test environment issue: document it, temporarily skip the test with a Jira ticket attached, release, and fix the test within 24 hours as P1. Never permanently remove a test because it is inconvenient. The gate exists for a reason.

---

### 11.4 System Design Interview Question

**Question:** "Design a test infrastructure for a payment processing platform that handles 100,000 transactions per day. The system has a React frontend, 8 microservices, a PostgreSQL database, and integrates with 3 external payment providers. How would you structure the testing strategy and infrastructure?"

**Clarifying questions to ask:**
- What is the deployment frequency? (daily? per commit?)
- What is the current test coverage and suite size?
- What is the acceptable CI pipeline time?
- Are the external payment providers mockable in non-prod environments?
- What are the compliance requirements? (PCI DSS, SOC2)

**Model Answer:**

```
Testing Layers:
1. Unit tests (Jest) — service-level, 80% coverage minimum, run in <2 min
2. Contract tests (Pact) — verify all 8 service contracts, run on every PR
3. API integration tests (Playwright APIRequestContext) — per service, mocked payment providers via WireMock
4. E2E tests (Playwright) — 20 critical user journeys only, sharded across 4 agents, under 8 min
5. Performance tests (k6) — smoke test on every deploy, full load test weekly
6. Security tests (ZAP) — OWASP scan on every release candidate

Infrastructure:
- GitHub Actions matrix for sharding
- Docker Compose for local dev (app + db + wiremock)
- ECS Fargate for CI test execution (scales on demand, no idle cost)
- S3 for test artifacts and reports
- Allure for test reporting
- CloudWatch for test metrics and alerting

Payment Provider Mocking:
- WireMock for Stripe/PayPal/Adyen in CI
- Dedicated sandbox accounts for staging
- Never use real transactions in tests

Compliance:
- PII scrubbing in screenshots and logs
- RBAC test matrix for PCI scope
- Audit trail tests for transaction logging
```

---

### 11.5 Quick Reference Cheat Sheet

```
TOP 10 THINGS TO REMEMBER:

1.  Always use getByRole / getByLabel — never CSS or XPath as first choice
2.  storageState = save login once, reuse everywhere — never log in per test
3.  Fixtures are the backbone — put all setup/teardown there, not in tests
4.  fullyParallel: true + 4 workers = 4x faster suite with zero code changes
5.  Shard across machines in CI — --shard=1/4, --shard=2/4, merge-reports
6.  route() is your best friend — mock any API call in 5 lines
7.  trace: 'retain-on-failure' — always enable, it saves hours of debugging
8.  ROLLBACK in DB fixtures — zero test data left behind, every time
9.  retries: 2 in CI only — never in local, it hides real bugs locally
10. Never assert inside Page Objects — actions only, assert in tests

NEVER DO THIS:
- page.waitForTimeout(3000)           — use expect().toBeVisible() instead
- page.click('.some-css-class')       — use getByRole() instead
- Assertions inside Page Object methods
- Shared browser context between tests
- Skipping test cleanup to "save time"
- Using test.only in committed code
- Real credentials in source code
- Storing .auth/*.json files in git
```

---

## SECTION 12 — 30-DAY SPRINT PLAN

```
WEEK 1 — Foundation
Day 01: Install Playwright, run first test, understand project structure
Day 02: Locator strategies — getByRole, getByLabel, getByTestId — write 10 locators
Day 03: Assertions — all web-first assertions, soft assertions, custom matchers
Day 04: playwright.config.ts — multi-browser, retry, reporters, webServer
Day 05: Page Object Model — build LoginPage, DashboardPage from scratch
Day 06: Fixtures — custom test.extend(), fixture scope, composition
Day 07: Review + debug with Trace Viewer and PWDEBUG

WEEK 2 — Implementation
Day 08: API testing — APIRequestContext, GET/POST/PUT/DELETE with assertions
Day 09: Network interception — route(), fulfill(), continue(), abort()
Day 10: Auth flows — storageState, globalSetup, role-based fixtures
Day 11: Data-driven testing — CSV with papaparse, Excel with xlsx
Day 12: RBAC testing — permission matrix, fixture-per-role
Day 13: Database integration — pg fixture with transaction rollback
Day 14: Hands-on project — build the payment suite from Section 4.2

WEEK 3 — Advanced Patterns
Day 15: Design patterns — Factory, Builder, Strategy in TypeScript
Day 16: SOLID principles — refactor week 2 code to follow SOLID
Day 17: Parallelism — workers, fullyParallel, serial groups
Day 18: Sharding — --shard, merge-reports, GitHub Actions matrix
Day 19: Custom reporter — implement Slack-notifying reporter
Day 20: Docker — containerise tests, docker-compose full environment
Day 21: GitHub Actions — full CI pipeline with matrix, artifacts, notifications

WEEK 4 — Mastery + Interview Prep
Day 22: k6 smoke test — write and run post-deploy performance check
Day 23: Security tests — auth bypass, RBAC, injection tests
Day 24: AWS/Azure — deploy test runner to ECS Fargate or ACI
Day 25: Playwright MCP — set up MCP server, test with Claude Desktop
Day 26: AI test generation — use the prompt template from Section 8.3
Day 27: Interview prep — answer all Senior SDET questions out loud
Day 28: Interview prep — answer all Staff SDET questions, draw architecture
Day 29: Interview prep — system design mock — payment platform question
Day 30: Capstone — run full suite, measure time, write ADRs, review everything
```

---

*This prompt output covers all 12 sections. Use each section independently or as a full deep-dive. Replace the topic in the master prompt and repeat this structure for every subject on your roadmap.*
