# **Week 1 – Day 5: Testing Levels**



Today's topic

**Unit Testing, Integration Testing, System Testing & Acceptance Testing**



**Today's goal:** Understand the four testing levels, why each level is needed, who generally performs it, and how to explain them in an SDET interview.

We'll use the same e-commerce application throughout, so you can see how testing progresses from a small piece of code to the complete customer purchase journey.

## **1. What are testing levels?**

Testing levels describe **at what stage and at what scope we test the application**.

Think of an e-commerce purchase flow:

**1. Unit Testing**

Test one small unit, such as a price-calculation function.

**2. Integration Testing**

Test whether components or services work correctly together.

**3. System Testing**

Test the complete e-commerce application end to end.

**4. Acceptance Testing**

Check whether the application meets business and user needs.

**Simple way to remember:**

- Unit: Does this individual piece work?
- Integration: Do the connected pieces work together?
- System: Does the complete application work?
- Acceptance: Is the application acceptable for business or users?

## **2. Unit Testing**

**Definition:** Unit testing means testing the smallest testable part of an application independently.

A unit can be a function, method, or class.

**E-commerce example:** Suppose we have a function to calculate the total price of products in the cart.

```
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

console.log(calculateTotal(1000, 2)); // 2000
```

We want to verify:

- Price ₹1,000 and quantity 2 → ₹2,000.
- Price ₹500 and quantity 3 → ₹1,500.
- Quantity 0 → ₹0, if allowed by the requirement.
- Negative quantity → should be rejected, if the requirement disallows it.

A unit test using Jest-style syntax:

```
describe('calculateTotal', () => {
  it('should calculate total price', () => {
    expect(calculateTotal(1000, 2)).toBe(2000);
  });

  it('should return zero for zero quantity', () => {
    expect(calculateTotal(1000, 0)).toBe(0);
  });
});
```

**Who generally performs it?** Developers usually write and run unit tests. SDETs may also contribute to unit-level test strategy, testability, and quality.

**Key points:**

- Tests a small piece of code.
- Usually runs quickly.
- Dependencies are often isolated using mocks or stubs.
- Helps find defects early, close to the code where they were introduced.

**Interview explanation:**

"Unit testing verifies individual functions, methods, or classes independently. For example, in an e-commerce application, I can test the cart price-calculation function without involving the database, payment service, or UI."

## **3. Integration Testing**

**Definition:** Integration testing verifies whether two or more components, modules, or services work correctly together.

A component can work perfectly on its own but fail when connected to another component.

**E-commerce example:**

When a customer adds a product to the cart:

1. UI sends an add-to-cart request.
2. Cart API receives the request.
3. Cart service fetches product details.
4. Cart service updates cart data.
5. API returns the updated cart.

We need to verify whether these connected components communicate correctly.

**Example: Add to Cart integration**

**UI / Frontend**

User clicks Add to Cart

**Cart API**

Receives product ID and quantity

**Product / Inventory Service**

Checks product and availability

**Cart Database**

Stores updated cart

**What should we verify?**

- Does the Cart API receive the correct product ID?
- Does it fetch the correct product price?
- Does it reject an out-of-stock product?
- Is the correct quantity saved in the database?
- Does the API return the updated cart?

**Who generally performs it?** Developers and QA/SDET engineers, depending on the team's testing strategy and architecture.

**Interview explanation:**

"Integration testing verifies the interaction between connected components. For example, I would verify that the Cart Service correctly communicates with the Product Service, checks availability, stores the cart item, and returns the correct response."

## **4. System Testing**

**Definition:** System testing verifies the complete, integrated application against its functional and non-functional requirements.

Here, we test the application as a whole rather than testing only individual components or their connections.

**E-commerce example: Complete purchase journey**

1. **Login**

   User logs in successfully.
2. **Product search**

   User searches for a product.
3. **Add to cart**

   User adds the product and verifies quantity and price.
4. **Checkout**

   User enters shipping and billing details.
5. **Payment**

   User completes payment using a valid card.
6. **Order confirmation**

   Order is created and confirmation is displayed.

We verify the complete flow:

- Can the user complete the purchase?
- Is the correct amount charged?
- Is the order created with the correct details?
- Does the order appear in order history?
- Does the customer receive confirmation?

**Playwright example:**

