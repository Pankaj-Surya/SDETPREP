# Design Patterns for Test Automation — Complete Guide

## Pattern 1 — Page Object Model (POM)

### What it is

POM is the most used pattern in test automation. The idea is simple: create one class per page or screen. That class owns all the locators for that page and all the actions a user can perform on it. Tests never touch locators directly — they call methods on the page class.

### Why it applies to test automation

Without POM, every test has raw selectors scattered everywhere. When a developer renames a button, you update 40 test files. With POM, you update one class. The pattern enforces encapsulation — locators are private, actions are public. Tests become readable because they describe what a user does, not which CSS selector to click.

### When to use

Use it for every page or significant UI component in your application. Any UI that multiple tests interact with should have a page object.

### When NOT to use

Do not use it for one-off tests that run once and will never be maintained. Do not create a page object for a page that only one test touches if that test is a throwaway script. Do not use it for API-only tests — that is what the Service Object pattern is for.

---

### TypeScript — Playwright

```typescript
// pages/base.page.ts
// Base class that all page objects extend
// Contains shared behavior: navigation, waiting, common assertions

import { type Page, type Locator } from '@playwright/test';

export abstract class BasePage {
  protected readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Every page must know how to navigate to itself
  abstract goto(): Promise<void>;

  // Every page must know when it has fully loaded
  abstract isLoaded(): Promise<boolean>;

  // Shared utilities all pages can use
  async getTitle(): Promise<string> {
    return this.page.title();
  }

  async getCurrentUrl(): Promise<string> {
    return this.page.url();
  }

  async waitForNetworkIdle(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }

  async scrollToBottom(): Promise<void> {
    await this.page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
  }
}
```

```typescript
// pages/login.page.ts
// Encapsulates everything about the login page
// Tests call methods — they never touch locators directly

import { type Page, type Locator } from '@playwright/test';
import { BasePage } from './base.page';

export class LoginPage extends BasePage {
  // Private: no test can directly access these selectors
  // This is the key value of POM — change selector here, nothing else breaks
  private readonly emailInput: Locator;
  private readonly passwordInput: Locator;
  private readonly submitButton: Locator;
  private readonly errorAlert: Locator;
  private readonly forgotPasswordLink: Locator;
  private readonly rememberMeCheckbox: Locator;

  constructor(page: Page) {
    super(page);
    this.emailInput          = page.getByLabel('Email');
    this.passwordInput       = page.getByLabel('Password');
    this.submitButton        = page.getByRole('button', { name: 'Sign in' });
    this.errorAlert          = page.getByRole('alert');
    this.forgotPasswordLink  = page.getByRole('link', { name: 'Forgot password' });
    this.rememberMeCheckbox  = page.getByRole('checkbox', { name: 'Remember me' });
  }

  // Required by BasePage
  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async isLoaded(): Promise<boolean> {
    return this.submitButton.isVisible();
  }

  // Actions — each method does one thing
  async fillEmail(email: string): Promise<void> {
    await this.emailInput.fill(email);
  }

  async fillPassword(password: string): Promise<void> {
    await this.passwordInput.fill(password);
  }

  async clickSubmit(): Promise<void> {
    await this.submitButton.click();
  }

  async checkRememberMe(): Promise<void> {
    await this.rememberMeCheckbox.check();
  }

  async clickForgotPassword(): Promise<void> {
    await this.forgotPasswordLink.click();
  }

  // Convenience method combining multiple actions
  // Useful for tests that just need to get past login
  async login(email: string, password: string): Promise<void> {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.clickSubmit();
  }

  async loginWithRememberMe(email: string, password: string): Promise<void> {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.checkRememberMe();
    await this.clickSubmit();
  }

  // State readers — return data for tests to assert on
  // Assertion happens in the TEST, not here
  async getErrorText(): Promise<string> {
    return (await this.errorAlert.textContent()) ?? '';
  }

  async isErrorVisible(): Promise<boolean> {
    return this.errorAlert.isVisible();
  }

  async isSubmitButtonEnabled(): Promise<boolean> {
    return this.submitButton.isEnabled();
  }
}
```

