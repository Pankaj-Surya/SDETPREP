# Debugging Techniques for Test Automation

## The One Sentence to Anchor Everything

Before any technique, remember this: **"Debugging is the process of asking better questions, not guessing randomly."**

Every technique below is just a structured way to ask a better question. When a test fails, a junior engineer asks "why is this broken?" A senior engineer asks "what changed, where did the state diverge from my expectation, and what is the smallest piece of evidence that proves it?" That shift from vague to specific is what debugging skill actually is.

---

## How to Answer This in an Interview

Structure your answer in four beats:

**Beat 1** — State your mindset. Debugging is systematic, not random.
**Beat 2** — Explain how you read failure information before touching the code.
**Beat 3** — Walk through your isolation strategy — shrink the problem space.
**Beat 4** — Explain how you prove the fix, not just hope the fix worked.

This structure works for any debugging question, not just test automation. The techniques below map onto this structure.

---

## Technique 1 — Read the Failure Before Touching Anything

### What
Before you change a single line of code, extract maximum information from what already exists: the error message, the stack trace, the screenshot, the network log, the trace file.

### Why
Most debugging time is wasted by people who start changing code before understanding the failure. They end up in a cycle of change, rerun, change, rerun, with no mental model of what is actually wrong. Reading everything that the failure gives you first cuts that cycle short.

### When
Every single time a test fails. No exceptions. Even if you think you know what's wrong — read the failure first, because assumptions kill debugging time.

### How

```typescript
// playwright typescript
// When a test fails, Playwright gives you several artifacts.
// Know how to read all of them before you touch code.

// 1. Read the error message — it tells you WHAT failed
// Example: "Error: expect(received).toHaveText(expected)
//           Expected: 'Payment Successful'
//           Received: 'Payment Pending'"
// This tells you the page showed "Payment Pending" — a state problem, not a selector problem.
// Wrong fix: changing the selector
// Right fix: investigating why payment is pending, not completed

// 2. Read the stack trace — it tells you WHERE it failed
// Example:
// at LoginPage.login (pages/login.page.ts:23)
// at test (tests/checkout.spec.ts:15)
// This tells you the failure is in login, called from checkout test.
// The checkout test has nothing to do with payments — login is broken.

// 3. Enable trace on failure — it shows you the full timeline
// In playwright.config.ts:
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    // Records trace only on failing tests — no overhead on passing ones
    trace: 'retain-on-failure',
    // Screenshot on failure — see exactly what the browser showed
    screenshot: 'only-on-failure',
    // Video on failure — see what happened before the failure
    video: 'retain-on-failure',
  },
});

// After a failure, open the trace:
// npx playwright show-trace test-results/trace.zip
// This shows: every action, every assertion, every network call, timeline
// You can see EXACTLY what the page looked like at each step
```

```java
// selenium java
// Java gives you less automatically — you have to set up artifact capture yourself.
// But the principle is the same: read everything before touching code.

import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.OutputType;
import org.testng.ITestResult;
import org.testng.annotations.AfterMethod;
import java.io.File;
import java.nio.file.Files;
import java.nio.file.Paths;

public class BaseTest {

  @AfterMethod
  public void captureOnFailure(ITestResult result) {
    // Only capture when the test fails — not on every test
    if (result.getStatus() == ITestResult.FAILURE) {

      // 1. Screenshot — what did the browser show when it failed
      try {
        File screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
        String path = "test-results/" + result.getName() + "-failure.png";
        Files.copy(screenshot.toPath(), Paths.get(path));
        System.out.println("Screenshot saved: " + path);
      } catch (Exception e) {
        System.err.println("Screenshot capture failed: " + e.getMessage());
      }

      // 2. Page source — what was actually in the DOM
      try {
        String pageSource = driver.getPageSource();
        Files.writeString(
          Paths.get("test-results/" + result.getName() + "-page-source.html"),
          pageSource
        );
      } catch (Exception e) {
        System.err.println("Page source capture failed: " + e.getMessage());
      }

      // 3. Current URL — where was the browser when it failed
      System.out.println("URL at failure: " + driver.getCurrentUrl());

      // 4. Stack trace — already printed by TestNG, but log the test method too
      System.out.println("Failed test: " + result.getName());
      System.out.println("Failure message: " + result.getThrowable().getMessage());
    }
  }
}
```

