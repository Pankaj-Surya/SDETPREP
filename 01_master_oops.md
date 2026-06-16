# OOP for SDET — Day 1 of 100

## SECTION 1 — CONCEPT MASTERY

### 1.1 What Is OOP and Why Does It Matter for SDETs?

OOP is a way of organizing code around objects — things that have data (fields) and behavior (methods). Instead of writing a list of instructions top to bottom, you model the real world: a Browser, a LoginPage, a User, a PaymentService.

For SDETs specifically, OOP is not just academic. Every framework you use — Playwright, Selenium, TestNG, JUnit — is built using OOP. When you write a Page Object, you are applying OOP. When you share setup across test classes, you are using inheritance. When you swap a mock service for a real one, you are using polymorphism.

**The real-world analogy:** Think of a car. Every car has an engine, wheels, and doors (data). Every car can accelerate, brake, and steer (behavior). A Toyota and a BMW are both cars — they share the same blueprint but have different implementations. Your test framework works the same way. `BasePage` is the car blueprint. `LoginPage` and `DashboardPage` are Toyota and BMW.

**Why Selenium Java and Playwright TypeScript developers must master this:** Both frameworks are class-based. If you cannot design a proper class hierarchy, your test code becomes a pile of duplicated methods, brittle selectors hardcoded everywhere, and tests that break when one thing changes.

---

### 1.2 Full Topic Hierarchy

```
OOP for SDET
│
├── 1. CLASS & OBJECT
│   ├── What is a class (blueprint)
│   ├── What is an object (instance)
│   ├── Fields (instance variables)
│   ├── Methods (behavior)
│   ├── Constructors (initialization)
│   ├── Static vs instance members
│   └── Object lifecycle in test context
│       ├── Created in @BeforeEach / beforeEach fixture
│       ├── Used in test method
│       └── Garbage collected after test
│
├── 2. ENCAPSULATION
│   ├── Private fields — hide implementation
│   ├── Public methods — expose behavior only
│   ├── Getters and setters (use sparingly)
│   ├── Why locators must be private in Page Objects
│   ├── Why test setup must be encapsulated in fixtures
│   └── Access modifiers: private, protected, public
│       ├── Java: private, protected, public, package-private
│       └── TypeScript: private, protected, public, readonly
│
├── 3. ABSTRACTION
│   ├── Abstract classes
│   │   ├── Cannot be instantiated directly
│   │   ├── Can have concrete + abstract methods
│   │   └── BasePage as abstract class
│   ├── Interfaces
│   │   ├── Define a contract, no implementation
│   │   ├── IPage, IAuthenticatable, INavigable
│   │   └── Java interface vs TypeScript interface
│   ├── Why abstraction matters in testing
│   │   ├── Tests talk to abstractions, not implementations
│   │   ├── Swap real service for mock without touching tests
│   │   └── Driver abstraction — swap Chrome for Firefox
│   └── Abstract vs Interface — when to use which
│
├── 4. INHERITANCE
│   ├── Extends keyword (Java and TypeScript)
│   ├── Single inheritance (both languages)
│   ├── super() — calling parent constructor
│   ├── Method override (@Override in Java, override in TS)
│   ├── When to use inheritance in tests
│   │   ├── BasePage → LoginPage, DashboardPage
│   │   ├── BaseTest → LoginTest, CheckoutTest
│   │   └── BaseAPIClient → UserServiceClient, PaymentClient
│   ├── When NOT to use inheritance
│   │   ├── Deep chains (3+ levels) — use composition instead
│   │   └── Forcing is-a when it is really has-a
│   └── Inheritance vs Composition tradeoff
│
├── 5. POLYMORPHISM
│   ├── Compile-time (method overloading)
│   │   ├── Same method name, different parameters
│   │   └── fill(String), fill(String, Boolean)
│   ├── Runtime (method overriding)
│   │   ├── Parent reference, child object
│   │   └── BasePage ref holds LoginPage object
│   ├── Interface polymorphism
│   │   ├── ILoginStrategy — FormLogin, SSOLogin, OAuthLogin
│   │   └── Switch login method without changing test code
│   └── Why polymorphism is critical for SDET
│       ├── Cross-browser: same test, different driver
│       ├── Cross-environment: same test, different config
│       └── Mock vs real: same test, different service impl
│
├── 6. ASSOCIATION
│   ├── What it means: A uses B
│   ├── Loose coupling — A does not own B
│   ├── Example: LoginPage uses WebDriver (gets it, does not create it)
│   └── Example: Test class uses LoginPage (does not own the browser)
│
├── 7. AGGREGATION
│   ├── What it means: A has B, but B can live without A
│   ├── Weak ownership
│   ├── Example: TestSuite has TestCases
│   │   └── TestCase can exist without the suite
│   ├── Example: PageFactory has pages
│   │   └── Pages can exist without the factory
│   └── Lifecycle: B survives if A is destroyed
│
├── 8. COMPOSITION
│   ├── What it means: A owns B — B cannot live without A
│   ├── Strong ownership
│   ├── Example: BrowserContext owns Page
│   │   └── Page destroyed when context closes
│   ├── Example: TestFixture owns DatabaseConnection
│   │   └── Connection closed when fixture tears down
│   ├── Favour composition over inheritance (GoF principle)
│   └── Composition in Page Objects
│       └── LoginPage has a FormComponent (not extends)
│
├── 9. SOLID PRINCIPLES (applied to testing)
│   ├── S — Single Responsibility
│   ├── O — Open/Closed
│   ├── L — Liskov Substitution
│   ├── I — Interface Segregation
│   └── D — Dependency Inversion
│
├── 10. APPLIED PATTERNS IN TEST FRAMEWORKS
│   ├── Page Object Model (encapsulation + abstraction)
│   ├── Service Object Model (abstraction + single responsibility)
│   ├── Factory Pattern (encapsulates object creation)
│   ├── Builder Pattern (step-by-step object construction)
│   ├── Strategy Pattern (polymorphism for swappable behavior)
│   ├── Singleton Pattern (one instance, global access)
│   ├── Template Method (inheritance + abstraction for test flow)
│   └── Decorator Pattern (extend behavior without inheritance)
│
└── 11. COMMON MISTAKES
    ├── Locators as public fields — exposed, fragile
    ├── Business logic in Page Objects
    ├── Assertions inside Page Objects
    ├── Deep inheritance chains (4+ levels)
    ├── Static state shared across parallel tests
    ├── God classes that do everything
    └── Inheritance when composition was the right choice
```

---

### 1.3 How Each Concept Works Internally

**Class and Object:** A class is compiled to bytecode (Java) or JavaScript (TypeScript). When you call `new LoginPage(driver)`, the JVM or V8 engine allocates memory, runs the constructor, and returns a reference. In Playwright, the `page` fixture is the object. In Selenium, `driver` is the object.

