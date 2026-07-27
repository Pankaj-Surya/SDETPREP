# Playwright — Design Patterns in Test Automation — Notes with Real-Time Examples

Simple language, real test-automation scenarios, all in TypeScript. These aren't Playwright APIs — they're **architecture patterns** commonly used to keep large Playwright suites maintainable.

---

## 1. Page Object Model (POM)

**In simple words:** One class per page/screen, holding its locators and actions. Tests read like plain English instead of being full of raw locators.

**Real-time example — a login page used across many tests:**

```ts
// pages/LoginPage.ts
export class LoginPage {
  constructor(private page: Page) {}

  private email = this.page.getByLabel('Email');
  private password = this.page.getByLabel('Password');
  private submit = this.page.getByRole('button', { name: 'Log In' });

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.email.fill(email);
    await this.password.fill(password);
    await this.submit.click();
  }
}
```

```ts
test('valid login redirects to dashboard', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@example.com', 'Secret123');
  await expect(page).toHaveURL(/dashboard/);
});
```

---

## 2. Service Object Model

**In simple words:** Same idea as POM, but for **APIs** instead of pages — one class per backend service/resource, wrapping `request.get/post/...` calls.

**Real-time example — an `OrdersService` used to set up test data via API instead of the UI:**

```ts
// services/OrdersService.ts
export class OrdersService {
  constructor(private request: APIRequestContext) {}

  async createOrder(item: string, qty: number) {
    const res = await this.request.post('/api/orders', { data: { item, qty } });
    return res.json();
  }

  async deleteOrder(id: string) {
    await this.request.delete(`/api/orders/${id}`);
  }
}
```

```ts
test('newly created order appears in the list', async ({ page, request }) => {
  const orders = new OrdersService(request);
  const order = await orders.createOrder('Laptop', 1);

  await page.goto('/orders');
  await expect(page.getByText('Laptop')).toBeVisible();

  await orders.deleteOrder(order.id); // cleanup
});
```

---

## 3. Component Object Model

**In simple words:** Instead of one giant class per page, break the page into **reusable UI component classes** (a nav bar, a data table, a modal) that show up on many pages — avoids duplicating the same locators everywhere.

**Real-time example — a reusable `DataTable` component used on both the "Orders" and "Customers" pages:**

```ts
// components/DataTable.ts
export class DataTable {
  constructor(private root: Locator) {}

  row(name: string) {
    return this.root.getByRole('row', { name });
  }

  async sortBy(column: string) {
    await this.root.getByRole('columnheader', { name: column }).click();
  }
}
```

```ts
// pages/OrdersPage.ts
export class OrdersPage {
  readonly table: DataTable;
  constructor(private page: Page) {
    this.table = new DataTable(page.getByTestId('orders-table'));
  }
}
```

```ts
test('orders can be sorted by date', async ({ page }) => {
  const ordersPage = new OrdersPage(page);
  await ordersPage.table.sortBy('Date');
  await expect(ordersPage.table.row('Order #1023')).toBeVisible();
});
```

---

## 4. Fixture-First architecture

**In simple words:** Instead of `new`-ing up page objects/services inside every test, expose them as **fixtures** — tests just declare what they need in the function signature.

**Real-time example — combining POM + Service Object Model, both delivered as fixtures:**

```ts
export const test = base.extend<{ loginPage: LoginPage; ordersService: OrdersService }>({
  loginPage: async ({ page }, use) => use(new LoginPage(page)),
  ordersService: async ({ request }, use) => use(new OrdersService(request)),
});
```

```ts
test('order created via API shows on dashboard', async ({ page, loginPage, ordersService }) => {
  await loginPage.goto();
  await loginPage.login('user@example.com', 'Secret123');
  await ordersService.createOrder('Laptop', 1);

  await page.goto('/orders');
  await expect(page.getByText('Laptop')).toBeVisible();
});
```

This is the direction Playwright itself nudges you toward — fixtures compose naturally, and teardown is automatic.

---

## 5. Singleton (browser manager)

**In simple words:** Ensure only **one instance** of an expensive resource (like a browser connection to a remote Selenium-Grid-style service, or a shared config loader) exists across the whole run.

**Real-time example — a singleton wrapper around a remote browser connection used by many test files:**

```ts
// browserManager.ts
class BrowserManager {
  private static instance: Browser;

  static async getBrowser(): Promise<Browser> {
    if (!BrowserManager.instance) {
      BrowserManager.instance = await chromium.connect(process.env.REMOTE_BROWSER_WS!);
    }
    return BrowserManager.instance;
  }
}

export default BrowserManager;
```

```ts
const browser = await BrowserManager.getBrowser(); // same instance reused everywhere
const page = await (await browser.newContext()).newPage();
```

> Note: Playwright's own `browser` fixture is already worker-scoped (effectively a per-worker singleton) — you'd reach for a manual singleton mainly for something Playwright doesn't manage itself, like a shared remote grid connection or a config/logger instance.

---

## 6. Factory (page / data factory)

**In simple words:** A function/class whose only job is to **produce ready-made objects** — a page object for whichever page you need, or realistic fake test data — so tests don't hardcode values or repeat `new Page(...)` logic.

**Real-time example 1 — a page factory that returns the right page object based on a name:**

```ts
function createPage(name: 'login' | 'orders' | 'checkout', page: Page) {
  switch (name) {
    case 'login': return new LoginPage(page);
    case 'orders': return new OrdersPage(page);
    case 'checkout': return new CheckoutPage(page);
  }
}
```