---

## Technique 2 — Reproduce the Failure in Isolation

### What
Run only the failing test, in isolation, with no other tests before or after it. This rules out one of the most common causes of test failure: test ordering dependency, where test B only fails because test A left behind some state.

### Why
Test suites that run in parallel or sequence often have hidden dependencies. Test A creates a user, test B accidentally uses that same user. Test A deletes something, test B expects it to exist. These bugs only appear when tests run together. Running in isolation immediately tells you whether the failure is real or an artifact of ordering.

### When
The moment a test fails in CI but you cannot reproduce it locally, or when a test fails intermittently but not consistently.

### How

```typescript
// playwright typescript

// Run one specific test file
// npx playwright test tests/payment.spec.ts

// Run one specific test by name
// npx playwright test -g "payment is completed after card submission"

// Run with headed browser so you can watch what happens
// npx playwright test tests/payment.spec.ts --headed

// Run with debug mode — steps through each action one by one
// PWDEBUG=1 npx playwright test tests/payment.spec.ts

// Run with UI mode — interactive, shows test timeline, can re-run single tests
// npx playwright test --ui

// If it passes in isolation but fails with other tests:
// you have a test ordering dependency — find what shared state is being polluted

// Check for shared state in your fixtures
// BAD: storing state at module level (shared across tests)
let sharedUserId: string;  // this leaks between tests — DO NOT DO THIS

// GOOD: state scoped to each test
test('payment test', async ({ request }) => {
  const user = await createUser(request);  // created fresh for this test
  // test logic
  await deleteUser(request, user.id);      // cleaned up after this test
});
```

```java
// selenium java

// Run one specific test class
// mvn test -Dtest=PaymentTest

// Run one specific test method
// mvn test -Dtest=PaymentTest#paymentIsCompletedAfterCardSubmission

// Run in debug mode — attach your IDE debugger
// mvn test -Dtest=PaymentTest -Dmaven.surefire.debug

// If it fails with the full suite but passes alone:
// Look for shared static state — the most common Java culprit
public class BadExample {
  // PROBLEM: static field shared across ALL tests in ALL threads
  static WebDriver driver;
  static String userId;

  @Test
  public void test1() {
    userId = createUser();  // test1 sets userId
  }

  @Test
  public void test2() {
    deleteUser(userId);  // test2 uses it — breaks if test1 ran first, different order
  }
}

// GOOD: instance fields, each test class gets fresh state
public class GoodExample {
  WebDriver driver;  // not static — each test instance has its own

  @BeforeMethod
  public void setUp() {
    driver = new ChromeDriver();  // fresh browser per test
  }

  @AfterMethod
  public void tearDown() {
    if (driver != null) driver.quit();
  }
}
```

---

## Technique 3 — Use Logging to Create a Breadcrumb Trail

### What
Add structured log statements at key decision points so when a test fails, you have a timeline of what happened leading up to the failure — not just the failure itself.

### Why
A failure message tells you what broke. Logs tell you what the test was doing for the 30 seconds before it broke. Without logs, you are looking at a crime scene with no witnesses. With logs, you have a timeline. The difference between debugging with logs and without is roughly the difference between reading a map and wandering randomly.

### When
Add logs at the start of each major action, when you receive API responses, when you hit conditional logic, and just before any assertion. Not every single line — just the decision points.

### How

