In Playwright test automation, Association, Aggregation, and Composition are heavily used to structure the Page Object Model (POM).
Using these OOP principles prevents code duplication, organizes locator modules, and creates clean, maintainable automation frameworks. [1] 
------------------------------
## 1. Association (Weakest Link)
In test automation, an association occurs when one class uses or interacts with another class briefly to perform an action, but neither owns the other. They are completely independent. [2, 3, 4, 5, 6] 

* Automation Analogy: A LoginTest script interacts with a DatabaseUtility to fetch a one-time OTP or user credentials. Once the test finishes, the database utility still exists to serve other tests.
* Playwright Example: Passing a utility object into a page method. [7, 8] 

```
// Independent Class 1export class DatabaseUtility {
  async getLatestOTP(userEmail: string): Promise<string> {
    return '123456'; // Mimics fetching from DB
  }
}
// Independent Class 2export class LoginPage {
  // Association: The DatabaseUtility is passed as a parameter 
  // It is used briefly and is NOT stored as a permanent property
  async loginWithOTP(email: string, dbUtil: DatabaseUtility) {
    const otp = await dbUtil.getLatestOTP(email);
    console.log(`Logging in with OTP: ${otp}`);
  }
}
```

------------------------------
## 2. Aggregation (Has-A / Shared Relationship)
Aggregation occurs when a container class references outside objects. The parent class has these objects, but if the parent class is destroyed, the child objects survive because they were created externally. [9, 10, 11, 12] 

* Automation Analogy: A BaseTest setup or a test file aggregates multiple Page Objects (like LoginPage and DashboardPage). If one specific test block finishes and is destroyed, those Page Object classes still exist globally to be used by other test blocks.
* Playwright Example: Injecting pre-instantiated page objects into a Playwright custom fixture or test block. [13, 14, 15] 

```
import { test as base } from '@playwright/test';import { LoginPage } from './LoginPage';import { DashboardPage } from './DashboardPage';
// Defining the Aggregation Container via Playwright Fixturestype MyFixtures = {
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
};
export const test = base.extend<MyFixtures>({
  // The fixture receives existing, externally created page instances
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page)); 
  },
  dashboardPage: async ({ page }, use) => {
    await use(new DashboardPage(page));
  },
});
// Inside the test file:
test('User can checkout', async ({ loginPage, dashboardPage }) => {
  // The test aggregates both pages. 
  // If this specific test finishes, the LoginPage class definition remains untouched.
  await loginPage.navigate();
  await dashboardPage.verifyHeader();
});
```
------------------------------
## 3. Composition (Part-Of / Strict Ownership)
Composition is a strong relationship where a complex page object is built out of smaller component objects. The child objects are created inside the parent. If the parent page object is destroyed, the child components are destroyed with it. [16, 17] 

* Automation Analogy: A global DashboardPage contains a Navbar component, a Sidebar component, and a Footer component. These components cannot logically exist in the application without the main Dashboard page context.
* Playwright Example: Instantiating sub-components directly inside the constructor of a main Page Object. [18] 

```
import { Page, Locator } from '@playwright/test';
// Child Component: Only exists as a part of a pageclass NavbarComponent {
  readonly profileIcon: Locator;

  constructor(page: Page) {
    this.profileIcon = page.locator('#nav-profile');
  }

  async clickProfile() {
    await this.profileIcon.click();
  }
}
// Parent Page Objectexport class DashboardPage {
  readonly page: Page;
  // Composition: The Dashboard page strictly OWNS the Navbar
  readonly navbar: NavbarComponent; 

  constructor(page: Page) {
    this.page = page;
    // The child object is created inside the parent constructor
    this.navbar = new NavbarComponent(page); 
  }

  async loadDashboard() {
    await this.page.goto('/dashboard');
  }
}
// Usage in a Test File:
test('Composition Test', async ({ page }) => {
  const dashboard = new DashboardPage(page);
  await dashboard.loadDashboard();
  
  // Accessing the composed child component directly through the parent
  await dashboard.navbar.clickProfile(); 
  
  // If 'dashboard' goes out of scope here, 'navbar' is cleared from memory with it.
});
```
------------------------------
## Summary Cheat Sheet for Playwright

| Principle [19] | Playwright Implementation | Relationship | Lifecycle |
|---|---|---|---|
| Association | Passing an API/DB utility tool into a method argument. | Uses-A | Fully Independent |
| Aggregation | Combining multiple independent POMs into a single test.extend fixture. | Has-A | Standalone Survival |
| Composition | Instantiating a Header or Table fragment inside a MainPage constructor. | Part-Of | Dies with Parent |

------------------------------
To help refine your test suite structure, tell me:

* Are you currently building a framework from scratch, or refactoring an existing codebase?
* Do you use TypeScript/JavaScript or Python/Java for your Playwright scripts? [20] 

I can generate a tailored folder layout showing where to store these components!

[1] [https://idavidov.eu](https://idavidov.eu/understanding-object-oriented-programming-in-the-context-of-automation-qa)
[2] [https://medium.com](https://medium.com/@iamprashantverma/association-vs-aggregation-vs-composition-understanding-object-relationships-in-oop-620d4009bb3e)
[3] [https://www.tutorialsteacher.com](https://www.tutorialsteacher.com/csharp/association-and-composition)
[4] [https://www.linkedin.com](https://www.linkedin.com/pulse/understanding-four-types-relationships-c-programming-karwan-othman)
[5] [https://advoop.sdds.ca](https://advoop.sdds.ca/C-Class-Relationships/compositions-aggregations-and-associations)
[6] [https://medium.com](https://medium.com/@mizanur.rahman03032/mastering-object-relationships-composition-aggregation-and-association-b906390619c5)
[7] [https://testrigor.com](https://testrigor.com/blog/what-is-testware/)
[8] [https://egghead.io](https://egghead.io/lessons/playwright-playwright-tests-api-basics)
[9] [https://www.youtube.com](https://www.youtube.com/watch?v=TPUdUkFHD5I)
[10] [https://www.digitalocean.com](https://www.digitalocean.com/community/tutorials/composition-in-java-example)
[11] [https://blog.devgenius.io](https://blog.devgenius.io/youre-using-composition-and-aggregation-wrong-in-python-4fbd2400936c)
[12] [https://bellekens.com](https://bellekens.com/2010/12/20/uml-composition-vs-aggregation-vs-association/)
[13] [https://ceroshjacob.medium.com](https://ceroshjacob.medium.com/clean-test-cases-using-page-object-model-pom-in-playwright-b4f7066243b7)
[14] [https://www.thetesttribe.com](https://www.thetesttribe.com/author/ayushsingh/)
[15] [https://testdino.com](https://testdino.com/blog/playwright-page-object-model)
[16] [https://blog.stackademic.com](https://blog.stackademic.com/understanding-the-building-blocks-of-oop-association-aggregation-and-composition-6d9f33840c31)
[17] [https://medium.com](https://medium.com/@kuercan/composition-vs-aggregation-understanding-the-differences-and-when-to-use-each-4b54d77fe892)
[18] [https://testdino.com](https://testdino.com/blog/playwright-page-object-model)
[19] [https://www.scribd.com](https://www.scribd.com/document/925763417/Composition-and-Aggregation)
[20] [https://testguild.com](https://testguild.com/podcast/a562-naeem/)