**Encapsulation at runtime:** Private fields exist only on the object's own memory space. No other object can read or write them unless you provide a public method. This is enforced at compile time in Java and TypeScript (strict mode). At runtime, JavaScript does not enforce it — but TypeScript compilation catches violations before the code runs.

**Inheritance at runtime:** When you call a method on a child class, Java/TypeScript first looks in the child class. If not found, it walks up the prototype chain (TypeScript) or class hierarchy (Java) until it finds the method. This is why `super.method()` explicitly calls the parent version.

**Polymorphism at runtime:** If you have `BasePage page = new LoginPage(driver)` in Java, and you call `page.load()`, the JVM looks at the actual object type at runtime (LoginPage), not the reference type (BasePage). It calls LoginPage's version. This is dynamic dispatch — the key mechanism behind polymorphism.

**Association vs Aggregation vs Composition — the memory difference:**
- Association: A holds a reference to B. B was created outside A and passed in.
- Aggregation: A holds a collection of B objects. B objects can be removed from A and still exist.
- Composition: A creates B inside itself. When A is garbage collected, B is too.

---

### 1.4 Key Terminology Glossary

| Term | Definition | SDET Relevance |
|------|-----------|----------------|
| Class | Blueprint defining fields and methods | Every Page Object is a class |
| Object | Instance of a class in memory | `new LoginPage(driver)` creates an object |
| Encapsulation | Hide internals, expose controlled interface | Locators private, actions public |
| Abstraction | Hide complexity behind simple interface | BasePage hides driver setup |
| Inheritance | Child class gets parent's fields and methods | LoginPage extends BasePage |
| Polymorphism | One interface, many implementations | Same test runs on Chrome and Firefox |
| Association | A uses B (no ownership) | Test uses LoginPage |
| Aggregation | A has B (B can live without A) | Suite has test cases |
| Composition | A owns B (B dies with A) | Context owns Page |
| Interface | Contract with no implementation | ILoginStrategy |
| Abstract class | Partial implementation, cannot instantiate | BasePage with abstract load() |
| Override | Child replaces parent method | LoginPage.load() overrides BasePage.load() |
| Overload | Same method name, different parameters | fill(String), fill(String, Boolean) |
| super() | Call parent constructor or method | super(driver) in child page constructor |
| static | Belongs to class, not instance | DriverFactory.getDriver() |

---

## SECTION 2 — CODE — TYPESCRIPT (PLAYWRIGHT)

### 2.1 Level 1 — Basic: Class, Object, Encapsulation

```typescript
// pages/LoginPage.ts
// This is a CLASS. It is the blueprint for a login page.
// It ENCAPSULATES the locators (private) and exposes actions (public).

import { type Page, type Locator } from '@playwright/test';

export class LoginPage {
  // PRIVATE FIELDS — no code outside this class can touch these
  // This is ENCAPSULATION: hiding the implementation details
  private readonly page: Page;
  private readonly emailInput: Locator;
  private readonly passwordInput: Locator;
  private readonly submitButton: Locator;
  private readonly errorMessage: Locator;

  // CONSTRUCTOR — runs when you do: new LoginPage(page)
  // It initializes all the locators in one place
  constructor(page: Page) {
    this.page = page;
    // Role-based locators — tied to what the user sees, not CSS
    this.emailInput    = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton  = page.getByRole('button', { name: 'Sign in' });
    this.errorMessage  = page.getByRole('alert');
  }

  // PUBLIC METHODS — the only way outside code interacts with this page
  // Tests call these, they never touch the locators directly

  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async fillEmail(email: string): Promise<void> {
    await this.emailInput.fill(email);
  }

  async fillPassword(password: string): Promise<void> {
    await this.passwordInput.fill(password);
  }

  async submit(): Promise<void> {
    await this.submitButton.click();
  }

  // Convenience method that combines the steps above
  async login(email: string, password: string): Promise<void> {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.submit();
  }

  // Returns text so the TEST can assert it — not asserting here
  async getErrorText(): Promise<string> {
    return (await this.errorMessage.textContent()) ?? '';
  }
}
```

```typescript
// tests/login.spec.ts
// This creates an OBJECT from the LoginPage CLASS
// The test talks to the page through the public interface only

import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test('user logs in with valid credentials', async ({ page }) => {
  // Create an OBJECT — an instance of the LoginPage class
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('user@example.com', 'Password123!');

  // Assertion in the test, not in the page object
  await expect(page).toHaveURL(/dashboard/);
});

test('wrong password shows error', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@example.com', 'wrongpassword');

  const error = await loginPage.getErrorText();
  expect(error).toContain('Invalid credentials');
});
```

---

### 2.2 Level 2 — Abstraction and Inheritance

```typescript
// pages/BasePage.ts
// ABSTRACT CLASS — cannot be instantiated directly
// Provides shared behavior that ALL pages need
// This is ABSTRACTION: hiding common setup from every page

import { type Page } from '@playwright/test';

export abstract class BasePage {
  // Protected — child classes can access this, outside code cannot
  protected readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Concrete method — all pages inherit this for free
  async getTitle(): Promise<string> {
    return this.page.title();
  }

  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }

  async getURL(): Promise<string> {
    return this.page.url();
  }

  // ABSTRACT METHOD — every child MUST implement this
  // This forces a contract: every page must know how to load itself
  abstract goto(): Promise<void>;

  // Abstract method — every page must define what "loaded" means
  abstract isLoaded(): Promise<boolean>;
}
```

```typescript
// pages/LoginPage.ts
// CHILD CLASS — INHERITS from BasePage
// Gets all BasePage methods for free
// Must implement goto() and isLoaded() because they are abstract

import { type Page, type Locator } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  // Private locators — encapsulation
  private readonly emailInput: Locator;
  private readonly passwordInput: Locator;
  private readonly submitButton: Locator;

  constructor(page: Page) {
    // MUST call super() first — initializes the parent (BasePage)
    super(page);
    this.emailInput    = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton  = page.getByRole('button', { name: 'Sign in' });
  }

  // IMPLEMENTING the abstract method from BasePage
  // This is MANDATORY — TypeScript will give a compile error if missing
  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  // Implementing the second abstract method
  async isLoaded(): Promise<boolean> {
    return this.submitButton.isVisible();
  }

  async login(email: string, password: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}
```

```typescript
// pages/DashboardPage.ts
// Another child class — same parent (BasePage), different behavior
// This is POLYMORPHISM: goto() and isLoaded() do different things
// but are called the same way

import { type Page, type Locator } from '@playwright/test';
import { BasePage } from './BasePage';

export class DashboardPage extends BasePage {
  private readonly welcomeHeading: Locator;
  private readonly newItemButton: Locator;
  private readonly userMenu: Locator;

  constructor(page: Page) {
    super(page);
    this.welcomeHeading = page.getByRole('heading', { name: /Welcome/ });
    this.newItemButton  = page.getByRole('button', { name: 'New item' });
    this.userMenu       = page.getByTestId('user-menu');
  }

  // DashboardPage's version of goto()
  // Same method name as LoginPage — different behavior = POLYMORPHISM
  async goto(): Promise<void> {
    await this.page.goto('/dashboard');
  }

  async isLoaded(): Promise<boolean> {
    return this.welcomeHeading.isVisible();
  }

  async clickNewItem(): Promise<void> {
    await this.newItemButton.click();
  }

  async openUserMenu(): Promise<void> {
    await this.userMenu.click();
  }
}
```