```typescript
// playwright typescript

// Create a simple structured logger — one file, used everywhere
// utils/logger.ts

class TestLogger {
  private testName: string;

  constructor(testName: string) {
    this.testName = testName;
  }

  // Logs with timestamp and test name — easy to grep in CI output
  step(description: string): void {
    console.log(`[${new Date().toISOString()}] [${this.testName}] STEP: ${description}`);
  }

  data(label: string, value: unknown): void {
    console.log(`[${this.testName}] DATA: ${label} =`, JSON.stringify(value, null, 2));
  }

  warn(message: string): void {
    console.warn(`[${this.testName}] WARN: ${message}`);
  }

  error(message: string, err?: unknown): void {
    console.error(`[${this.testName}] ERROR: ${message}`, err ?? '');
  }
}

// Usage in a test — real example
import { test, expect } from '@playwright/test';

test('payment flow completes end to end', async ({ page, request }) => {
  const log = new TestLogger('payment-flow-e2e');

  log.step('Creating test user via API');
  const userResponse = await request.post('/api/users', {
    data: { email: `pay+${Date.now()}@test.com`, role: 'user' },
  });
  const user = await userResponse.json() as { id: string; email: string };
  log.data('Created user', { id: user.id, email: user.email });

  log.step('Navigating to checkout page');
  await page.goto('/checkout');

  log.step('Filling payment details');
  await page.getByLabel('Card Number').fill('4111111111111111');
  await page.getByLabel('Amount').fill('100');

  log.step('Submitting payment');
  await page.getByRole('button', { name: 'Pay Now' }).click();

  log.step('Asserting confirmation screen');
  // If this fails, you know from logs that all previous steps completed
  await expect(page.getByText('Payment Successful')).toBeVisible();
  log.data('Final URL', page.url());
});

// When this test fails, CI output shows:
// [2024-01-15T10:23:01.123Z] [payment-flow-e2e] STEP: Creating test user via API
// [payment-flow-e2e] DATA: Created user = {"id": "usr_123", "email": "pay+..."}
// [2024-01-15T10:23:02.456Z] [payment-flow-e2e] STEP: Navigating to checkout page
// [2024-01-15T10:23:03.789Z] [payment-flow-e2e] STEP: Filling payment details
// [2024-01-15T10:23:04.012Z] [payment-flow-e2e] STEP: Submitting payment
// [2024-01-15T10:23:04.567Z] [payment-flow-e2e] STEP: Asserting confirmation screen
// ERROR: expect received "Payment Pending" to have text "Payment Successful"
//
// From this, you immediately know: all steps before assertion succeeded,
// the payment was submitted but came back as "Pending", not "Successful"
// Problem is in the backend payment processing, not in the test itself
```

```java
// selenium java

public class TestLogger {
  private final String testName;

  public TestLogger(String testName) {
    this.testName = testName;
  }

  public void step(String description) {
    System.out.printf("[%s] [%s] STEP: %s%n",
      java.time.Instant.now(), testName, description);
  }

  public void data(String label, Object value) {
    System.out.printf("[%s] DATA: %s = %s%n", testName, label, value);
  }

  public void warn(String message) {
    System.out.printf("[%s] WARN: %s%n", testName, message);
  }

  public void error(String message, Exception e) {
    System.err.printf("[%s] ERROR: %s — %s%n", testName, message,
      e != null ? e.getMessage() : "");
  }
}

// Usage in a test
public class PaymentTest extends BaseTest {

  @Test
  public void paymentFlowCompletesEndToEnd() {
    TestLogger log = new TestLogger("payment-flow-e2e");

    log.step("Creating test user via API");
    String userId = userService.createUser("pay+" + System.currentTimeMillis() + "@test.com");
    log.data("Created user ID", userId);

    log.step("Navigating to checkout page");
    driver.get(BASE_URL + "/checkout");

    log.step("Filling payment details");
    driver.findElement(By.id("cardNumber")).sendKeys("4111111111111111");
    driver.findElement(By.id("amount")).sendKeys("100");

    log.step("Submitting payment");
    driver.findElement(By.id("payButton")).click();

    log.step("Asserting confirmation screen");
    // If this fails, logs show exactly what happened before it
    String confirmationText = wait
      .until(ExpectedConditions.visibilityOfElementLocated(By.id("confirmation")))
      .getText();
    log.data("Confirmation text", confirmationText);
    Assert.assertEquals(confirmationText, "Payment Successful");
  }
}
```

