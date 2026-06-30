# Exception Handling Patterns for Robust Test Automation

## The Core Idea — Say This First in an Interview

If an interviewer asks about exception handling, start with this one sentence: "A test should fail loudly with a clear reason, never fail silently or pass when it shouldn't have." Everything else is just different techniques to achieve that.

There are two failure modes you are trying to prevent. One: a real bug exists but your test passes anyway because an exception got swallowed. Two: your test fails but the error message is useless, so debugging takes 30 minutes instead of 30 seconds. Every pattern below solves one of these two problems.

---

## Pattern 1 — Never Use Empty Catch Blocks

### WHAT
An empty catch block catches an exception and does nothing with it. The error disappears. Nobody knows it happened.

### WHY
This is the single most dangerous thing in test automation. If an element click fails because the element moved, an empty catch swallows that failure and the test continues as if nothing happened. The next assertion might pass by accident, and you ship a bug to production with a green test suite.

### HOW — simple code to write in front of an interviewer

```typescript
// playwright typescript

// BAD — swallows the error, test continues blindly
async function clickBad(locator: Locator) {
  try {
    await locator.click();
  } catch (e) {
    // nothing here — this is the bug
  }
}

// GOOD — log it and re-throw so the test fails with a clear reason
async function clickGood(locator: Locator) {
  try {
    await locator.click();
  } catch (e) {
    console.error(`Click failed on locator: ${locator}`, e);
    throw e;  // re-throw — let the test fail
  }
}
```

```java
// selenium java

// BAD
public void clickBad(WebElement element) {
  try {
    element.click();
  } catch (Exception e) {
    // nothing — silent failure
  }
}

// GOOD
public void clickGood(WebElement element) {
  try {
    element.click();
  } catch (Exception e) {
    System.err.println("Click failed: " + e.getMessage());
    throw e;  // re-throw
  }
}
```

### WHEN
Never use an empty catch. If you genuinely don't care about an exception, you still log it at minimum. The only exception to this rule is pattern 2 below.

---

## Pattern 2 — Catch Specific Exceptions, Not Generic Ones

### WHAT
Catch the exact exception type you expect, not a blanket `catch (Exception e)` or `catch (e)`. 

### WHY
If you catch everything generically, you might accidentally swallow a real bug that has nothing to do with what you intended to handle. Say you wanted to catch "element not found" so you can retry — a generic catch will also catch "network timeout" or "assertion failed" and hide them too.

### HOW

```typescript
// playwright typescript

import { TimeoutError } from 'playwright';

// BAD — catches everything, hides unrelated bugs
async function waitForElementBad(locator: Locator) {
  try {
    await locator.waitFor({ state: 'visible', timeout: 5000 });
  } catch (e) {
    return false; // swallows ANY error, not just timeout
  }
}

// GOOD — only catch the timeout, let other errors propagate
async function waitForElementGood(locator: Locator): Promise<boolean> {
  try {
    await locator.waitFor({ state: 'visible', timeout: 5000 });
    return true;
  } catch (e) {
    if (e instanceof TimeoutError) {
      console.warn(`Element not visible within timeout: ${locator}`);
      return false;
    }
    throw e; // anything else — let it fail loudly
  }
}
```

```java
// selenium java

import org.openqa.selenium.TimeoutException;
import org.openqa.selenium.NoSuchElementException;

// BAD — catches everything
public boolean waitForElementBad(WebDriver driver, By locator) {
  try {
    new WebDriverWait(driver, Duration.ofSeconds(5))
      .until(ExpectedConditions.visibilityOfElementLocated(locator));
    return true;
  } catch (Exception e) {
    return false; // hides real bugs, not just timeouts
  }
}

// GOOD — catch only what you expect
public boolean waitForElementGood(WebDriver driver, By locator) {
  try {
    new WebDriverWait(driver, Duration.ofSeconds(5))
      .until(ExpectedConditions.visibilityOfElementLocated(locator));
    return true;
  } catch (TimeoutException e) {
    System.out.println("Element not visible in time: " + locator);
    return false;
  }
  // NoSuchElementException, StaleElementReferenceException, etc. — NOT caught here
  // They propagate up and fail the test, which is correct
}
```

## WHEN
Use this every time you write a try-catch. Ask yourself: "What specific exception am I expecting?" Catch only that one. Let everything else bubble up.

## Q. do you simply mean to say that waitforelement method should only catch or handle timeout related exception since it is wait method?
Yes, exactly. You nailed it.
A waitForElement method has one job: to wait for a specific time limit. [1, 2] 
## Why this is the rule