```typescript
// tests/navigation.spec.ts
// POLYMORPHISM in action — both pages are treated as BasePage
// But each executes its own goto() and isLoaded()

import { test, expect } from '@playwright/test';
import { BasePage } from '../pages/BasePage';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

test('all pages load correctly', async ({ page }) => {
  // Polymorphism: pages array holds objects of different classes
  // but all are treated as BasePage
  const pages: BasePage[] = [
    new LoginPage(page),
    new DashboardPage(page),
  ];

  for (const appPage of pages) {
    // Calls the correct goto() for EACH class at runtime
    // This is dynamic dispatch — the actual type determines behavior
    await appPage.goto();
    const loaded = await appPage.isLoaded();
    expect(loaded).toBe(true);
  }
});
```

---

### 2.3 Level 3 — Composition, Association, Aggregation

```typescript
// components/FormComponent.ts
// A reusable UI component — used in COMPOSITION
// LoginPage OWNS a FormComponent — this is COMPOSITION

import { type Page, type Locator } from '@playwright/test';

export class FormComponent {
  private readonly submitButton: Locator;
  private readonly errorAlert: Locator;

  constructor(
    private readonly page: Page,
    private readonly formSelector: string
  ) {
    // Scoped locators — only looks inside the form
    this.submitButton = page.locator(formSelector).getByRole('button', { name: /submit|save|sign in/i });
    this.errorAlert   = page.locator(formSelector).getByRole('alert');
  }

  async submit(): Promise<void> {
    await this.submitButton.click();
  }

  async getErrorText(): Promise<string> {
    return (await this.errorAlert.textContent()) ?? '';
  }

  async isErrorVisible(): Promise<boolean> {
    return this.errorAlert.isVisible();
  }
}
```

```typescript
// pages/LoginPageComposed.ts
// COMPOSITION: LoginPage HAS a FormComponent
// Instead of inheriting from a FormPage, we compose it in
// Favour composition over inheritance — easier to change, test, and maintain

import { type Page, type Locator } from '@playwright/test';
import { BasePage } from './BasePage';
import { FormComponent } from '../components/FormComponent';

export class LoginPageComposed extends BasePage {
  private readonly emailInput: Locator;
  private readonly passwordInput: Locator;

  // COMPOSITION: LoginPage creates and OWNS this FormComponent
  // If LoginPage is destroyed, form is destroyed with it
  private readonly form: FormComponent;

  constructor(page: Page) {
    super(page);
    this.emailInput    = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    // Creating the component INSIDE the constructor = composition
    this.form = new FormComponent(page, 'form[data-testid="login-form"]');
  }

  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async isLoaded(): Promise<boolean> {
    return this.emailInput.isVisible();
  }

  async login(email: string, password: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    // Delegate to the composed form component
    await this.form.submit();
  }

  async getFormError(): Promise<string> {
    return this.form.getErrorText();
  }
}
```

```typescript
// services/UserService.ts
// ASSOCIATION: Tests USE this service but do not own it
// The service is created outside the test and passed in (or via fixture)

import { type APIRequestContext } from '@playwright/test';

export interface User {
  id: string;
  email: string;
  role: string;
  token: string;
}

// Interface = ABSTRACTION: defines what a user service must do
// Tests depend on this interface, not the concrete class
export interface IUserService {
  createUser(email: string, role?: string): Promise<User>;
  deleteUser(id: string): Promise<void>;
  getUserById(id: string): Promise<User>;
}

// Concrete implementation
export class UserService implements IUserService {
  // ASSOCIATION: receives APIRequestContext — does not create it
  // The fixture creates it and passes it in
  constructor(private readonly request: APIRequestContext) {}

  async createUser(email: string, role = 'user'): Promise<User> {
    const response = await this.request.post('/api/users', {
      data: { email, role },
    });
    if (!response.ok()) {
      throw new Error(`Create user failed: ${await response.text()}`);
    }
    return response.json() as Promise<User>;
  }

  async deleteUser(id: string): Promise<void> {
    const response = await this.request.delete(`/api/users/${id}`);
    if (!response.ok()) {
      throw new Error(`Delete user failed: ${await response.text()}`);
    }
  }

  async getUserById(id: string): Promise<User> {
    const response = await this.request.get(`/api/users/${id}`);
    return response.json() as Promise<User>;
  }
}

// MOCK implementation — same interface, no real HTTP calls
// Used in unit tests of the test framework itself
export class MockUserService implements IUserService {
  private users = new Map<string, User>();

  async createUser(email: string, role = 'user'): Promise<User> {
    const user: User = { id: `mock-${Date.now()}`, email, role, token: 'mock-token' };
    this.users.set(user.id, user);
    return user;
  }

  async deleteUser(id: string): Promise<void> {
    this.users.delete(id);
  }

  async getUserById(id: string): Promise<User> {
    const user = this.users.get(id);
    if (!user) throw new Error(`User ${id} not found`);
    return user;
  }
}
```

---

### 2.4 Level 4 — Expert: Strategy Pattern + Dependency Inversion

```typescript
// strategies/LoginStrategy.ts
// STRATEGY PATTERN using POLYMORPHISM + INTERFACE
// The test does not care HOW login happens — just that it does
// Swap login method without changing any test code

import { type Page } from '@playwright/test';
import { type APIRequestContext } from '@playwright/test';

// The INTERFACE — ABSTRACTION defining the contract
export interface ILoginStrategy {
  login(page: Page, email: string, password: string): Promise<void>;
}

// Strategy 1: Standard form login
export class FormLoginStrategy implements ILoginStrategy {
  async login(page: Page, email: string, password: string): Promise<void> {
    await page.goto('/login');
    await page.getByLabel('Email').fill(email);
    await page.getByLabel('Password').fill(password);
    await page.getByRole('button', { name: 'Sign in' }).click();
  }
}

// Strategy 2: API token login — skip the form, set token directly
export class ApiTokenLoginStrategy implements ILoginStrategy {
  constructor(private readonly request: APIRequestContext) {}

  async login(page: Page, email: string, password: string): Promise<void> {
    // Get token via API — faster than form
    const response = await this.request.post('/api/auth/login', {
      data: { email, password },
    });
    const { token } = await response.json() as { token: string };

    // Set token in localStorage — no UI interaction needed
    await page.goto('/');
    await page.evaluate((t) => {
      localStorage.setItem('auth_token', t);
    }, token);
    await page.goto('/dashboard');
  }
}

// Strategy 3: SSO login — redirect to SSO provider
export class SSOLoginStrategy implements ILoginStrategy {
  async login(page: Page, email: string, _password: string): Promise<void> {
    await page.goto('/login');
    await page.getByRole('button', { name: 'Sign in with SSO' }).click();
    await page.getByLabel('Email').fill(email);
    await page.getByRole('button', { name: 'Continue' }).click();
    // Handle SSO popup/redirect flow
    await page.waitForURL(/dashboard/);
  }
}
```