---

## Technique 4 — Bisect to Find Where the Test Diverges From Reality

### What
When a test fails and you don't know why, add an assertion or log at the halfway point of the test. If that passes, the problem is in the second half. If it fails, the problem is in the first half. Repeat, narrowing by half each time. This is binary search applied to debugging.

### Why
Long tests have many steps. Without bisecting, you read every line looking for the bug. With bisecting, a test with 20 steps takes at most 5 iterations to locate the exact failing step. This is not just faster — it forces you to stop guessing and start eliminating, which is the core mental model of all expert debugging.

### When
When a test has more than 5–6 steps and you can't immediately identify which one is wrong from the error message. Also when the failure is subtle — "the page looks correct but the data is wrong somewhere."

### How

```typescript
// playwright typescript

// Scenario: test fails at step 8 of 10, but you don't know why
// Step 1: add a checkpoint at step 5 to see if state is correct there

test('complete user registration flow', async ({ page, request }) => {

  // Steps 1-2: setup
  const user = await createTestUser(request);
  await page.goto('/register');

  // Steps 3-4: fill registration form
  await page.getByLabel('First Name').fill('John');
  await page.getByLabel('Last Name').fill('Doe');
  await page.getByLabel('Email').fill(user.email);
  await page.getByLabel('Password').fill('Password123!');

  // BISECT CHECKPOINT — add this temporarily to find the problem
  // If this passes, the bug is in step 6-10. If this fails, it is in 1-5.
  const emailField = await page.getByLabel('Email').inputValue();
  console.log('BISECT: email field value after fill =', emailField);
  // Did the fill actually work? Is email what we expect here?

  // Steps 5-7: submit and verify email
  await page.getByRole('button', { name: 'Create Account' }).click();

  // SECOND BISECT CHECKPOINT
  console.log('BISECT: URL after submit =', page.url());
  // Did we land on the right page? Or did a redirect not happen?

  await page.goto('/email-verification');

  // Steps 8-10: verify email
  await page.getByRole('button', { name: 'Verify Email' }).click();

  // Final assertion — this is where the test was failing
  await expect(page.getByText('Email Verified')).toBeVisible();

  // After you find the bug: remove all BISECT console.logs
  // They were temporary diagnostic tools, not permanent logging
});
```

```java
// selenium java

@Test
public void completeUserRegistrationFlow() {

  // Steps 1-2: setup
  String email = "reg+" + System.currentTimeMillis() + "@test.com";
  driver.get(BASE_URL + "/register");

  // Steps 3-4: fill form
  driver.findElement(By.id("firstName")).sendKeys("John");
  driver.findElement(By.id("lastName")).sendKeys("Doe");
  driver.findElement(By.id("email")).sendKeys(email);
  driver.findElement(By.id("password")).sendKeys("Password123!");

  // BISECT CHECKPOINT 1 — verify state at the halfway point
  String emailValue = driver.findElement(By.id("email")).getAttribute("value");
  System.out.println("BISECT: email field value = " + emailValue);
  Assert.assertEquals(emailValue, email, "BISECT: email field has wrong value at step 4");

  // Steps 5-6: submit
  driver.findElement(By.id("createAccount")).click();

  // BISECT CHECKPOINT 2 — verify we landed on the right page
  System.out.println("BISECT: URL after submit = " + driver.getCurrentUrl());
  Assert.assertTrue(
    driver.getCurrentUrl().contains("/verify"),
    "BISECT: redirect to verification page did not happen"
  );

  // Steps 7-9: verify email
  driver.findElement(By.id("verifyEmail")).click();

  // Final assertion
  String confirmation = wait
    .until(ExpectedConditions.visibilityOfElementLocated(By.id("confirmation")))
    .getText();
  Assert.assertEquals(confirmation, "Email Verified");
}
```

---

## Technique 5 — Inspect Network Calls to Separate UI Bugs From API Bugs

### What
Capture what API calls the UI is actually making during a test. When a test fails, check whether the API call was made with the right payload and what the API returned. This immediately tells you whether the bug is in the frontend or the backend.