* Expected Failures: Timing out is an expected outcome of waiting. Therefore, the method should handle it (by returning false or printing a log). [3, 4] 
* Unexpected Failures: Anything else (like a broken locator, a dead browser, or a NullPointerException) is a structural bug. A wait method should never hide structural bugs. [5] 

By only catching TimeoutException, you let the method do its job without accidentally covering up developer errors.
Would you like to look at how to properly log these timeouts, or should we write a custom exception wrapper for your framework next?


---

## Pattern 3 — Wrap Low-Level Errors With Business Context

### WHAT
When an exception happens, the raw error message is often technical and unhelpful, for example "TimeoutError: locator.click: Timeout 5000ms exceeded." Wrap it with what the test was actually trying to do, in plain English.

### WHY
At 2am when a CI pipeline fails, you don't want to spend 20 minutes figuring out which business flow broke from a generic timeout message. A good error message tells you exactly what failed and why it matters.

### HOW

```typescript
// playwright typescript

class PaymentPage {
  constructor(private page: Page) {}

  async submitPayment(cardNumber: string): Promise<void> {
    try {
      await this.page.getByLabel('Card Number').fill(cardNumber);
      await this.page.getByRole('button', { name: 'Pay Now' }).click();
    } catch (e) {
      // wrap with business context — this is what the next engineer needs
      throw new Error(
        `Failed to submit payment for card ending in ${cardNumber.slice(-4)}. ` +
        `Original error: ${(e as Error).message}`
      );
    }
  }
}
```

```java
// selenium java

public class PaymentPage {
  private WebDriver driver;

  public void submitPayment(String cardNumber) {
    try {
      driver.findElement(By.id("cardNumber")).sendKeys(cardNumber);
      driver.findElement(By.id("payButton")).click();
    } catch (Exception e) {
      throw new RuntimeException(
        "Failed to submit payment for card ending in " + cardNumber.substring(cardNumber.length() - 4) +
        ". Original error: " + e.getMessage(), e
      );
    }
  }
}
```

### WHEN
Use this in your Page Objects and Service Objects, especially for business-critical flows like payments, login, checkout. Don't bother for trivial actions like clicking a static link.

---

## Pattern 4 — Fail Fast at Setup, Not Mid-Test

### WHAT
Validate everything you need before the test starts — environment variables, test data, API connectivity. If something is missing, fail immediately with a clear message, before any test logic runs.

### WHY
If a config value is missing, you don't want 50 tests to fail one by one with confusing errors 10 minutes into a run. You want the very first thing that runs to say "BASE_URL is not set" and stop the whole suite instantly.

### HOW

```typescript
// playwright typescript — global setup

// playwright.config.ts calls this before any test runs
function validateEnv(): void {
  const required = ['BASE_URL', 'API_URL', 'ADMIN_EMAIL', 'ADMIN_PASSWORD'];
  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    // fail immediately, before a single test starts
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

```java
// selenium java — TestNG suite-level setup

import org.testng.annotations.BeforeSuite;

public class SuiteSetup {

  @BeforeSuite
  public void validateEnv() {
    String[] required = { "BASE_URL", "API_URL", "ADMIN_EMAIL", "ADMIN_PASSWORD" };
    for (String key : required) {
      if (System.getenv(key) == null) {
        throw new IllegalStateException("Missing required environment variable: " + key);
      }
    }
  }
}
```

### WHEN
Always do this at the start of a test run — global setup in Playwright, `@BeforeSuite` in TestNG. Never let a missing config value surface as a random mid-test failure.

---

## Pattern 5 — Custom Exception Classes for Different Failure Categories

### WHAT
Instead of throwing generic `Error` or `Exception` everywhere, create specific exception classes: `ElementNotFoundException`, `ApiAuthenticationException`, `TestDataSetupException`. Each one tells you at a glance what category of problem occurred.

### WHY
When you look at a CI failure log, the exception type alone should tell you where to start looking. "TestDataSetupException" means look at your database seed script. "ApiAuthenticationException" means check if a token expired. This saves debugging time across a whole team, not just you.

### HOW

```typescript
// playwright typescript

// Define custom exception types
class TestDataSetupException extends Error {
  constructor(message: string, public readonly cause?: Error) {
    super(message);
    this.name = 'TestDataSetupException';
  }
}

class ApiAuthenticationException extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ApiAuthenticationException';
  }
}

// Usage
class UserService {
  async createTestUser(email: string): Promise<{ id: string }> {
    const response = await this.request.post('/api/users', { data: { email } });

    if (response.status() === 401) {
      throw new ApiAuthenticationException('API token expired or invalid — check TOKEN env var');
    }

    if (!response.ok()) {
      throw new TestDataSetupException(
        `Failed to create test user ${email}, status: ${response.status()}`
      );
    }

    return response.json() as Promise<{ id: string }>;
  }