```typescript
// fixtures/auth.fixture.ts
// DEPENDENCY INVERSION: fixtures inject dependencies into tests
// Tests depend on abstractions (ILoginStrategy), not concrete classes
// This is the glue between OOP concepts and the Playwright fixture system

import { test as base, type Page } from '@playwright/test';
import { FormLoginStrategy, ApiTokenLoginStrategy, type ILoginStrategy } from '../strategies/LoginStrategy';
import { UserService, MockUserService, type IUserService } from '../services/UserService';

type AuthFixtures = {
  loginStrategy: ILoginStrategy;     // abstraction injected
  userService: IUserService;         // abstraction injected
  loggedInPage: Page;
};

export const test = base.extend<AuthFixtures>({

  // Inject the login strategy — switch between form/API/SSO via env var
  loginStrategy: async ({ request }, use) => {
    const strategy = process.env.LOGIN_STRATEGY === 'api'
      ? new ApiTokenLoginStrategy(request)
      : new FormLoginStrategy();

    await use(strategy);
  },

  // Inject the user service — use mock in unit tests, real in E2E
  userService: async ({ request }, use) => {
    const service = process.env.USE_MOCK_SERVICES === 'true'
      ? new MockUserService()
      : new UserService(request);

    await use(service);
  },

  // Composed fixture: creates user + logs in using injected strategy
  loggedInPage: async ({ page, loginStrategy, userService }, use) => {
    // ASSOCIATION: loggedInPage uses userService and loginStrategy
    // It does not own them — they are passed in
    const user = await userService.createUser(
      `e2e+${Date.now()}@example.com`,
      'user'
    );

    // Polymorphism: calls the right login() based on which strategy was injected
    await loginStrategy.login(page, user.email, 'Password123!');

    await use(page);

    // Cleanup after test
    await userService.deleteUser(user.id);
  },
});

export { expect } from '@playwright/test';
```

```typescript
// tests/dashboard.spec.ts — using all OOP concepts together

import { test, expect } from '../fixtures/auth.fixture';

test.describe('Dashboard tests — OOP in action', () => {
  // Tests only know about the ABSTRACTION (loggedInPage)
  // Do not know or care about:
  // - How the user was created (real API or mock)
  // - How login happened (form or API token or SSO)
  // - What browser context was used

  test('dashboard loads for authenticated user', async ({ loggedInPage }) => {
    await expect(loggedInPage.getByRole('heading', { name: /Welcome/ })).toBeVisible();
  });

  test('user can see their profile', async ({ loggedInPage }) => {
    await loggedInPage.getByTestId('user-menu').click();
    await expect(loggedInPage.getByText('@example.com')).toBeVisible();
  });

  test('new item button is visible', async ({ loggedInPage }) => {
    await expect(loggedInPage.getByRole('button', { name: 'New item' })).toBeVisible();
  });
});
```

---

### 2.5 SOLID Applied — Full TypeScript Example

```typescript
// SOLID — all 5 principles in one file with clear labels

import { type Page, type Locator, type APIRequestContext } from '@playwright/test';

// ─── S: Single Responsibility ───────────────────────────────────────────────
// ONE class does ONE thing.
// Bad: LoginPage that also validates email format and sends alerts
// Good: Each class has exactly one reason to change

class LoginFormActions {
  // Responsibility: fill and submit the login form. Nothing else.
  private email: Locator;
  private password: Locator;
  private submit: Locator;

  constructor(page: Page) {
    this.email    = page.getByLabel('Email');
    this.password = page.getByLabel('Password');
    this.submit   = page.getByRole('button', { name: 'Sign in' });
  }

  async fill(email: string, password: string): Promise<void> {
    await this.email.fill(email);
    await this.password.fill(password);
  }

  async clickSubmit(): Promise<void> {
    await this.submit.click();
  }
}

class LoginErrorReader {
  // Responsibility: read error messages from login page. Nothing else.
  private alert: Locator;
  constructor(page: Page) { this.alert = page.getByRole('alert'); }
  async getText(): Promise<string> { return (await this.alert.textContent()) ?? ''; }
  async isVisible(): Promise<boolean> { return this.alert.isVisible(); }
}

// ─── O: Open/Closed ─────────────────────────────────────────────────────────
// Open for EXTENSION, closed for MODIFICATION.
// Add new behavior by adding a new class — do not edit existing ones.

interface IReporter {
  report(testName: string, status: 'passed' | 'failed', duration: number): void;
}

class ConsoleReporter implements IReporter {
  report(testName: string, status: string, duration: number): void {
    console.log(`[${status.toUpperCase()}] ${testName} (${duration}ms)`);
  }
}

// Adding Slack reporting: NEW class, no changes to ConsoleReporter
class SlackReporter implements IReporter {
  constructor(private webhookUrl: string) {}
  report(testName: string, status: string, duration: number): void {
    // POST to Slack — ConsoleReporter untouched
    fetch(this.webhookUrl, {
      method: 'POST',
      body: JSON.stringify({ text: `${status}: ${testName} in ${duration}ms` }),
    });
  }
}

// ─── L: Liskov Substitution ─────────────────────────────────────────────────
// A child class must be usable wherever the parent is used.
// If you replace BasePage with LoginPage, tests must still pass.

abstract class BasePage {
  constructor(protected readonly page: Page) {}
  abstract goto(): Promise<void>;
  abstract isLoaded(): Promise<boolean>;

  // This method works for ANY page — child or parent
  async waitUntilLoaded(): Promise<void> {
    // Works because isLoaded() is guaranteed to exist on any child
    const loaded = await this.isLoaded();
    if (!loaded) throw new Error(`Page not loaded: ${await this.page.title()}`);
  }
}

class LoginPageLSP extends BasePage {
  private submitBtn: Locator;
  constructor(page: Page) {
    super(page);
    this.submitBtn = page.getByRole('button', { name: 'Sign in' });
  }
  async goto(): Promise<void> { await this.page.goto('/login'); }
  // isLoaded() returns true when submit button is visible
  // This is a VALID substitution — fulfills the contract
  async isLoaded(): Promise<boolean> { return this.submitBtn.isVisible(); }
}

// ─── I: Interface Segregation ───────────────────────────────────────────────
// Do not force a class to implement methods it does not need.
// Split fat interfaces into small focused ones.

// BAD — one big interface forces every page to implement search
// interface IPage { goto(): Promise<void>; search(): Promise<void>; upload(): Promise<void>; }

// GOOD — split into small interfaces
interface INavigable    { goto(): Promise<void>; }
interface ISearchable   { search(query: string): Promise<void>; }
interface IFileUploader { upload(filePath: string): Promise<void>; }

// LoginPage only needs INavigable — does not need search or upload
class LoginPageISP implements INavigable {
  constructor(private page: Page) {}
  async goto(): Promise<void> { await this.page.goto('/login'); }
}

// SearchPage needs INavigable + ISearchable
class SearchPage implements INavigable, ISearchable {
  private searchInput: Locator;
  constructor(private page: Page) {
    this.searchInput = page.getByLabel('Search');
  }
  async goto(): Promise<void> { await this.page.goto('/search'); }
  async search(query: string): Promise<void> { await this.searchInput.fill(query); }
}

// ─── D: Dependency Inversion ─────────────────────────────────────────────────
// High-level modules (tests) should not depend on low-level modules (real API).
// Both should depend on abstractions (interfaces).

// The ABSTRACTION
interface IAuthService {
  login(email: string, password: string): Promise<{ token: string }>;
  logout(token: string): Promise<void>;
}

// Low-level: real HTTP implementation
class RealAuthService implements IAuthService {
  constructor(private readonly request: APIRequestContext) {}
  async login(email: string, password: string): Promise<{ token: string }> {
    const res = await this.request.post('/api/auth/login', { data: { email, password } });
    return res.json() as Promise<{ token: string }>;
  }
  async logout(token: string): Promise<void> {
    await this.request.post('/api/auth/logout', { headers: { Authorization: `Bearer ${token}` } });
  }
}

// Low-level: mock for isolated tests
class MockAuthService implements IAuthService {
  async login(_email: string, _password: string): Promise<{ token: string }> {
    return { token: 'mock-token-abc123' };
  }
  async logout(_token: string): Promise<void> {
    // No-op
  }
}

// High-level: test helper depends on IAuthService ABSTRACTION, not RealAuthService
class AuthTestHelper {
  // Takes IAuthService — works with real or mock
  constructor(private readonly authService: IAuthService) {}

  async getValidToken(email: string, password: string): Promise<string> {
    const { token } = await this.authService.login(email, password);
    return token;
  }
}

// In tests:
// E2E: new AuthTestHelper(new RealAuthService(request))
// Unit: new AuthTestHelper(new MockAuthService())
// The test code is IDENTICAL — only the injected dependency differs
```