```typescript
// pages/dashboard.page.ts
// Different page — same pattern, different behavior

import { type Page, type Locator } from '@playwright/test';
import { BasePage } from './base.page';

export class DashboardPage extends BasePage {
  private readonly welcomeHeading: Locator;
  private readonly newItemButton: Locator;
  private readonly searchInput: Locator;
  private readonly userMenuButton: Locator;
  private readonly signOutButton: Locator;
  private readonly notificationBadge: Locator;

  constructor(page: Page) {
    super(page);
    this.welcomeHeading    = page.getByRole('heading', { level: 1 });
    this.newItemButton     = page.getByRole('button', { name: 'New item' });
    this.searchInput       = page.getByPlaceholder('Search...');
    this.userMenuButton    = page.getByTestId('user-menu-button');
    this.signOutButton     = page.getByRole('menuitem', { name: 'Sign out' });
    this.notificationBadge = page.getByTestId('notification-badge');
  }

  async goto(): Promise<void> {
    await this.page.goto('/dashboard');
  }

  async isLoaded(): Promise<boolean> {
    return this.welcomeHeading.isVisible();
  }

  async getWelcomeText(): Promise<string> {
    return (await this.welcomeHeading.textContent()) ?? '';
  }

  async clickNewItem(): Promise<void> {
    await this.newItemButton.click();
  }

  async search(query: string): Promise<void> {
    await this.searchInput.fill(query);
    await this.page.keyboard.press('Enter');
  }

  async signOut(): Promise<void> {
    await this.userMenuButton.click();
    await this.signOutButton.click();
  }

  async getNotificationCount(): Promise<number> {
    const text = await this.notificationBadge.textContent();
    return parseInt(text ?? '0', 10);
  }
}
```

```typescript
// tests/login.spec.ts
// Clean test — calls page methods, asserts results
// Zero locators in test code

import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/login.page';
import { DashboardPage } from '../pages/dashboard.page';

test.describe('Login', () => {

  test('valid credentials redirect to dashboard', async ({ page }) => {
    const loginPage     = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    await loginPage.goto();
    await loginPage.login('user@example.com', 'Password123!');

    // Assert in the test — never inside the page object
    expect(await dashboardPage.isLoaded()).toBe(true);
    await expect(page).toHaveURL(/dashboard/);
  });

  test('wrong password shows error message', async ({ page }) => {
    const loginPage = new LoginPage(page);

    await loginPage.goto();
    await loginPage.login('user@example.com', 'wrongpassword');

    expect(await loginPage.isErrorVisible()).toBe(true);
    expect(await loginPage.getErrorText()).toContain('Invalid credentials');
  });

  test('empty email keeps submit disabled', async ({ page }) => {
    const loginPage = new LoginPage(page);

    await loginPage.goto();
    await loginPage.fillPassword('somepassword');

    expect(await loginPage.isSubmitButtonEnabled()).toBe(false);
  });

});
```

---

### Java — Selenium

```java
// src/main/java/pages/BasePage.java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public abstract class BasePage {

  protected final WebDriver driver;
  protected final WebDriverWait wait;

  public BasePage(WebDriver driver) {
    this.driver = driver;
    this.wait   = new WebDriverWait(driver, Duration.ofSeconds(15));
  }

  public abstract void goTo();
  public abstract boolean isLoaded();

  public String getTitle()      { return driver.getTitle(); }
  public String getCurrentUrl() { return driver.getCurrentUrl(); }
}
```

```java
// src/main/java/pages/LoginPage.java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;

public class LoginPage extends BasePage {

  // Private locators — the whole point of POM
  private final By emailLocator    = By.id("email");
  private final By passwordLocator = By.id("password");
  private final By submitLocator   = By.cssSelector("button[type='submit']");
  private final By errorLocator    = By.cssSelector("[role='alert']");

  public LoginPage(WebDriver driver) {
    super(driver);
  }

  @Override
  public void goTo() {
    driver.get(System.getenv("BASE_URL") + "/login");
  }

  @Override
  public boolean isLoaded() {
    try {
      return driver.findElement(submitLocator).isDisplayed();
    } catch (org.openqa.selenium.NoSuchElementException e) {
      return false;
    }
  }

  public void enterEmail(String email) {
    WebElement el = wait.until(ExpectedConditions.visibilityOfElementLocated(emailLocator));
    el.clear();
    el.sendKeys(email);
  }

  public void enterPassword(String password) {
    WebElement el = wait.until(ExpectedConditions.visibilityOfElementLocated(passwordLocator));
    el.clear();
    el.sendKeys(password);
  }

  public void clickSubmit() {
    wait.until(ExpectedConditions.elementToBeClickable(submitLocator)).click();
  }

  public void login(String email, String password) {
    enterEmail(email);
    enterPassword(password);
    clickSubmit();
  }

  public String getErrorMessage() {
    return wait.until(ExpectedConditions.visibilityOfElementLocated(errorLocator)).getText();
  }

  public boolean isErrorDisplayed() {
    try { return driver.findElement(errorLocator).isDisplayed(); }
    catch (org.openqa.selenium.NoSuchElementException e) { return false; }
  }
}
```

---

## 