  constructor(private request: import('@playwright/test').APIRequestContext) {}
}
```

```java
// selenium java

// Custom exception classes
class TestDataSetupException extends RuntimeException {
  public TestDataSetupException(String message) { super(message); }
  public TestDataSetupException(String message, Throwable cause) { super(message, cause); }
}

class ApiAuthenticationException extends RuntimeException {
  public ApiAuthenticationException(String message) { super(message); }
}

// Usage
public class UserService {

  public String createTestUser(String email) {
    Response response = RestAssured.given()
      .baseUri(System.getenv("API_URL"))
      .header("Authorization", "Bearer " + System.getenv("TOKEN"))
      .body("{\"email\":\"" + email + "\"}")
      .post("/api/users");

    if (response.statusCode() == 401) {
      throw new ApiAuthenticationException("API token expired or invalid — check TOKEN env var");
    }

    if (response.statusCode() != 201) {
      throw new TestDataSetupException(
        "Failed to create test user " + email + ", status: " + response.statusCode()
      );
    }

    return response.jsonPath().getString("id");
  }
}
```

### WHEN
Use this once your framework has grown past a handful of tests. For a quick prototype it is overkill — but for any framework that 5+ engineers will use, custom exceptions massively speed up triage.

---

## Pattern 6 — Always Clean Up With finally, Even on Failure

### WHAT
Use `finally` blocks (or `afterEach`/`@AfterMethod`) to guarantee cleanup runs whether the test passes or fails. Test data, browser sessions, and database connections must be cleaned up no matter what.

### WHY
If test setup creates a user via API and the test fails midway, without `finally` that user is never deleted. After a few hundred CI runs you have thousands of orphaned records polluting your test database, which eventually causes unrelated tests to fail because of duplicate emails or hit row limits.

### HOW

```typescript
// playwright typescript

test('payment can be refunded', async ({ request }) => {
  let userId: string | undefined;

  try {
    const user = await userService.createUser('test@test.com');
    userId = user.id;

    // actual test logic — might throw
    const payment = await paymentService.createPayment(userId, 100);
    expect(payment.status).toBe('completed');

  } finally {
    // ALWAYS runs — whether the test passed or threw an exception above
    if (userId) {
      await userService.deleteUser(userId).catch(err =>
        console.warn(`Cleanup failed for user ${userId}:`, err)
      );
    }
  }
});
```

```java
// selenium java

@Test
public void paymentCanBeRefunded() {
  String userId = null;

  try {
    String user = userService.createUser("test@test.com");
    userId = user;

    // actual test logic — might throw
    Payment payment = paymentService.createPayment(userId, 100);
    Assert.assertEquals(payment.getStatus(), "completed");

  } finally {
    // ALWAYS runs
    if (userId != null) {
      try {
        userService.deleteUser(userId);
      } catch (Exception e) {
        System.err.println("Cleanup failed for user " + userId + ": " + e.getMessage());
      }
    }
  }
}
```

### WHEN
Use this for any test that creates external state: database records, API resources, files, browser contexts. Even simpler — use Playwright fixtures or TestNG `@AfterMethod`, which apply this pattern automatically.

---

## Pattern 7 — Retry Only Transient Failures, Never Logic Failures

### WHAT
Some failures are temporary — a flaky network blip, a slow server response. Others are real bugs — wrong calculation, broken assertion. Retry should only apply to the first category, never the second.

### WHY
If you retry blindly on every failure, you hide real bugs behind "it passed on attempt 2." A test that retries an assertion failure is lying to you — the bug is still there, you just stopped looking.

### HOW

```typescript
// playwright typescript

// GOOD — retry only network-level transient errors
async function fetchWithRetry(request: APIRequestContext, url: string, maxAttempts = 3) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const response = await request.get(url);
      // 5xx = server issue, worth retrying. 4xx = client/logic error, don't retry
      if (response.status() >= 500 && attempt < maxAttempts) {
        console.warn(`Attempt ${attempt}: server error ${response.status()}, retrying...`);
        await new Promise(r => setTimeout(r, 1000 * attempt));
        continue;
      }
      return response;
    } catch (e) {
      if (attempt === maxAttempts) throw e;
      console.warn(`Attempt ${attempt} failed with network error, retrying...`);
      await new Promise(r => setTimeout(r, 1000 * attempt));
    }
  }
  throw new Error('unreachable');
}