---

## SECTION 3 — CODE — JAVA (SELENIUM)

### 3.1 Level 1 — Basic: Class, Object, Encapsulation

```java
// src/main/java/pages/LoginPage.java
// Class with ENCAPSULATION — private locators, public actions

package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class LoginPage {

    // PRIVATE fields — ENCAPSULATION
    // Nothing outside this class can directly access these
    private final WebDriver driver;
    private final WebDriverWait wait;

    // Using By locators — define once, use many times
    private final By emailLocator    = By.id("email");
    private final By passwordLocator = By.id("password");
    private final By submitLocator   = By.cssSelector("button[type='submit']");
    private final By errorLocator    = By.cssSelector("[role='alert']");

    // CONSTRUCTOR — takes the driver (ASSOCIATION — we use it, we don't own it)
    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait   = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    // PUBLIC methods — the controlled interface to this page
    public void goTo() {
        driver.get(System.getenv("BASE_URL") + "/login");
    }

    public void enterEmail(String email) {
        WebElement input = wait.until(ExpectedConditions.visibilityOfElementLocated(emailLocator));
        input.clear();
        input.sendKeys(email);
    }

    public void enterPassword(String password) {
        WebElement input = wait.until(ExpectedConditions.visibilityOfElementLocated(passwordLocator));
        input.clear();
        input.sendKeys(password);
    }

    public void clickSubmit() {
        wait.until(ExpectedConditions.elementToBeClickable(submitLocator)).click();
    }

    // Convenience method combining all steps
    public void login(String email, String password) {
        enterEmail(email);
        enterPassword(password);
        clickSubmit();
    }

    public String getErrorMessage() {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(errorLocator)).getText();
    }

    public boolean isErrorDisplayed() {
        try {
            return driver.findElement(errorLocator).isDisplayed();
        } catch (org.openqa.selenium.NoSuchElementException e) {
            return false;
        }
    }
}
```

---

### 3.2 Level 2 — Abstract Class and Inheritance in Java

```java
// src/main/java/pages/BasePage.java
// ABSTRACT CLASS — defines template, cannot be instantiated
// This is ABSTRACTION: every page shares this, hidden from tests

package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public abstract class BasePage {

    // Protected — child classes can access, outside cannot
    protected final WebDriver driver;
    protected final WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait   = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    // Concrete methods — children inherit these for free
    public String getPageTitle() {
        return driver.getTitle();
    }

    public String getCurrentUrl() {
        return driver.getCurrentUrl();
    }

    public void refreshPage() {
        driver.navigate().refresh();
    }

    // ABSTRACT methods — every child MUST implement
    // Forces a contract: every page knows how to navigate to itself
    public abstract void goTo();

    // Forces every page to define what "loaded" means
    public abstract boolean isLoaded();
}
```

```java
// src/main/java/pages/LoginPage.java — inherits BasePage
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;

public class LoginPage extends BasePage {

    private final By emailLocator    = By.id("email");
    private final By passwordLocator = By.id("password");
    private final By submitLocator   = By.cssSelector("button[type='submit']");

    // super(driver) MUST be first line — initializes BasePage
    public LoginPage(WebDriver driver) {
        super(driver);
    }

    // Implementing ABSTRACT method — mandatory
    @Override
    public void goTo() {
        driver.get(System.getenv("BASE_URL") + "/login");
    }

    // Implementing second ABSTRACT method
    @Override
    public boolean isLoaded() {
        try {
            return driver.findElement(submitLocator).isDisplayed();
        } catch (org.openqa.selenium.NoSuchElementException e) {
            return false;
        }
    }

    public void login(String email, String password) {
        wait.until(ExpectedConditions.visibilityOfElementLocated(emailLocator)).sendKeys(email);
        wait.until(ExpectedConditions.visibilityOfElementLocated(passwordLocator)).sendKeys(password);
        wait.until(ExpectedConditions.elementToBeClickable(submitLocator)).click();
    }
}
```

