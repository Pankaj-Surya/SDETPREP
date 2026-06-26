## Facade Pattern

The **Facade pattern** in test automation provides a simplified, high-level interface that hides the complex interactions of multiple underlying classes (like Page Objects or API clients). It allows test scripts to execute entire workflows with a single method call, significantly improving test readability and maintainability.

### How the Facade Pattern Works
In test automation, tests often require navigating through multiple screens, inputting data, and asserting states. Instead of calling these individual page methods inside your test, a Facade class bundles them together into a single business-level action (e.g., `checkoutProcess.completeOrder(user)`). 

* **Facade:** Exposes a simple, unified method for a complex workflow.
* **Subsystems (Page Objects/API):** Contains the actual code interacting with the UI or services. 
* **Test Case:** Calls the Facade method, unaware of the underlying pages being navigated.

---

### Benefits for Test Automation
* **Reduces Code Duplication:** Common user flows are defined once in the Facade and reused across multiple test cases.
* **Simplifies Maintenance:** If the UI changes and a button sequence is updated, you only need to modify the Facade rather than dozens of test files.
* **Improves Readability:** Tests read more like business requirements (e.g., `bookingFacade.bookAFlight(...)`) rather than a sequence of mechanical UI clicks.
* **Encapsulates State:** Manages complex setup steps, like executing multiple API calls to build test data before a UI test runs.

---

### Example Scenario: E-Commerce Checkout
Instead of writing extensive navigation logic directly inside your test cases, you can abstract it using the Facade pattern.

**Traditional Approach (Without Facade):**
```java
cartPage.viewCart();
checkoutPage.enterShippingInfo(details);
checkoutPage.enterPaymentInfo(card);
checkoutPage.clickPlaceOrder();
confirmationPage.verifyOrderSuccess();
```

**Facade Approach:**
```java
// 1. Define the Facade
public class CheckoutFacade {
    private CartPage cartPage;
    private CheckoutPage checkoutPage;
    private ConfirmationPage confirmationPage;
    
    public CheckoutFacade(WebDriver driver) {
         // Initialize pages
    }

    public void completeFullPurchase(CustomerDetails details, PaymentInfo card) {
        cartPage.viewCart();
        checkoutPage.enterShippingInfo(details);
        checkoutPage.enterPaymentInfo(card);
        checkoutPage.clickPlaceOrder();
        confirmationPage.verifyOrderSuccess();
    }
}

// 2. Execute within the Test Class
CheckoutFacade checkout = new CheckoutFacade(driver);
checkout.completeFullPurchase(customerDetails, paymentInfo);
```


The **Facade pattern** in test automation provides a simplified, high-level interface that hides the complex interactions of multiple underlying classes (like Page Objects or API clients). It allows test scripts to execute entire workflows with a single method call, significantly improving test readability and maintainability.

### How the Facade Pattern Works
In test automation, tests often require navigating through multiple screens, inputting data, and asserting states. Instead of calling these individual page methods inside your test, a Facade class bundles them together into a single business-level action (e.g., `checkoutProcess.completeOrder(user)`). 

* **Facade:** Exposes a simple, unified method for a complex workflow.
* **Subsystems (Page Objects/API):** Contains the actual code interacting with the UI or services. 
* **Test Case:** Calls the Facade method, unaware of the underlying pages being navigated.

---

### Benefits for Test Automation
* **Reduces Code Duplication:** Common user flows are defined once in the Facade and reused across multiple test cases.
* **Simplifies Maintenance:** If the UI changes and a button sequence is updated, you only need to modify the Facade rather than dozens of test files.
* **Improves Readability:** Tests read more like business requirements (e.g., `bookingFacade.bookAFlight(...)`) rather than a sequence of mechanical UI clicks.
* **Encapsulates State:** Manages complex setup steps, like executing multiple API calls to build test data before a UI test runs.

---

### Example Scenario: E-Commerce Checkout

#### 1. Java / Selenium Implementation

**Traditional Approach (Without Facade):**
```java
cartPage.viewCart();
checkoutPage.enterShippingInfo(details);
checkoutPage.enterPaymentInfo(card);
checkoutPage.clickPlaceOrder();
confirmationPage.verifyOrderSuccess();
```

**Facade Approach:**
```java
// Define the Facade
public class CheckoutFacade {
    private CartPage cartPage;
    private CheckoutPage checkoutPage;
    private ConfirmationPage confirmationPage;
    
    public CheckoutFacade(WebDriver driver) {
         // Initialize pages
    }

    public void completeFullPurchase(CustomerDetails details, PaymentInfo card) {
        cartPage.viewCart();
        checkoutPage.enterShippingInfo(details);
        checkoutPage.enterPaymentInfo(card);
        checkoutPage.clickPlaceOrder();
        confirmationPage.verifyOrderSuccess();
    }
}

// Execute within the Test Class
CheckoutFacade checkout = new CheckoutFacade(driver);
checkout.completeFullPurchase(customerDetails, paymentInfo);
```

#### 2. Playwright / TypeScript Implementation

**Define the Facade (`CheckoutFacade.ts`):**
```typescript
import { Page } from '@playwright/test';
import { CartPage } from './CartPage';
import { CheckoutPage } from './CheckoutPage';
import { ConfirmationPage } from './ConfirmationPage';

export class CheckoutFacade {
    private cartPage: CartPage;
    private checkoutPage: CheckoutPage;
    private confirmationPage: ConfirmationPage;

    constructor(page: Page) {
        this.cartPage = new CartPage(page);
        this.checkoutPage = new CheckoutPage(page);
        this.confirmationPage = new ConfirmationPage(page);
    }

    /**
     * High-level business flow combining multiple page object operations
     */
    async completeFullPurchase(details: object, card: object): Promise<void> {
        await this.cartPage.viewCart();
        await this.checkoutPage.enterShippingInfo(details);
        await this.checkoutPage.enterPaymentInfo(card);
        await this.checkoutPage.clickPlaceOrder();
        await this.confirmationPage.verifyOrderSuccess();
    }
}
```

**Execute within the Test File (`checkout.spec.ts`):**
```typescript
import { test } from '@playwright/test';
import { CheckoutFacade } from '../facades/CheckoutFacade';

test('User can successfully purchase an item', async ({ page }) => {
    const checkout = new CheckoutFacade(page);
    
    const customerDetails = { name: 'John Doe', address: '123 Main St' };
    const paymentInfo = { cardNumber: '1234567890123456', expiry: '12/28' };

    // The test remains incredibly clean and readable
    await checkout.completeFullPurchase(customerDetails, paymentInfo);
});
```