// NEVER do this — retrying an assertion hides real bugs
// for (let i = 0; i < 3; i++) {
//   try { expect(total).toBe(100); break; } catch (e) { continue; }
// }
```

```java
// selenium java

public Response fetchWithRetry(String url, int maxAttempts) {
  Exception lastException = null;

  for (int attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      Response response = RestAssured.given().baseUri(url).get();
      // Only retry server errors, not client errors
      if (response.statusCode() >= 500 && attempt < maxAttempts) {
        System.out.println("Attempt " + attempt + ": server error, retrying...");
        Thread.sleep(1000L * attempt);
        continue;
      }
      return response;
    } catch (Exception e) {
      lastException = e;
      if (attempt == maxAttempts) break;
      try { Thread.sleep(1000L * attempt); } catch (InterruptedException ignored) {}
    }
  }
  throw new RuntimeException("Failed after " + maxAttempts + " attempts", lastException);
}

// NEVER retry an assertion — this hides bugs
// for (int i = 0; i < 3; i++) {
//   try { Assert.assertEquals(total, 100); break; } catch (AssertionError e) { continue; }
// }
```

### WHEN
Use retry for network calls, flaky third-party services, or known infrastructure flakiness. Never use retry to make an assertion pass. If a test needs retries to consistently pass, that is a signal the test or the app has a real timing problem worth fixing, not hiding.

---

## Pattern 8 — Soft Assertions to Collect Multiple Failures in One Run

### WHAT
Normally, the first failed assertion stops the test immediately. Soft assertions let you continue checking multiple things and report all failures together at the end.

### WHY
If you are validating 10 fields on a form and the first one fails, a hard assertion stops the test — you fix that field, rerun, and find the second field is also broken. With soft assertions you find all 10 problems in one run, saving multiple rerun cycles.

### HOW

```typescript
// playwright typescript — built-in soft assertion

test('user profile shows correct data', async ({ page }) => {
  await page.goto('/profile');

  // soft assertions — test continues even if one fails
  await expect.soft(page.getByTestId('name')).toHaveText('John Doe');
  await expect.soft(page.getByTestId('email')).toHaveText('john@test.com');
  await expect.soft(page.getByTestId('role')).toHaveText('Admin');

  // all three failures (if any) are reported together at the end of the test
});
```

```java
// selenium java — TestNG SoftAssert

import org.testng.asserts.SoftAssert;

@Test
public void userProfileShowsCorrectData() {
  driver.get(BASE_URL + "/profile");
  SoftAssert softAssert = new SoftAssert();

  softAssert.assertEquals(driver.findElement(By.id("name")).getText(), "John Doe");
  softAssert.assertEquals(driver.findElement(By.id("email")).getText(), "john@test.com");
  softAssert.assertEquals(driver.findElement(By.id("role")).getText(), "Admin");

  // collects all 3 failures and reports them together at this line
  softAssert.assertAll();
}
```

### WHEN
Use soft assertions when checking multiple independent fields on the same screen — form validation, profile data, dashboard widgets. Do not use soft assertions for sequential steps where step 2 depends on step 1 succeeding — use hard assertions there, since continuing after a real failure wastes time and produces confusing downstream errors.

### Practical Examples of Soft Assertions in Test Automation

**Soft assertions** allow a test to continue running even if an assertion fails. Unlike hard assertions that immediately abort the test, soft assertions record failures and report them all together at the end. This is ideal for validating multiple UI elements or deep API responses in a single run.

---

## UI Test Automation Example
* **Scenario:** Validating a complex E-commerce product details page.
* **Why use soft assertions:** If the "Add to Cart" button is missing, it’s a critical failure. However, if the product description has a typo or the product review count is off by one, the test should still evaluate the rest of the page layout so you know exactly which elements are broken.

### Playwright (TypeScript) Implementation:

```typescript
test('Verify multiple product details on E-commerce page', async ({ page }) => {
  await page.goto('https://example.com');

  // Use expect.soft() instead of expect()
  await expect.soft(page.locator('#product-title')).toHaveText('Wireless Headphones');
  await expect.soft(page.locator('#product-price')).toHaveText('\$99.99');
  await expect.soft(page.locator('#review-count')).toHaveText('1,250');
  await expect.soft(page.locator('#add-to-cart-btn')).toBeVisible();

  // Test fails here if any soft assertions above failed
  expect(page.errors()).toHaveLength(0); 
});
```

---

## API Test Automation Example
* **Scenario:** Validating a GET response for a user profile API endpoint.
* **Why use soft assertions:** Instead of making separate network calls to check the status code, user ID, email, and account status, you retrieve them all at once. If the user's email is wrong, you still want to verify their account status and the HTTP response headers in the exact same payload.

### REST-Assured & AssertJ (Java) Implementation:

```java
@Test
public void verifyUserProfileApiResponse() {
    Response response = given()
        .baseUri("https://example.com")
        .when()
        .get("/users/101")
        .then()
        .statusCode(200) // Hard assertion: Stop if API is down
        .extract().response();

    // Soft Assertions start here
    SoftAssertions softly = new SoftAssertions();

    softly.assertThat(response.jsonPath().getString("data.email")).isEqualTo("test@example.com");
    softly.assertThat(response.jsonPath().getString("data.first_name")).isEqualTo("John");
    softly.assertThat(response.jsonPath().getBoolean("data.is_active")).isTrue();
    softly.assertThat(response.getHeader("Content-Type")).isEqualTo("application/json");

    // Report all failures together
    softly.assertAll(); 
}
```

---

## Pattern 9 — Don't Catch Exceptions Just to Swallow Test Failures

### WHAT
This is the most common mistake junior SDETs make under pressure to turn a red build green: wrapping a flaky or broken step in a try-catch that marks the test as passed regardless of the outcome.

### WHY
This is worse than a flaky test. A flaky test at least tells you something is wrong. A test that catches its own failure and reports success is actively lying to the team — nobody will ever investigate the underlying issue because the dashboard shows green.

### HOW

```typescript
// playwright typescript