```java
// src/main/java/pages/DashboardPage.java — same parent, different behavior
// POLYMORPHISM: goTo() and isLoaded() do different things

package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;

public class DashboardPage extends BasePage {

    private final By welcomeHeading = By.tagName("h1");
    private final By newItemButton  = By.xpath("//button[contains(text(),'New item')]");
    private final By userMenu       = By.cssSelector("[data-testid='user-menu']");

    public DashboardPage(WebDriver driver) {
        super(driver);
    }

    @Override
    public void goTo() {
        driver.get(System.getenv("BASE_URL") + "/dashboard");
    }

    @Override
    public boolean isLoaded() {
        try {
            return driver.findElement(welcomeHeading).isDisplayed();
        } catch (org.openqa.selenium.NoSuchElementException e) {
            return false;
        }
    }

    public void clickNewItem() {
        wait.until(ExpectedConditions.elementToBeClickable(newItemButton)).click();
    }

    public void openUserMenu() {
        wait.until(ExpectedConditions.elementToBeClickable(userMenu)).click();
    }
}
```

---

### 3.3 Level 3 — Interface, Strategy Pattern, Polymorphism in Java

```java
// src/main/java/strategies/ILoginStrategy.java
// INTERFACE = ABSTRACTION + CONTRACT
// Tests depend on this, not on any specific implementation

package strategies;

import org.openqa.selenium.WebDriver;

public interface ILoginStrategy {
    void login(WebDriver driver, String email, String password);
}
```

```java
// src/main/java/strategies/FormLoginStrategy.java
package strategies;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

// POLYMORPHISM: implements the interface — one of many possible login strategies
public class FormLoginStrategy implements ILoginStrategy {

    @Override
    public void login(WebDriver driver, String email, String password) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        driver.get(System.getenv("BASE_URL") + "/login");
        wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("email"))).sendKeys(email);
        driver.findElement(By.id("password")).sendKeys(password);
        driver.findElement(By.cssSelector("button[type='submit']")).click();
    }
}
```

```java
// src/main/java/strategies/ApiTokenLoginStrategy.java
// Different strategy — same interface
// POLYMORPHISM: called the same way, behaves differently

package strategies;

import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;

public class ApiTokenLoginStrategy implements ILoginStrategy {

    @Override
    public void login(WebDriver driver, String email, String password) {
        // Get token via REST API — skip the form entirely
        Response response = RestAssured
            .given()
                .baseUri(System.getenv("API_BASE_URL"))
                .contentType("application/json")
                .body(String.format("{\"email\":\"%s\",\"password\":\"%s\"}", email, password))
            .when()
                .post("/api/auth/login")
            .then()
                .statusCode(200)
                .extract().response();

        String token = response.jsonPath().getString("token");

        // Set token in localStorage — no UI needed
        driver.get(System.getenv("BASE_URL"));
        ((JavascriptExecutor) driver).executeScript(
            "localStorage.setItem('auth_token', arguments[0]);", token
        );
        driver.get(System.getenv("BASE_URL") + "/dashboard");
    }
}
```

```java
// src/test/java/base/BaseTest.java
// DEPENDENCY INVERSION + AGGREGATION
// BaseTest AGGREGATES page objects — does not own them (they are assigned per test)
// Depends on ILoginStrategy abstraction, not a concrete class

package base;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Parameters;
import strategies.FormLoginStrategy;
import strategies.ApiTokenLoginStrategy;
import strategies.ILoginStrategy;

public abstract class BaseTest {

    // AGGREGATION: driver is shared, page objects are set up per test
    protected WebDriver driver;
    protected ILoginStrategy loginStrategy;  // depends on ABSTRACTION

    @BeforeMethod
    @Parameters("browser")
    public void setUp(String browser) {
        // POLYMORPHISM: same driver reference, different browser implementations
        this.driver = browser.equalsIgnoreCase("firefox")
            ? new FirefoxDriver()
            : new ChromeDriver();

        // DEPENDENCY INVERSION: inject strategy based on env config
        String strategyType = System.getenv().getOrDefault("LOGIN_STRATEGY", "form");
        this.loginStrategy = strategyType.equals("api")
            ? new ApiTokenLoginStrategy()
            : new FormLoginStrategy();
    }

    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

```java
// src/test/java/tests/LoginTest.java
// Test using INHERITANCE from BaseTest and POLYMORPHISM through loginStrategy

package tests;

import base.BaseTest;
import org.testng.Assert;
import org.testng.annotations.Test;
import pages.LoginPage;
import pages.DashboardPage;

public class LoginTest extends BaseTest {

    @Test
    public void loginWithValidCredentials() {
        // ASSOCIATION: test uses pages, does not own them
        LoginPage    loginPage    = new LoginPage(driver);
        DashboardPage dashPage    = new DashboardPage(driver);

        loginPage.goTo();

        // POLYMORPHISM in action:
        // loginStrategy could be FormLoginStrategy or ApiTokenLoginStrategy
        // The test does not know or care which one — calls .login() the same way
        loginStrategy.login(driver, "user@example.com", "Password123!");

        Assert.assertTrue(dashPage.isLoaded(), "Dashboard should be loaded after login");
        Assert.assertTrue(driver.getCurrentUrl().contains("dashboard"));
    }

    @Test
    public void invalidPasswordShowsError() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.goTo();
        loginPage.login("user@example.com", "wrongpassword");

        Assert.assertTrue(loginPage.isErrorDisplayed(), "Error message should appear");
        Assert.assertTrue(loginPage.getErrorMessage().contains("Invalid credentials"));
    }
}
```

---

## SECTION 4 — ARCHITECTURE

### 4.1 OOP Relationships Visualized

```
OOP IN TEST FRAMEWORK — RELATIONSHIP MAP

ASSOCIATION (uses — no ownership)
─────────────────────────────────
Test ─────uses──────► LoginPage
Test ─────uses──────► UserService
LoginPage ────uses──► WebDriver / Page  (passed in, not created)

COMPOSITION (owns — dies together)
───────────────────────────────────
TestFixture ──owns──► DatabaseConnection
BrowserContext ─owns─► Page
LoginPage ─────owns──► FormComponent    (created inside constructor)

AGGREGATION (has — survives independently)
────────────────────────────────────────
TestSuite ──has──► TestCase[]           (tests can run without the suite)
PageFactory ─has─► LoginPage, DashboardPage

INHERITANCE (is-a)
───────────────────
LoginPage ────────► BasePage
DashboardPage ────► BasePage
LoginTest ────────► BaseTest
FormLoginStrategy ─► ILoginStrategy
ApiTokenStrategy ──► ILoginStrategy

INTERFACE (contract)
─────────────────────
ILoginStrategy  ◄──── FormLoginStrategy
                ◄──── ApiTokenLoginStrategy
                ◄──── SSOLoginStrategy

IUserService    ◄──── UserService (real HTTP)
                ◄──── MockUserService (in-memory)

IReporter       ◄──── ConsoleReporter
                ◄──── SlackReporter
                ◄──── AllureReporter
