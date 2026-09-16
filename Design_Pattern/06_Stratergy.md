Real-Time Usecase: Playwright TypeScript with Strategy Pattern & DIPA common automation challenge is handling different authentication strategies (e.g., standard login via UI, SSO via Okta, or API session injection) without rewriting your test scripts.By applying the Dependency Inversion Principle (DIP) and the Strategy Pattern, your high-level test code remains completely decoupled from the low-level login mechanics.1. Define the Abstraction (Interface)This satisfies DIP by creating an interface that both the test script and the concrete classes depend on.

```
// authStrategy.ts
import { Page } from '@playwright/test';

export interface AuthStrategy {
  login(page: Page): Promise<void>;
}
```

```
// uiLoginStrategy.ts
import { Page } from '@playwright/test';
import { AuthStrategy } from './authStrategy';

export class UiLoginStrategy implements AuthStrategy {
  async login(page: Page): Promise<void> {
    await page.goto('/login');
    await page.fill('#username', 'test_user');
    await page.fill('#password', 'secure_password');
    await page.click('#submit-btn');
  }
}

// apiLoginStrategy.ts
import { Page } from '@playwright/test';
import { AuthStrategy } from './authStrategy';

export class ApiLoginStrategy implements AuthStrategy {
  async login(page: Page): Promise<void> {
    // Injecting storage state directly to bypass UI login
    await page.context().addCookies([{
      name: 'session_token',
      value: 'mocked_api_token_12345',
      domain: 'example.com',
      path: '/'
    }]);
    await page.goto('/dashboard');
  }
}


```


```
// loginContext.ts
import { Page } from '@playwright/test';
import { AuthStrategy } from './authStrategy';

export class LoginContext {
  private strategy: AuthStrategy;

  // Dependency Injection happens here
  constructor(strategy: AuthStrategy) {
    this.strategy = strategy;
  }

  async executeLogin(page: Page): Promise<void> {
    await this.strategy.login(page);
  }
}

```

```
// e2e-dashboard.spec.ts
import { test, expect } from '@playwright/test';
import { LoginContext } from './loginContext';
import { UiLoginStrategy } from './uiLoginStrategy';
import { ApiLoginStrategy } from './apiLoginStrategy';

test.describe('Dashboard Tests with Dynamic Login', () => {

  test('Should load dashboard fast using API Login Injection', async ({ page }) => {
    // Injecting the API strategy directly
    const authContext = new LoginContext(new ApiLoginStrategy());
    await authContext.executeLogin(page);

    await expect(page.locator('#dashboard-header')).toBeVisible();
  });

  test('Should verify full UI Login flow', async ({ page }) => {
    // Swapping seamlessly to the UI strategy 
    const authContext = new LoginContext(new UiLoginStrategy());
    await authContext.executeLogin(page);

    await expect(page.locator('#dashboard-header')).toBeVisible();
  });
});
```


Use Case 1: Multi-User Roles (RBAC Testing)The Problem: You have a dashboard test. If an Admin logs in, they see a "Delete User" button. If a Viewer logs in, that button must be hidden. Instead of writing two separate tests with repetitive login code, you use a Strategy pattern to change who logs in.1. Define the Strategy Interfacetypescriptimport { Page } from '@playwright/test';

export interface UserRoleStrategy {
  login(page: Page): Promise<void>;
  shouldSeeDeleteButton(): boolean;
}
Use code with caution.2. Implement Concrete Rolestypescriptimport { Page } from '@playwright/test';
import { UserRoleStrategy } from './userRoleStrategy';

export class AdminRole implements UserRoleStrategy {
  async login(page: Page): Promise<void> {
    await page.goto('/login');
    await page.fill('#username', 'admin_user');
    await page.fill('#password', 'admin_pass');
    await page.click('#login-btn');
  }
  shouldSeeDeleteButton(): boolean { return true; }
}

export class ViewerRole implements UserRoleStrategy {
  async login(page: Page): Promise<void> {
    await page.goto('/login');
    await page.fill('#username', 'viewer_user');
    await page.fill('#password', 'viewer_pass');
    await page.click('#login-btn');
  }
  shouldSeeDeleteButton(): boolean { return false; }
}
Use code with caution.3. Create the Clean Test Filetypescriptimport { test, expect } from '@playwright/test';
import { UserRoleStrategy } from './userRoleStrategy';
import { AdminRole } from './adminRole';
import { ViewerRole } from './viewerRole';

const roles: UserRoleStrategy[] = [new AdminRole(), new ViewerRole()];