### Why
This is one of the highest-value debugging techniques for SDET roles because it separates two fundamentally different problems. If the UI sent the right request and the API returned an error, that's a backend bug. If the UI sent the wrong request or didn't send any request, that's a frontend bug. Without network inspection, you spend time debugging the wrong layer.

### When
Whenever a test fails on an assertion that depends on data from the backend — payment confirmation, user creation, search results, anything that involves an API call behind the scenes.

### How

```typescript
// playwright typescript
// Playwright makes network inspection easy — built into the API

import { test, expect } from '@playwright/test';

test('payment submission sends correct API call', async ({ page }) => {

  // Capture all requests and responses before the action happens
  const apiCalls: { url: string; method: string; status: number; requestBody: string; responseBody: string }[] = [];

  // Listen for every response
  page.on('response', async (response) => {
    // Only capture API calls, not static assets
    if (response.url().includes('/api/')) {
      let responseBody = '';
      let requestBody = '';
      try {
        responseBody = await response.text();
        requestBody  = response.request().postData() ?? '';
      } catch {
        // response body might already be consumed
      }
      apiCalls.push({
        url:          response.url(),
        method:       response.request().method(),
        status:       response.status(),
        requestBody,
        responseBody,
      });
    }
  });

  await page.goto('/checkout');
  await page.getByLabel('Card Number').fill('4111111111111111');
  await page.getByLabel('Amount').fill('100');
  await page.getByRole('button', { name: 'Pay Now' }).click();

  // Test fails here — now check what the API actually did
  await expect(page.getByText('Payment Successful')).toBeVisible();

  // If the test fails, inspect the captured calls:
  console.log('API calls made during test:');
  apiCalls.forEach(call => {
    console.log(`${call.method} ${call.url} → ${call.status}`);
    if (call.requestBody)  console.log('  Request:', call.requestBody);
    if (call.responseBody) console.log('  Response:', call.responseBody);
  });

  // From this output you can see:
  // If no /api/payment call was made → UI bug, button didn't submit
  // If call was made with wrong amount → UI bug, wrong value sent
  // If call returned 500 → API bug, backend crashed
  // If call returned {"status": "pending"} → logic bug, payment not completing
});
```

```typescript
// playwright typescript — mock API to isolate frontend bugs

test('payment form shows error when API returns 500', async ({ page }) => {

  // Intercept and mock the payment API call
  await page.route('**/api/payments', async (route) => {
    const request = route.request();

    // Log what the frontend actually sent
    console.log('Payment request body:', request.postData());

    // Return a 500 response to test error handling
    await route.fulfill({
      status: 500,
      contentType: 'application/json',
      body: JSON.stringify({ error: 'Internal server error' }),
    });
  });

  await page.goto('/checkout');
  await page.getByLabel('Card Number').fill('4111111111111111');
  await page.getByRole('button', { name: 'Pay Now' }).click();

  // Now you can test the error state in isolation
  // without needing a broken backend
  await expect(page.getByRole('alert')).toContainText('Payment failed');
});
```