```

### 4.2 Full Framework Architecture Using OOP

```
src/
├── pages/                          ← ENCAPSULATION (private locators)
│   ├── BasePage.ts                 ← ABSTRACTION (abstract class)
│   ├── LoginPage.ts                ← INHERITANCE (extends BasePage)
│   ├── DashboardPage.ts            ← INHERITANCE + POLYMORPHISM
│   └── components/
│       └── FormComponent.ts        ← COMPOSITION (owned by pages)
│
├── services/                       ← ABSTRACTION + SINGLE RESPONSIBILITY
│   ├── IUserService.ts             ← Interface (contract only)
│   ├── UserService.ts              ← Real implementation
│   └── MockUserService.ts          ← Mock (same interface, no HTTP)
│
├── strategies/                     ← STRATEGY PATTERN + POLYMORPHISM
│   ├── ILoginStrategy.ts           ← Interface
│   ├── FormLoginStrategy.ts
│   ├── ApiTokenLoginStrategy.ts
│   └── SSOLoginStrategy.ts
│
├── builders/                       ← BUILDER PATTERN
│   ├── UserBuilder.ts
│   └── PaymentBuilder.ts
│
├── factories/                      ← FACTORY PATTERN
│   └── PageFactory.ts
│
├── fixtures/                       ← DEPENDENCY INVERSION
│   ├── base.fixture.ts             ← Injects page, request, browser
│   └── auth.fixture.ts             ← Injects loginStrategy, userService
│
└── tests/                          ← Thin — delegates to pages/services
    ├── login.spec.ts
    └── dashboard.spec.ts