```
import { test, expect } from '@playwright/test';

test('customer can complete a purchase', async ({ page }) => {
  await page.goto('/login');

  await page.getByLabel('Email').fill('test@example.com');
  await page.getByLabel('Password').fill('Test@123');
  await page.getByRole('button', { name: 'Login' }).click();

  await page.getByRole('button', { name: 'Add to Cart' }).click();
  await page.getByRole('link', { name: 'Cart' }).click();

  await expect(page.getByText('₹1,000')).toBeVisible();

  await page.getByRole('button', { name: 'Checkout' }).click();

  // Continue with shipping, payment and order confirmation checks.
});
```

This is an illustrative system-level UI test; the actual selectors and checkout steps depend on the application.

**Who generally performs it?** QA/SDET engineers commonly own or execute system testing, with developers contributing to automation and debugging.

**Interview explanation:**

"System testing validates the complete application end to end. For an e-commerce application, I would test the entire purchase journey from login and product selection through payment and order confirmation."

## **5. Acceptance Testing**

**Definition:** Acceptance testing verifies whether the application meets business requirements and is ready to be accepted by the customer or business.

The focus is not just whether the software works technically, but whether it solves the intended business problem.

**E-commerce example:**

Business requirement:

"Customers should be able to place an order using a valid credit card. After successful payment, the system should create an order and send an order confirmation."

Acceptance checks:

- Can a customer successfully place an order using a valid credit card?
- Is an order created only after successful payment?
- Is the correct order amount charged?
- Is the confirmation sent to the customer?
- Does the overall flow meet the agreed business requirements?

**Who generally performs it?**

- Business users
- Product Owner
- Customer/client representatives
- End users in selected acceptance exercises

QA/SDET can support acceptance testing by preparing scenarios, test data, and evidence.

**Common types to know:**

| Type                          | Meaning                                                           |
| ----------------------------- | ----------------------------------------------------------------- |
| UAT (User Acceptance Testing) | Business users or end users verify business requirements          |
| Alpha testing                 | Evaluation in a controlled environment, often by internal users   |
| Beta testing                  | Evaluation by selected external users in a real-world environment |

**Interview explanation:**

"Acceptance testing checks whether the application meets business expectations and is ready for release. For example, business users verify that a customer can place an order, make payment, and receive confirmation as defined in the requirements."

## **6. Compare all four testing levels**

| Area          | Unit                 | Integration                             | System                 | Acceptance                         |
| ------------- | -------------------- | --------------------------------------- | ---------------------- | ---------------------------------- |
| Scope         | Individual unit      | Connected components                    | Complete application   | Business requirements              |
| Main focus    | Code correctness     | Component interaction                   | End-to-end behavior    | Business acceptance                |
| Typical owner | Developer            | Developer / QA                          | QA / SDET              | Business / users                   |
| Example       | Calculate cart total | Cart API + Product service              | Complete purchase flow | Business approves purchase journey |
| Dependencies  | Usually isolated     | Real or controlled connected components | Integrated environment | Business-like environment          |

*Remember:* Ownership can vary by organization. These are common patterns, not strict rules.

## **7. Important interview distinction: Testing levels vs testing types**

This is a common interview question.

**Testing levels** tell us *where or at what scope* we test:

- Unit
- Integration
- System
- Acceptance

**Testing types** tell us *what kind of quality or behavior* we are checking:

- Functional testing
- Regression testing
- Smoke testing
- Sanity testing
- Exploratory testing
- Performance testing
- Security testing

For example, performance testing is a testing type, not a testing level. It can be performed at different scopes, depending on the objective.

## **8. Real-world scenario: How would you test checkout at all four levels?**

Let's take one requirement:

"Customer should be able to add a product to cart, complete payment, and receive order confirmation."

| Testing level | What would you test?                                                         |
| ------------- | ---------------------------------------------------------------------------- |
| Unit          | Cart total calculation, discount calculation, tax calculation                |
| Integration   | Cart service with Product service; Order service with Payment service        |
| System        | Complete purchase journey from login to order confirmation                   |
| Acceptance    | Business verifies the purchase flow meets customer and business requirements |

**Important SDET perspective:** We should not depend only on end-to-end UI tests. A good strategy tests business logic at unit level, service interactions at integration level, and critical customer journeys at system level. Acceptance testing validates business expectations.

## **9. Interview questions with expected answers**

Try to understand the answers and explain them in your own words.

**Q1. What are the different levels of testing?**

There are four commonly recognized levels: Unit, Integration, System, and Acceptance testing. They cover testing individual units, interactions between components, the complete application, and business acceptance.

**Q2. What is the difference between unit and integration testing?**

Unit testing verifies an individual function or component independently. Integration testing verifies whether multiple components work correctly together.

**Q3. What is the difference between integration and system testing?**

Integration testing focuses on communication between connected components. System testing validates the complete integrated application against its requirements.