for (const role of roles) {
  test(`Dashboard verification for role: ${role.constructor.name}`, async ({ page }) => {
    // 1. Execute the specific login strategy
    await role.login(page);

    // 2. Run the assertion based on the strategy's rule
    const deleteButton = page.locator('#delete-user-btn');
    
    if (role.shouldSeeDeleteButton()) {
      await expect(deleteButton).toBeVisible();
    } else {
      await expect(deleteButton).toBeHidden();
    }
  });
}
Use code with caution.Use Case 2: Multi-Payment Gateways (E-Commerce Checkout)The Problem: Your e-commerce checkout flow is identical until the very last step. At the end, a user can choose Credit Card (requires typing into input fields) or PayPal (requires clicking an external checkout modal).1. Define the Strategy Interfacetypescriptimport { Page } from '@playwright/test';

export interface PaymentStrategy {
  pay(page: Page): Promise<void>;
}
Use code with caution.2. Implement Concrete Paymentstypescriptimport { Page } from '@playwright/test';
import { PaymentStrategy } from './paymentStrategy';

export class CreditCardPayment implements PaymentStrategy {
  async pay(page: Page): Promise<void> {
    await page.fill('#card-number', '4111222233334444');
    await page.fill('#card-expiry', '12/29');
    await page.fill('#card-cvv', '123');
    await page.click('#submit-payment');
  }
}

export class PayPalPayment implements PaymentStrategy {
  async pay(page: Page): Promise<void> {
    await page.click('#paypal-express-btn');
    // Mimic interacting with an external iframe/popup window
    await page.fill('#paypal-email', 'buyer@example.com');
    await page.click('#paypal-login');
  }
}
Use code with caution.3. Create the Checkout Runner Class (The Context)typescriptimport { Page } from '@playwright/test';
import { PaymentStrategy } from './paymentStrategy';

export class CheckoutRunner {
  private paymentStrategy: PaymentStrategy;

  constructor(paymentStrategy: PaymentStrategy) {
    this.paymentStrategy = paymentStrategy;
  }

  async completePurchase(page: Page): Promise<void> {
    // Standard immutable steps
    await page.goto('/cart');
    await page.click('#checkout-btn');
    await page.fill('#shipping-address', '123 Main St');
    
    // Dynamic interchangeable step
    await this.paymentStrategy.pay(page);
    
    // Final verification step
    await page.waitForURL('/order-confirmation');
  }
}
Use code with caution.4. The Clean Test Scripttypescriptimport { test } from '@playwright/test';
import { CheckoutRunner } from './checkoutRunner';
import { CreditCardPayment } from './creditCardPayment';

test('Successful purchase using Credit Card', async ({ page }) => {
  // Inject the specific payment strategy
  const checkout = new CheckoutRunner(new CreditCardPayment());
  await checkout.completePurchase(page);
});
Use code with caution.Use Case 3: Test Data Providers (Environment/Mock vs. Real API)The Problem: You need a dynamic API user token for your tests. In the Staging environment, you want to generate it by hitting a real database query. In the Local environment, you want to return a quick, hardcoded mock token to save time.1. Define the Strategy Interfacetypescriptexport interface TokenProviderStrategy {
  getToken(): Promise<string>;
}
Use code with caution.2. Implement Concrete Providerstypescriptimport { TokenProviderStrategy } from './tokenProviderStrategy';

export class MockTokenProvider implements TokenProviderStrategy {
  async getToken(): Promise<string> {
    return 'local_mocked_token_123';
  }
}

export class DatabaseTokenProvider implements TokenProviderStrategy {
  async getToken(): Promise<string> {
    // Imagine real DB querying code here
    const realTokenFromDb = "db_prod_secure_token_abcXYZ"; 
    return realTokenFromDb;
  }
}
Use code with caution.3. Consume in your Playwright Test via Env Configtypescriptimport { test, expect } from '@playwright/test';
import { TokenProviderStrategy } from './tokenProviderStrategy';
import { MockTokenProvider } from './mockTokenProvider';
import { DatabaseTokenProvider } from './databaseTokenProvider';

test('API integration check using environment-based strategy', async ({ request }) => {
  // Pick strategy dynamically based on process environment variables
  let tokenProvider: TokenProviderStrategy;
  
  if (process.env.ENV === 'STAGING') {
    tokenProvider = new DatabaseTokenProvider();
  } else {
    tokenProvider = new MockTokenProvider();
  }

  const activeToken = await tokenProvider.getToken();

  // Send request using the strategy-provided token
  const response = await request.get('/api/v1/user/profile', {
    headers: { 'Authorization': `Bearer ${activeToken}` }
  });

  expect(response.status()).toBe(200);
});
Use code with caution.💡 Why this makes your code highly maintainable:If tomorrow your manager says "We are adding Apple Pay as a payment method" or "We are adding an Editor user role", you do not touch your existing tests or runner frameworks. You simply create one single new class file that implements the interface. The rest of your framework remains completely unaffected.If you are ready to implement this, let me know:Would you like to see how to pair this with a Factory Pattern to automatically select these strategies using custom string inputs?Do you want to see how to bundle these strategy objects directly into Playwright Custom Fixtures so they load automatically inside test('', async ({ adminRole }) => { ... })?