// BAD — this test will ALWAYS pass, even when the feature is broken
test('checkout completes successfully', async ({ page }) => {
  try {
    await page.goto('/checkout');
    await page.getByRole('button', { name: 'Place order' }).click();
    await expect(page.getByText('Order confirmed')).toBeVisible();
  } catch (e) {
    console.log('something went wrong but continuing'); // DO NOT DO THIS
  }
});

// GOOD — let it fail, that's the whole point of a test
test('checkout completes successfully', async ({ page }) => {
  await page.goto('/checkout');
  await page.getByRole('button', { name: 'Place order' }).click();
  await expect(page.getByText('Order confirmed')).toBeVisible();
});
```

```java
// selenium java

// BAD — always passes
@Test
public void checkoutCompletesSuccessfully() {
  try {
    driver.get(BASE_URL + "/checkout");
    driver.findElement(By.id("placeOrder")).click();
    Assert.assertTrue(driver.findElement(By.id("confirmation")).isDisplayed());
  } catch (Exception e) {
    System.out.println("ignoring failure"); // DO NOT DO THIS
  }
}

// GOOD
@Test
public void checkoutCompletesSuccessfully() {
  driver.get(BASE_URL + "/checkout");
  driver.findElement(By.id("placeOrder")).click();
  Assert.assertTrue(driver.findElement(By.id("confirmation")).isDisplayed());
}
```

### WHEN
This is the single anti-pattern you should call out unprompted if an interviewer asks "what's the worst exception handling mistake you've seen." It usually comes from pressure to hit a green CI dashboard before a release, and it is a culture problem as much as a code problem.

---

## How to Structure Your Answer in an Interview

If asked "tell me about your approach to exception handling in test automation," structure your answer in this order:

First, state the principle: tests should fail loudly with clear context, never silently. Second, give one concrete example of an empty catch block causing a false-positive test pass — this shows you've actually been burned by this in production. Third, mention you catch specific exception types, not generic ones, so you never accidentally hide unrelated bugs. Fourth, mention you use `finally` blocks or fixtures to guarantee cleanup happens regardless of test outcome, which prevents test data pollution. Fifth, mention retry is reserved strictly for transient infrastructure issues, never for hiding real assertion failures. If there's time, mention custom exception classes for faster triage across a team, and soft assertions for collecting multiple field-level failures in one test run.

---

## Quick Reference Table

```
SITUATION                              WHAT TO DO
─────────────────────────────────────────────────────────────────────
Element click fails                    Catch TimeoutError specifically, log + re-throw
Don't know what exception to expect    Catch the narrowest type, let rest propagate
Generic error message on failure       Wrap with business context before throwing
Missing config / env var               Validate at suite startup, fail before tests run
Test creates DB/API records            Use finally or fixture teardown — always cleanup
Network call sometimes times out       Retry 2-3 times with backoff — only for network
Assertion sometimes fails              DO NOT retry — that's a real bug, investigate it
Checking 5+ independent fields         Use soft assertions, collect all failures at once
Step 2 depends on step 1 succeeding    Use hard assertion on step 1 — don't continue if it fails
Tempted to try-catch a failing feature DO NOT — let the test fail, that is its job
```