```

---

## SECTION 5 — INTERVIEW PREPARATION

### 5.1 Senior SDET Questions (with plain answers)

**Q1: What is the difference between an abstract class and an interface? When do you use each in a test framework?**

Use an abstract class when you have shared implementation that child classes should inherit — like `BasePage` which has `wait`, `driver`, `getTitle()`. Use an interface when you only want to define a contract with no shared implementation — like `ILoginStrategy`. The rule: if you need to share code, use abstract class. If you need to define a contract for unrelated classes to fulfill, use interface. In TypeScript both can define method signatures, but only abstract classes can hold fields and concrete methods.

**Q2: What is encapsulation and how does it protect your Page Object design?**

Encapsulation means hiding the internal details and only exposing what is needed. In a Page Object, locators are private. Tests cannot touch them directly. If a developer renames a CSS class, only the Page Object changes — tests are untouched because they called `loginPage.login()`, not `loginPage.emailInput.fill()`. Without encapsulation, every selector change breaks every test that used it.

**Q3: Explain polymorphism with a real example from your framework.**

In our framework, `ILoginStrategy` has a `login()` method. We have three implementations: `FormLoginStrategy`, `ApiTokenLoginStrategy`, and `SSOLoginStrategy`. The `BaseTest` fixture holds a reference of type `ILoginStrategy`. At runtime, depending on the environment variable `LOGIN_STRATEGY`, it gets the right object injected. The test calls `loginStrategy.login(...)` without knowing which strategy is running. On CI, we use the API token strategy (faster). In staging, we use form login. The test code is unchanged.

**Q4: What is the difference between association, aggregation, and composition? Give a testing example for each.**

Association: a test class uses a `LoginPage` object. The test did not create the browser — it was passed in. No ownership. The browser lives on after the test ends.

Aggregation: a `TestSuite` has a list of `TestCase` objects. If you delete the suite, the test cases still exist — they can be added to another suite. The test cases do not die with the suite.

Composition: a `BrowserContext` owns a `Page`. When the context is closed, the page is destroyed. The page cannot live without its context. Or: `LoginPage` creates a `FormComponent` inside its constructor. The component is owned by the page and dies when the page object goes out of scope.

**Q5: Why is "favour composition over inheritance" a practical rule in test framework design?**

Deep inheritance chains are brittle. If `BaseTest` changes, it breaks `LoginTest`, `CheckoutTest`, and `AdminTest`. Composition is more flexible: instead of `LoginTest extends BaseTest extends AuthenticatedBase extends UIBase`, you compose — `LoginTest` has a `LoginPage`, a `UserService`, and an `AuthHelper`. You can change any one without breaking the others. Composition also makes it easier to test the components in isolation.

**Q6: How do you apply the Single Responsibility Principle to a Page Object?**

A Page Object has one responsibility: represent the interactions with a single page or component. It does not assert results. It does not create test data. It does not manage the browser. If `LoginPage.login()` also sends a Slack notification on failure, that is two responsibilities. Split it: `LoginPage` handles the form, `TestReporter` handles notifications. When notification logic needs to change, you touch only `TestReporter`.

**Q7: What problem does the Strategy Pattern solve in test frameworks?**

Different environments need different login methods. On CI you want API token login (fast). In staging you want form login (realistic). In production smoke tests you want SSO login. Without Strategy Pattern, you put `if (env === 'ci')` blocks inside test code. With Strategy Pattern, you define `ILoginStrategy`, create three implementations, and inject the right one via a fixture. Test code has zero conditional logic.

**Q8: What is Dependency Inversion and how do Playwright fixtures implement it?**

Dependency Inversion says: high-level modules (tests) should not depend on low-level modules (real HTTP clients, real databases). Both should depend on abstractions. Playwright fixtures implement this perfectly. A test says "I need a `loggedInPage`" — it does not know that behind the scenes a `UserService` made an HTTP call to create the user and a `FormLoginStrategy` filled in the login form. The fixture is the composition root — it wires everything up. The test only sees the abstraction.

**Q9: What happens at runtime when you call a method on a child class through a parent reference?**

This is dynamic dispatch. In Java: `BasePage page = new LoginPage(driver); page.goTo();`. At compile time, Java knows `page` is a `BasePage`. At runtime, the JVM checks the actual object type (LoginPage) and calls its `goTo()`. This is why abstract methods work — the parent defines the method signature, the child provides the body, and the JVM routes to the right one at runtime. In TypeScript it works the same way via the prototype chain.

**Q10: You have a team where tests have assertions inside Page Objects. How do you fix it without rewriting all the tests?**

First, add a lint rule or code review checklist to stop new violations from being added. Then assess the damage: run a search for `expect(` inside `pages/` folder. Extract assertions to the test layer gradually — start with the most-changed page objects. Create a shared assertion helper class if the same assertion is needed in many tests. Do this over 2-3 sprints in parallel with new feature work — not a big-bang rewrite.

---

### 5.2 Staff SDET Questions

**Q1: Design the OOP structure for a test framework that must support web, API, and mobile tests sharing the same service layer.**

Answer: Create `IApiClient` interface with `get`, `post`, `put`, `delete`. Implement `RestApiClient` for HTTP tests and `MockApiClient` for unit tests of the framework itself. For pages: `BasePage` (web), `BaseMobilePage` (Appium), both implement `IPage` interface with `goto()` and `isLoaded()`. Service classes (`UserService`, `PaymentService`) take `IApiClient` — they work whether the test is web or mobile. The test fixture injects the right client. Web and mobile tests share service layer 100%.

**Q2: How do you handle a test helper class that has grown into a 2000-line God Class?**

Find all the different things it does. Group them. Extract each group into its own class. If `TestHelper` has login logic, database logic, file upload logic, and reporting logic — extract `AuthHelper`, `DbHelper`, `FileHelper`, `ReportHelper`. Update the fixture to compose them. Tests that used `helper.login()` now use `authHelper.login()`. Run tests after each extraction to verify no regressions. This is the Single Responsibility Principle applied as a refactoring exercise.

**Q3: A junior SDET proposes a 5-level inheritance chain for page objects. How do you respond?**

The chain is: `Object → BasePage → AuthenticatedPage → NavigationPage → SidebarPage → DashboardPage`. This is fragile. A change to `AuthenticatedPage` ripples through 4 levels. Instead, use composition: `DashboardPage` has a `Sidebar` component, uses an `AuthHelper`, and extends only `BasePage`. Two levels max. Explain it with the car analogy: a car does not inherit from Vehicle inherits from Machine inherits from PhysicalObject. A car has an engine, has wheels, has seats. The has-a relationships are easier to change than is-a chains.

---

### 5.3 Principal SDET Questions

**Q1: How do you enforce OOP standards across 15 teams each with their own Playwright frameworks?**

Create a shared `@company/test-base` npm package. It exports `BasePage`, `BaseFixture`, `ILoginStrategy`, `IApiClient`, `UserBuilder`, and common utilities. All teams install it. CI enforces the version via Renovate or Dependabot. Write an ESLint plugin that flags: assertions inside pages, public locators, and more than 2 levels of inheritance. Document the decisions in ADRs. Run a monthly internal tech talk on framework patterns. The package owns the OOP foundation; teams own the test logic.

**Q2: Your organisation is migrating from Selenium Java to Playwright TypeScript. How do OOP principles guide the migration?**

The OOP structure maps directly. Java `BasePage` becomes TypeScript `BasePage`. Java `ILoginStrategy` interface becomes TypeScript `ILoginStrategy` interface. Java `@BeforeMethod` setup becomes Playwright fixtures. The migration plan: (1) extract all service objects first — they have no UI dependency and can be ported as pure TypeScript classes. (2) Port page objects one by one — start with the least-used pages. (3) Port test classes last — they are thin wrappers around page objects once OOP is done right. The principles (SOLID, patterns) are language-independent.

---

### 5.4 Quick Reference — OOP Cheat Sheet for SDET Interviews

```
CONCEPT → TESTING EXAMPLE
────────────────────────────────────────────────────────────
Class            → LoginPage, UserService, PaymentBuilder
Object           → new LoginPage(page), new UserService(request)
Encapsulation    → private locators, public action methods
Abstraction      → BasePage (abstract), ILoginStrategy (interface)
Inheritance      → LoginPage extends BasePage
Polymorphism     → FormLogin / ApiLogin / SSO — same interface, different behavior
Association      → Test uses LoginPage (passed in)
Aggregation      → Suite has Tests (tests outlive suite)
Composition      → LoginPage owns FormComponent (component dies with page)

SOLID → TEST FRAMEWORK RULE
────────────────────────────────────────────────────────────
S  → One class per concern: Page, Service, Builder, Helper — never mix
O  → Add new reporter by creating new class, never edit existing ones
L  → AdminPage extends DashboardPage without breaking DashboardPage tests
I  → INavigable, ISearchable — not one giant IPage interface
D  → Tests depend on ILoginStrategy, not FormLoginStrategy directly

INTERVIEW RED FLAGS TO AVOID
────────────────────────────────────────────────────────────
× Confusing interface with abstract class
× Saying "polymorphism is just method overloading"
× Not knowing the difference between aggregation and composition
× Putting assertions inside page objects
× Public locators ("it is easier to access from tests")
× "We use inheritance everywhere because it is reuse"
× Cannot explain why composition is preferred over deep inheritance
× Never using interfaces in a test framework
```

---

### 5.5 System Design Interview Question

**Question:** "Design an OOP-based test automation framework for a FinTech company. The app has a React web UI, a REST API, and a mobile app. 50 engineers will use this framework. Walk me through the class hierarchy, relationships, and design patterns."

**What to say:**

Start with the layers. Layer 1: Abstractions. Define `IPage`, `IApiClient`, `ILoginStrategy`. No implementation yet — just contracts. Layer 2: Base classes. `BasePage` (web, abstract), `BaseMobilePage` (Appium, abstract), `BaseApiClient` (concrete). Layer 3: Concrete pages, services, strategies. Layer 4: Fixtures that compose everything and inject into tests. Layer 5: Tests that are thin — call methods, assert results, nothing else.

Patterns to mention: Page Object (encapsulation), Service Object (single responsibility), Factory (page creation), Builder (test data), Strategy (login/environment), Singleton (browser pool for parallel tests), Template Method (BaseTest defines flow, children implement steps).

OOP concepts to highlight: Inheritance for shared browser setup. Composition for page components. Polymorphism for cross-browser and cross-environment. Dependency Inversion so tests never import concrete service implementations.

What separates a good answer from a great one: The great answer talks about what happens when things go wrong. "If we used inheritance deeply, a change to BaseTest breaks 200 test files. So we use composition in fixtures. Each fixture composes exactly what that group of tests needs. A login test fixture composes AuthHelper + LoginPage. A payment test fixture composes AuthHelper + PaymentService + PaymentPage. No cross-contamination."

---

## SECTION 6 — HANDS-ON PROJECT — Day 1 Task

**Your task for today:**

1. Create a folder: `oop-day1/`
2. Inside it, create these files from scratch — no copy-paste of the full classes above, type them out:
   - `BasePage.ts` — abstract class with `goto()` abstract, `getTitle()` concrete
   - `LoginPage.ts` — extends BasePage, private locators, public `login()` method
   - `DashboardPage.ts` — extends BasePage, different `goto()` and `isLoaded()`
   - `ILoginStrategy.ts` — interface with one method
   - `FormLoginStrategy.ts` — implements interface
   - `tests/login.spec.ts` — use LoginPage, assert via expect in the test (not in the page)

3. Run: `npx playwright test` — all tests must pass

4. Answer these questions in a `notes.md` file:
   - Why are locators private in LoginPage?
   - What is the difference between FormLoginStrategy and ApiTokenLoginStrategy at the test level?
   - If you change the login button selector, how many files need to change?
   - Is the relationship between `LoginPage` and `FormComponent` composition or aggregation?

**Expected output:** 4 passing tests, 6 TypeScript files, `notes.md` with your answers.

**If you finish early — extension challenge:** Add a `MockApiClient` that implements `IApiClient` and returns hardcoded users. Write a test that uses it via a fixture. Verify the test passes without any real HTTP calls.