**Q4. Who performs acceptance testing?**

Usually business users, Product Owners, customers, or their representatives. QA supports them by preparing test scenarios, data, and evidence.

**Q5. Can we automate all four testing levels?**

Automation can support all four levels, but the tools and ownership differ. Unit tests are commonly automated by developers; integration and system tests can be automated by developers and SDETs. Acceptance testing may also be automated for repeatable business workflows, while business sign-off remains a business decision.

**Q6. Is API testing integration testing?**

Not always. API testing is a way to test an interface. If we verify interactions between services through APIs, it can be integration testing. If we validate a single service's behavior against its requirements, it may be service-level or component testing.

**Q7. Is end-to-end testing the same as system testing?**

- They overlap considerably. System testing validates the complete application, while end-to-end testing usually emphasizes a complete business journey across the systems and services involved. The terms are sometimes used interchangeably by teams.
- No, end-to-end (E2E) testing and system testing are not the exact same thing, though they overlap
- System Testing is a broad phase that evaluates the entire software application as a whole against its specified functional and non-functional requirements. It looks at internal functionality, performance, security, and stability in a controlled/lab environment.
- End-to-End Testing is a specific subset or approach that focuses strictly on validating complete user workflows and behavioral flows from start to finish—often including external interfaces, databases, third-party services, and real-world user scenarios in a production-like environment.

## **10. Practical assignment – Identify testing levels for checkout**



Your task

&#x20;Try this independently before looking up answers.



Consider the following e-commerce scenarios. Identify whether each one is Unit, Integration, System, or Acceptance testing.

**Classify the scenarios**

8/8 answered

**1. Verify that the cart total function calculates ₹2,000 for two products costing ₹1,000 each.**

Unit

Integration

System

Acceptance

Correct — Unit

**2. Verify that the Cart Service receives the correct product price from the Product Service.**

Unit

Integration

System

Acceptance

Correct — Integration

**3. Verify that a customer can log in, add a product, pay, and see the order confirmation.**

Unit

Integration

System

Acceptance

Correct — System

**4. Business users verify that the checkout flow satisfies the agreed business requirement.**

Unit

Integration

System

Acceptance

Correct — Acceptance

**5. Verify that the Order Service correctly communicates with the Payment Service and handles a payment failure response.**

Unit

Integration

System

Acceptance

Correct — Integration

**6. Verify that the discount calculation function applies a 10% discount correctly.**

Unit

Integration

System

Acceptance

Correct — Unit

**7. The business team validates that an order confirmation is sent after a successful purchase.**

Unit

Integration

System

Acceptance

Correct — Acceptance

**8. Verify that the complete application handles a customer's purchase and displays the order in order history.**

Unit

Integration

System

Acceptance

Correct — System

Check my answersReset

# **8 / 8**

Review any incorrect answers and explain why the level applies.

## **11. Today's 60-minute study schedule**

| Time      | Activity                                        |
| --------- | ----------------------------------------------- |
| 0–10 min  | Understand the four testing levels              |
| 10–20 min | Unit and Integration testing examples           |
| 20–30 min | System and Acceptance testing examples          |
| 30–40 min | Compare levels and answer interview questions   |
| 40–50 min | Complete the checkout classification assignment |
| 50–60 min | Revision and coding exercise                    |

**20-minute coding/revision focus:** Write a small TypeScript unit test for cart total calculation, then explain how you would test the interaction between Cart API and Product API.

## **12. One-minute revision**

**Remember these four lines:**

1. **Unit:** Test an individual function or component.
2. **Integration:** Test communication between components.
3. **System:** Test the complete application.
4. **Acceptance:** Verify business requirements and user expectations.

**Interview tip:** Explain all four levels using one continuous e-commerce checkout example. This makes your answer easier to understand and demonstrates practical thinking.

## **13. Day 5 completion checklist**

**My progress**

## **0/7**

I can explain all four testing levels without notes.

I can differentiate Unit vs Integration testing.

I can differentiate Integration vs System testing.

I can explain System vs Acceptance testing.

I can identify the testing level from a real-world scenario.

I completed the checkout practical assignment.

I can explain testing levels in an interview.

**Expected output:** Be able to identify testing levels for an e-commerce checkout flow and explain the differences confidently in an interview. This matches the Day 5 exercise in your preparation roadmap.&#x20;

Pasted text(1).txt



**Next session – Day 6:** Testing Types — Functional, Regression, Smoke, Sanity, Exploratory and Non-functional Testing.

When you're ready, share your practical answers or say **"Day 5 assessment"** and I'll review your understanding like an interviewer.