**Real-time example 2 — a data factory generating realistic, unique test users (very common in real suites, often with a library like `@faker-js/faker`):**

```ts
import { faker } from '@faker-js/faker';

function createTestUser(overrides: Partial<User> = {}): User {
  return {
    name: faker.person.fullName(),
    email: faker.internet.email(),
    password: 'Secret123!',
    ...overrides,
  };
}

test('signup with a freshly generated user', async ({ page }) => {
  const user = createTestUser();
  await page.goto('/signup');
  await page.getByLabel('Name').fill(user.name);
  await page.getByLabel('Email').fill(user.email);
});
```

---

## 7. Builder (test data builder)

**In simple words:** Construct a complex object **step by step** with a fluent, readable chain — useful when a test only needs to override a couple of fields out of many.

**Real-time example — building an order with only the fields that matter for a given test, defaults for the rest:**

```ts
class OrderBuilder {
  private order = { item: 'Default Item', qty: 1, giftWrap: false, coupon: null as string | null };

  withItem(item: string) { this.order.item = item; return this; }
  withQty(qty: number) { this.order.qty = qty; return this; }
  withCoupon(code: string) { this.order.coupon = code; return this; }
  build() { return { ...this.order }; }
}
```

```ts
test('order with a coupon applies a discount', async ({ request }) => {
  const order = new OrderBuilder().withItem('Laptop').withCoupon('SAVE10').build();
  const res = await request.post('/api/orders', { data: order });
  expect((await res.json()).discount).toBeGreaterThan(0);
});
```

---

## 8. Strategy (login strategy)

**In simple words:** Different ways to achieve the same goal (logging in), swappable behind one interface — e.g., login via UI form, via API token, or via SSO — without changing test code.

**Real-time example:**

```ts
interface LoginStrategy {
  login(page: Page, request: APIRequestContext): Promise<void>;
}

class UiLoginStrategy implements LoginStrategy {
  async login(page: Page) {
    await page.goto('/login');
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('Secret123');
    await page.getByRole('button', { name: 'Log In' }).click();
  }
}

class ApiLoginStrategy implements LoginStrategy {
  async login(page: Page, request: APIRequestContext) {
    const res = await request.post('/auth/login', { data: { email: 'user@example.com', password: 'Secret123' } });
    const { token } = await res.json();
    await page.context().addCookies([{ name: 'session', value: token, domain: 'app.example.com', path: '/' }]);
  }
}
```

```ts
async function loginAs(strategy: LoginStrategy, page: Page, request: APIRequestContext) {
  await strategy.login(page, request);
}

test('dashboard loads after API login (fast path)', async ({ page, request }) => {
  await loginAs(new ApiLoginStrategy(), page, request);
  await page.goto('/dashboard');
  await expect(page.getByText('Welcome back')).toBeVisible();
});
```

---

## 9. Facade (app facade)

**In simple words:** A single, simplified entry point that hides a bunch of underlying page objects/services behind one clean interface — the test doesn't need to know how many classes are involved underneath.

**Real-time example:**

```ts
class ShopApp {
  readonly login: LoginPage;
  readonly cart: CartPage;
  readonly checkout: CheckoutPage;

  constructor(private page: Page) {
    this.login = new LoginPage(page);
    this.cart = new CartPage(page);
    this.checkout = new CheckoutPage(page);
  }

  async purchaseItem(email: string, password: string, item: string) {
    await this.login.goto();
    await this.login.login(email, password);
    await this.cart.addItem(item);
    await this.checkout.completeOrder();
  }
}
```

```ts
test('user can complete a purchase end-to-end', async ({ page }) => {
  const app = new ShopApp(page);
  await app.purchaseItem('user@example.com', 'Secret123', 'Laptop');
  await expect(page.getByText('Order Confirmed')).toBeVisible();
});
```

---

## 10. Repository (locator repo)

**In simple words:** Centralize all locators for a page/feature in one place (a "repository" of selectors) — separate from the action methods — so if the UI's structure changes, you fix locators in exactly one file.

**Real-time example:**

```ts
// locators/OrdersLocators.ts
export class OrdersLocators {
  constructor(private page: Page) {}
  get searchBox() { return this.page.getByPlaceholder('Search orders'); }
  get orderRow() { return (name: string) => this.page.getByRole('row', { name }); }
  get exportButton() { return this.page.getByRole('button', { name: 'Export' }); }
}
```

```ts
// pages/OrdersPage.ts — actions use the locator repo, kept separate from selector definitions
export class OrdersPage {
  readonly locators: OrdersLocators;
  constructor(private page: Page) {
    this.locators = new OrdersLocators(page);
  }

  async searchFor(term: string) {
    await this.locators.searchBox.fill(term);
  }
}
```

---

## Quick summary

| Pattern | One-line real-world use |
|---|---|
| Page Object Model | One class per page, tests read like plain English |
| Service Object Model | One class per API resource, used to set up/verify data fast |
| Component Object Model | Reusable widget classes (table, nav bar) shared across many pages |
| Fixture-First architecture | Deliver page objects/services as fixtures instead of manual `new` |
| Singleton | One shared expensive resource (remote browser connection, config) |
| Factory | Produce ready-made page objects or realistic fake test data |
| Builder | Fluent, step-by-step construction of complex test data |
| Strategy | Swap login methods (UI/API/SSO) behind one interface |
| Facade | One simple entry point hiding many page objects underneath |
| Repository | All locators for a feature centralized in one file |