```java
// selenium java — use BrowserMob Proxy to capture network calls

import net.lightbody.bmp.BrowserMobProxy;
import net.lightbody.bmp.BrowserMobProxyServer;
import net.lightbody.bmp.client.ClientUtil;
import net.lightbody.bmp.core.har.Har;
import net.lightbody.bmp.core.har.HarEntry;
import org.openqa.selenium.Proxy;
import org.openqa.selenium.chrome.ChromeOptions;

public class NetworkInspectionTest {

  private BrowserMobProxy proxy;
  private WebDriver driver;

  @BeforeMethod
  public void setUp() {
    // Start the proxy
    proxy = new BrowserMobProxyServer();
    proxy.start(0);

    // Tell Chrome to route traffic through our proxy
    Proxy seleniumProxy = ClientUtil.createSeleniumProxy(proxy);
    ChromeOptions options = new ChromeOptions();
    options.setProxy(seleniumProxy);
    options.addArguments("--ignore-certificate-errors");

    driver = new ChromeDriver(options);

    // Start capturing HAR (HTTP Archive)
    proxy.newHar("payment-test");
  }

  @Test
  public void paymentSubmitsSentWithCorrectAmount() {
    driver.get(BASE_URL + "/checkout");
    driver.findElement(By.id("cardNumber")).sendKeys("4111111111111111");
    driver.findElement(By.id("amount")).sendKeys("100");
    driver.findElement(By.id("payButton")).click();

    // Wait for assertion
    String confirmation = wait
      .until(ExpectedConditions.visibilityOfElementLocated(By.id("confirmation")))
      .getText();

    // Get all network calls made during the test
    Har har = proxy.getHar();
    for (HarEntry entry : har.getLog().getEntries()) {
      String url = entry.getRequest().getUrl();
      if (url.contains("/api/payment")) {
        int status = entry.getResponse().getStatus();
        System.out.println("Payment API call: " + url + " → " + status);

        // Check the request body
        String requestBody = entry.getRequest().getPostData() != null
          ? entry.getRequest().getPostData().getText()
          : "no body";
        System.out.println("Request body: " + requestBody);

        // Check the response
        String responseBody = entry.getResponse().getContent().getText();
        System.out.println("Response body: " + responseBody);

        // Assert the API call itself was correct
        Assert.assertEquals(status, 200, "Payment API returned non-200 status");
      }
    }

    Assert.assertEquals(confirmation, "Payment Successful");
  }

  @AfterMethod
  public void tearDown() {
    if (proxy != null) proxy.stop();
    if (driver != null) driver.quit();
  }
}
```

---

## Technique 6 — Use the Playwright Trace Viewer for Time-Travel Debugging

### What
Playwright records every action, every DOM state, every network call, every console log into a trace file. When a test fails, you open that trace file and walk backward from the failure to see exactly what the state of the page was at every step. This is "time-travel debugging."

### Why
This is one of the biggest advantages Playwright has over other tools. Instead of adding console logs and rerunning, you already have a complete recording of what happened. The trace file lets you see the DOM snapshot at each step — you can literally look at what the page showed before the click, after the click, during the wait. This cuts investigation time dramatically.

### When
Whenever a test fails in CI and you cannot reproduce it locally. The trace file from CI contains everything you need.

### How

```typescript
// playwright typescript

// playwright.config.ts — enable tracing
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    trace: 'retain-on-failure',  // only records when test fails — no overhead otherwise
  },
});

// To view the trace after a failure:
// npx playwright show-trace test-results/my-test/trace.zip

// What you will see in the trace viewer:
// Left panel: timeline of every action with timestamp
// Center panel: DOM snapshot at the selected action
// Right panel: network requests + console logs at that moment

// You can click on "page.getByRole('button').click()" and see
// exactly what the page looked like BEFORE that click
// This immediately shows you if the element was there, visible, enabled

// For CI: upload the trace as an artifact in GitHub Actions
```

```yaml
# .github/workflows/playwright.yml
# Always upload traces so you can debug CI failures locally

- name: Run tests
  run: npx playwright test

- name: Upload traces on failure
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: playwright-traces
    path: test-results/
    retention-days: 7
# Download the artifact, run: npx playwright show-trace trace.zip
# and you have full time-travel debugging of the CI failure
```

---

## Technique 7 — Check the State Before Asserting, Not Just After

### What
When an assertion fails, the natural instinct is to look at the assertion. But the real question is: what state led to this assertion failing? Add a log or a query to check the actual state of the system just before the assertion, so you know whether the state is wrong or the assertion is wrong.

### Why
Two different bugs produce the same assertion failure. Bug one: the state never got set correctly — the payment was never processed. Bug two: the state was set correctly but you are asserting against the wrong thing — you're checking "Payment Successful" but the text is actually "Payment Completed." These two bugs require completely different fixes. Checking state before asserting tells you which one you have.

### When
Whenever an assertion fails and the failure message alone doesn't tell you why the expected state was never reached.

