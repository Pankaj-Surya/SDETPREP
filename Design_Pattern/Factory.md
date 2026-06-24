## Pattern 4 — Factory

### What it is

Factory is a pattern where one class is responsible for creating objects. Instead of calling `new LoginPage(page)` in every test, you call `PageFactory.create('login', page)`. The factory decides what to create and how.

### Why it applies to test automation

As frameworks grow, tests create many different page objects and data objects. If the constructor signature changes, you update the factory — not every test. Factories also allow conditional creation: return a mobile version of a page when running on a mobile browser, return a desktop version otherwise.

### When to use

Use it when you have many types of the same kind of thing to create. Use it when creation logic is complex or conditional. Use it when you need to swap implementations based on environment.

### When NOT to use

Do not use it for simple cases where `new LoginPage(page)` is clear and direct. Factories add a level of indirection — only add that indirection when it is solving a real problem.

---

### TypeScript — Playwright

```typescript
import { Page } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

export class TestPageFactory {
  private readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Returns a new instance of LoginPage
  public loginPage(): LoginPage {
    return new LoginPage(this.page);
  }

  // Returns a new instance of DashboardPage
  public dashboardPage(): DashboardPage {
    return new DashboardPage(this.page);
  }
}
```

```typescript
import { Page } from '@playwright/test';

export class LoginPage {
  constructor(private page: Page) {}

  async navigate() {
    await this.page.goto('/login');
  }
}
```

```typescript
import { Page } from '@playwright/test';

export class DashboardPage {
  constructor(private page: Page) {}

  async getHeader() {
    return this.page.locator('h1');
  }
}
```
```typescript
import { test, expect } from '@playwright/test';
import { TestPageFactory } from '../factories/PageFactory';

test('Login and view dashboard', async ({ page }) => {
  // Initialize the factory
  const pages = new TestPageFactory(page);

  // Use the factory to get page instances
  await pages.loginPage().navigate();
  
  const dashboardHeader = await pages.dashboardPage().getHeader();
  await expect(dashboardHeader).toHaveText('Welcome Back');
});
```

---

### Java — Selenium

```java
// src/main/java/factories/PageFactory.java
// NOTE: Playwright has its own PageFactory — this is our custom one
package factories;

import org.openqa.selenium.WebDriver;
import pages.BasePage;
import pages.LoginPage;
import pages.DashboardPage;

public class TestPageFactory {

  private final WebDriver driver;

  public TestPageFactory(WebDriver driver) {
    this.driver = driver;
  }

  public LoginPage loginPage()         { return new LoginPage(driver); }
  public DashboardPage dashboardPage() { return new DashboardPage(driver); }
}
```



