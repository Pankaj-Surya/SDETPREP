# SOLID Principles in Playwright Test Automation — Notes with Real-Time Examples

Simple language, real test-automation scenarios showing each principle applied to a Playwright suite, all in TypeScript.

---

## 1. Single Responsibility Principle (SRP)

**In simple words:** A class should have **one reason to change**. In test automation: a page object should only know about that page's UI — not about API calls, not about test data generation, not about assertions.

**Real-time example — the problem, then the fix:**

```ts
// ❌ Violates SRP: LoginPage does UI actions AND API setup AND data generation
class LoginPage {
  async login(email: string, password: string) { /* ui actions */ }
  async createUserViaApi(request: APIRequestContext) { /* api call — wrong place */ }
  generateRandomEmail() { /* data generation — wrong place */ }
}
```

```ts
// ✅ Each class has exactly one job
class LoginPage {
  constructor(private page: Page) {}
  async login(email: string, password: string) { /* only UI actions */ }
}

class UsersService {
  constructor(private request: APIRequestContext) {}
  async createUser(email: string) { /* only API calls */ }
}

function generateRandomEmail() { /* only data generation */ }
```

Now, if the login **form** changes, only `LoginPage` needs editing — the API service and data generator are untouched.

---

## 2. Open/Closed Principle (OCP)

**In simple words:** Code should be **open for extension, but closed for modification** — you should be able to add new behavior (a new browser, a new login method, a new report format) without editing existing, already-tested code.

**Real-time example — adding a new login method without touching existing login code:**

```ts
interface LoginStrategy {
  login(page: Page): Promise<void>;
}

class UiLoginStrategy implements LoginStrategy {
  async login(page: Page) { /* existing UI login, untouched */ }
}

// Adding Google SSO later doesn't require changing UiLoginStrategy at all —
// we just add a new class that implements the same interface.
class GoogleSsoLoginStrategy implements LoginStrategy {
  async login(page: Page) {
    await page.getByRole('button', { name: 'Sign in with Google' }).click();
    // ... handle Google popup flow
  }
}
```

```ts
async function runLogin(strategy: LoginStrategy, page: Page) {
  await strategy.login(page); // works with ANY current or future strategy, unmodified
}
```

---

## 3. Liskov Substitution Principle (LSP)

**In simple words:** A subclass should be usable **anywhere its parent class is expected**, without breaking things. In test automation, this usually shows up with page objects that extend a common base — a subclass shouldn't secretly require different setup or throw where the base wouldn't.

**Real-time example — the problem, then the fix:**

```ts
// ❌ Violates LSP: GuestCheckoutPage throws if you call a method the base class promises works
class CheckoutPage {
  async applyCoupon(code: string) { /* works for logged-in users */ }
}

class GuestCheckoutPage extends CheckoutPage {
  async applyCoupon(code: string) {
    throw new Error('Guests cannot apply coupons'); // breaks the substitution promise
  }
}
```

```ts
// ✅ Model it honestly instead — don't force a subtype into a shape it can't fulfill
interface CheckoutPage {
  completeOrder(): Promise<void>;
}

interface CouponCapableCheckout extends CheckoutPage {
  applyCoupon(code: string): Promise<void>;
}

class RegisteredUserCheckoutPage implements CouponCapableCheckout {
  async completeOrder() { /* ... */ }
  async applyCoupon(code: string) { /* ... */ }
}

class GuestCheckoutPage implements CheckoutPage {
  async completeOrder() { /* ... */ } // no applyCoupon promised, none required
}
```

Any test written against `CheckoutPage` can now safely use **either** page object interchangeably — nothing unexpectedly throws.

---

## 4. Interface Segregation Principle (ISP)

**In simple words:** Don't force a class to depend on methods it doesn't use — prefer several small, focused interfaces over one giant one.

**Real-time example — the problem, then the fix:**

```ts
// ❌ One bloated interface forces every page object to implement things it doesn't need
interface PageActions {
  login(): Promise<void>;
  addToCart(): Promise<void>;
  applyCoupon(): Promise<void>;
  exportReport(): Promise<void>;
}
```

```ts
// ✅ Small, focused interfaces — a page object only implements what's relevant to it
interface Loginable { login(email: string, password: string): Promise<void>; }
interface CartActions { addToCart(item: string): Promise<void>; }
interface Exportable { exportReport(): Promise<void>; }

class LoginPage implements Loginable {
  async login(email: string, password: string) { /* ... */ }
}

class ProductPage implements CartActions {
  async addToCart(item: string) { /* ... */ }
}

class ReportsPage implements Exportable {
  async exportReport() { /* ... */ }
}
```

`ReportsPage` is never forced to pretend it knows how to `login()` or `addToCart()`.

---

## 5. Dependency Inversion Principle (DIP)

**In simple words:** High-level test logic shouldn't depend directly on low-level implementation details — both should depend on an **abstraction** (an interface). In Playwright terms: tests/fixtures should depend on an interface like `LoginStrategy`, not directly on a concrete `UiLoginStrategy` class.

**Real-time example — the problem, then the fix:**

```ts
// ❌ Violates DIP: the test setup is hardwired to one concrete implementation
class TestSetup {
  async login(page: Page) {
    const strategy = new UiLoginStrategy(); // hardcoded — can't swap without editing this file
    await strategy.login(page);
  }
}
```

```ts
// ✅ Depend on the interface, inject the concrete implementation from outside
interface LoginStrategy {
  login(page: Page): Promise<void>;
}

class TestSetup {
  constructor(private strategy: LoginStrategy) {} // depends on abstraction only
  async login(page: Page) {
    await this.strategy.login(page);
  }
}

// usage — swap strategies freely without touching TestSetup at all
const uiSetup = new TestSetup(new UiLoginStrategy());
const apiSetup = new TestSetup(new ApiLoginStrategy());
```

**Playwright's own fixture system is DIP in action:** a test declares `{ loginPage }` (an abstraction it needs), and Playwright injects whichever concrete implementation the fixture file provides — the test never constructs it directly.

```ts
test('checkout works for a logged-in user', async ({ loginPage }) => {
  // "loginPage" — the test only knows the shape it needs, not how it's built
});
```

---

## Quick summary

| Principle | One-line real-world application |
|---|---|
| Single Responsibility | Page objects only handle UI; services only handle API; keep data generation separate |
| Open/Closed | Add a new login method or reporter without editing existing, tested code |
| Liskov Substitution | Any page object implementing an interface should be swappable without surprises |
| Interface Segregation | Small, focused interfaces (`Loginable`, `CartActions`) instead of one giant one |
| Dependency Inversion | Tests/fixtures depend on interfaces (`LoginStrategy`), concrete classes are injected |