### How

```typescript
// playwright typescript

test('order status shows as shipped', async ({ page, request }) => {

  const orderId = 'ORD-123';
  await page.goto(`/orders/${orderId}`);

  // BAD: only assertion, no state check
  // await expect(page.getByTestId('order-status')).toHaveText('Shipped');
  // If this fails, you only know: text was not "Shipped". You don't know what it was.

  // GOOD: check the actual state first, then assert
  const statusElement = page.getByTestId('order-status');
  const actualStatus  = await statusElement.textContent();

  // Log actual state before asserting — tells you what the page actually showed
  console.log(`Order ${orderId} status on page: "${actualStatus}"`);

  // Check via API what the backend thinks the status is
  const apiResponse = await request.get(`/api/orders/${orderId}`);
  const apiData     = await apiResponse.json() as { status: string };
  console.log(`Order ${orderId} status from API: "${apiData.status}"`);

  // Now you have two pieces of information:
  // If page shows "Processing" and API says "Processing" → backend hasn't updated yet, timing issue
  // If page shows "Processing" and API says "Shipped" → UI rendering bug
  // If page shows "Shipped" but assertion fails → wrong selector or text mismatch

  await expect(statusElement).toHaveText('Shipped');
});
```

```java
// selenium java

@Test
public void orderStatusShowsAsShipped() {
  String orderId = "ORD-123";
  driver.get(BASE_URL + "/orders/" + orderId);

  // Check actual state from the page before asserting
  WebElement statusElement = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("orderStatus"))
  );
  String actualPageStatus = statusElement.getText();
  System.out.println("Page shows order status: " + actualPageStatus);

  // Cross-check with API
  Response apiResponse = RestAssured.given()
    .baseUri(System.getenv("API_URL"))
    .when().get("/api/orders/" + orderId);
  String apiStatus = apiResponse.jsonPath().getString("status");
  System.out.println("API says order status: " + apiStatus);

  // Now you have enough information to diagnose the bug before asserting
  Assert.assertEquals(actualPageStatus, "Shipped",
    "Expected Shipped but got: " + actualPageStatus +
    " (API says: " + apiStatus + ")");
}
```

---

## Technique 8 — Flaky Test Debugging — Find the Non-Determinism

### What
A flaky test is one that passes sometimes and fails sometimes with no code change. The debugging goal is to find the source of non-determinism — the thing that is different between a passing run and a failing run.

### Why
Flaky tests are the most trust-destroying thing in a test suite. Once engineers start ignoring failures because "it's probably just flaky," you have lost the value of the entire suite. The fix is not to retry or suppress — it is to find and eliminate the non-determinism.

### When
When a test fails in CI but passes locally, or passes in the morning but fails in the afternoon, or fails one in five runs with no obvious pattern.

### How

```typescript
// playwright typescript

// Strategy 1: Run the test 10 times in a row to reproduce the flake locally
// npx playwright test tests/payment.spec.ts --repeat-each=10
// If it fails even once, you've reproduced it and can investigate

// Strategy 2: Add timestamps to logs to find timing issues
test('dashboard loads after login', async ({ page }) => {
  const t0 = Date.now();

  await page.goto('/login');
  console.log(`goto /login: ${Date.now() - t0}ms`);

  await page.getByLabel('Email').fill('user@test.com');
  await page.getByLabel('Password').fill('pass');
  await page.getByRole('button', { name: 'Sign in' }).click();
  console.log(`login click: ${Date.now() - t0}ms`);

  // Is the flake here? Is this wait inconsistent?
  await page.waitForURL(/dashboard/, { timeout: 10000 });
  console.log(`reached dashboard: ${Date.now() - t0}ms`);

  // If you see sometimes it takes 2000ms and sometimes 8000ms
  // you know there's a backend timing variance here
  await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
  console.log(`heading visible: ${Date.now() - t0}ms`);
});

// Strategy 3: Check for hardcoded waits — the most common cause of flakes
// BAD: assumes 2 seconds is always enough
await page.waitForTimeout(2000);  // this is the most common cause of flakiness

// GOOD: wait for the actual condition, not a time duration
await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();

// Strategy 4: Check for shared state between parallel tests
// If two tests both create a user with email "test@test.com"
// one of them will fail because the email already exists

// BAD: hardcoded test data shared across parallel tests
const user = { email: 'test@test.com' };  // two parallel tests will conflict

// GOOD: unique test data per test run
const user = { email: `test+${Date.now()}+${Math.random()}@test.com` };
```

```java
// selenium java

// Strategy 1: Run test multiple times
// mvn test -Dtest=PaymentTest#paymentSubmits -Dsurefire.rerunFailingTestsCount=5

// Strategy 2: Log timing to find where the inconsistency is
@Test
public void dashboardLoadsAfterLogin() {
  long start = System.currentTimeMillis();

  driver.get(BASE_URL + "/login");
  System.out.println("goto login: " + (System.currentTimeMillis() - start) + "ms");

  driver.findElement(By.id("email")).sendKeys("user@test.com");
  driver.findElement(By.id("password")).sendKeys("pass");
  driver.findElement(By.id("loginBtn")).click();
  System.out.println("login click: " + (System.currentTimeMillis() - start) + "ms");

  // How long does this actually take?
  wait.until(ExpectedConditions.urlContains("/dashboard"));
  System.out.println("reached dashboard: " + (System.currentTimeMillis() - start) + "ms");

  // If it's 1500ms some times and 7000ms other times
  // the backend has a variable response time — that's your flake source
  WebElement heading = wait
    .until(ExpectedConditions.visibilityOfElementLocated(By.tagName("h1")));
  System.out.println("heading visible: " + (System.currentTimeMillis() - start) + "ms");

  Assert.assertEquals(heading.getText(), "Welcome");
}

// Strategy 3: Find hardcoded sleeps — remove every single one
// Thread.sleep(2000);  // this causes flakes — sometimes 2s is too short

// Replace with explicit condition
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("welcomeHeading")));
```

---

## How to Say All of This in an Interview

If asked "how do you debug test automation code," here is the spoken structure:

"My first rule is to never touch the code before reading everything the failure already gives you — error message, stack trace, screenshot, network logs. Most debugging time is wasted by people who start changing things before understanding what happened.

Then I run the failing test in isolation to determine whether it's a real failure or an ordering dependency — a test that fails with others but passes alone has a shared state problem, not a logic problem.

If the failure is real but not obvious, I add structured logging at the key decision points — the start of each major action, what data came back from the API, what URL we're on — so I have a timeline of what the test was doing before it broke.

For hard to locate bugs in longer flows, I bisect — add a checkpoint halfway through to see if state is correct there. If it is, the bug is in the second half. Keep halving until you find the exact step where state diverges.

For anything involving backend data, I always check what API call was actually made and what the API actually returned. That immediately tells me whether the bug is in the UI layer or the backend layer — two completely different teams to involve.

And for flaky tests, I never retry and suppress — I find the non-determinism. Usually it's a hardcoded sleep that isn't long enough sometimes, or shared test data that two parallel tests are both trying to use, or a backend response time that varies. Fix the root cause, don't mask the symptom."

---

## Quick Reference

```
FAILURE TYPE                        DEBUGGING TECHNIQUE
────────────────────────────────────────────────────────────────────
Fails in CI, passes locally         Run in isolation, check shared state
Fails with suite, passes alone      Test ordering dependency, find shared state
Assertion message unhelpful         Log actual state before asserting
Don't know which step is wrong      Bisect — checkpoint at halfway point
UI shows wrong data                 Check what API actually returned
Not sure if UI or backend bug       Capture network calls, inspect request/response
Fails intermittently                Log timestamps, find hardcoded sleeps, check parallel data conflicts
Fail message is generic timeout     Wrap with business context, read stack trace for location
Can't reproduce CI failure locally  Download trace file, time-travel through it
Setup fails partway through         Check finally block runs cleanup regardless
```
