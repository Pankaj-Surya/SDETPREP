# **Week 1 – Day 6: Testing Types**



Today's topic

**Functional, Regression, Smoke, Sanity, Exploratory & Non-functional Testing**



**Today's goal:** Understand different testing types, when to use them, and how to identify them in real-world scenarios and SDET interviews.

We'll continue with our e-commerce application and use examples from the complete purchase journey.

Today's assignment from your roadmap is to prepare **15 real-world examples of testing types**. I'll solve that assignment in the lesson itself, with answers and explanations.

## **1. What are testing types?**

Testing types describe **what we want to verify or what quality aspect we want to evaluate**.

For example, consider an e-commerce application:

- Does the login feature work correctly? → Functional testing.
- Does the application still work after a new feature is added? → Regression testing.
- Is the application stable enough for detailed testing? → Smoke testing.
- Does a particular bug fix work correctly? → Sanity testing.
- What happens when we explore the application without predefined test cases? → Exploratory testing.
- Can the application handle 10,000 concurrent users? → Performance testing.

**Remember the difference from Day 5:**

| Testing levels (Day 5) | Testing types (Day 6) |
| ---------------------- | --------------------- |
| Unit                   | Functional            |
| Integration            | Regression            |
| System                 | Smoke                 |
| Acceptance             | Sanity                |
|                        | Exploratory           |
|                        | Non-functional        |

Testing levels tell us the scope at which we test. Testing types tell us what we are checking. They can be combined: for example, functional testing can happen at unit, integration, or system level.

## **2. Functional Testing**

**Definition:** Functional testing verifies whether an application behaves according to the functional requirements.

In simple words: Are the features working as expected?

**E-commerce examples:**

- User can log in using valid credentials.
- User cannot log in using an incorrect password.
- User can search for a product.
- User can add an available product to the cart.
- Cart displays the correct price and quantity.
- User can complete payment using a valid card.
- Order is created after successful payment.

**Example test case:**

| Field           | Details                                   |
| --------------- | ----------------------------------------- |
| Scenario        | Add an available product to cart          |
| Input           | Product price ₹1,000, quantity 2          |
| Expected result | Cart displays quantity 2 and total ₹2,000 |
| Actual result   | Compare with expected result              |
| Status          | Pass / Fail                               |

**Interview answer:**

"Functional testing verifies application features against business requirements. For example, in an e-commerce application, I verify login, product search, add to cart, checkout, payment, and order creation."

## **3. Regression Testing**

**Definition:** Regression testing verifies that existing functionality is still working after code changes, bug fixes, or new feature additions.

In simple words: Did the new change break something that was already working?

**E-commerce example:**

Suppose the development team introduces a discount coupon feature.

Before the change:

- Product price = ₹1,000.
- Quantity = 2.
- Cart total = ₹2,000.

After the change, we should test:

- Does the existing add-to-cart functionality still work?
- Is the original product price displayed correctly?
- Is the cart quantity still correct?
- Does checkout still work?
- Does payment still work?
- Is the discount applied correctly?

The new coupon feature may work perfectly, but it could accidentally break the existing cart calculation.

**Regression testing flow**

**Existing application is working**

**New feature / Bug fix introduced**

**Run existing relevant test cases**

**Confirm existing functionality still works**

**When do we perform regression testing?**

- After adding a new feature.
- After fixing a bug.
- After code refactoring.
- After changing an existing module.
- Before a release.

**SDET perspective:** Regression testing is a strong candidate for automation because the same test cases need to be executed repeatedly.

**Interview answer:**

"Regression testing ensures that existing functionality is not broken by new code changes. For example, after introducing a coupon feature, I would rerun relevant cart, pricing, checkout, and payment tests to make sure the existing purchase flow continues to work."

## **4. Smoke Testing**

**Definition:** Smoke testing is a basic level of testing performed on a new build to verify that the critical functionalities are working and the build is stable enough for further testing.

In simple words: Is the build good enough to start detailed testing?

**E-commerce example:**

A new e-commerce build is deployed to the QA environment.

Before starting detailed testing, we check:

1. Can the application open?
2. Can the user log in?
3. Can the product listing page load?
4. Can the user add a product to the cart?
5. Can the checkout page open?
6. Can the payment page load?

If critical functionality is broken, the QA team may reject the build and report the issue rather than spending time on detailed testing.

**Key points:**

- Broad coverage of critical functionality.
- Usually a small set of high-priority tests.
- Performed on a new build or deployment.
- Helps decide whether further testing can begin.

**Interview answer:**

"Smoke testing is performed on a new build to verify that the critical functionalities are working. If the basic flows fail, we reject the build and report the issues instead of proceeding with detailed testing."

## **5. Sanity Testing**

**Definition:** Sanity testing is a focused testing activity performed after a small change or bug fix to verify that the affected functionality is working correctly.

In simple words: Did the specific change or bug fix work?

**E-commerce example:**

Bug reported:

"When a customer applies a 10% discount coupon, the cart displays the wrong final amount."

Developer fixes the issue.

Now QA performs focused checks:

- Apply a valid 10% coupon.
- Verify the discount amount.
- Verify the final cart total.
- Check an invalid coupon.
- Check an expired coupon, if applicable.

The purpose is to verify the changed functionality and closely related behavior, rather than rerunning the entire application.

**Key points:**

- Narrow and focused scope.
- Commonly performed after bug fixes or small changes.
- Checks whether the specific change works.
- Does not replace broader regression testing.

**Interview answer:**

"Sanity testing is focused testing performed after a small change or bug fix. For example, after fixing a coupon calculation issue, I verify the discount calculation and related scenarios rather than executing the complete application test suite."

## **6. Smoke Testing vs Sanity Testing**

Important interview question

| Smoke Testing                                                  | Sanity Testing                             |
| -------------------------------------------------------------- | ------------------------------------------ |
| Checks basic critical functionality                            | Checks a specific changed area             |
| Broad but shallow                                              | Narrow but focused                         |
| Usually after a new build                                      | Usually after a small change or bug fix    |
| Decides whether the build is stable enough for further testing | Checks whether the specific change works   |
| Example: Login, search, cart, checkout                         | Example: Verify a fixed coupon calculation |

**Easy way to remember:**

- Smoke: Can I start testing this build?
- Sanity: Does this particular change look correct?

*Note:* Teams sometimes use these terms differently in practice. In interviews, explain the distinction above and then mention that exact team workflows can vary.

## **7. Exploratory Testing**

**Definition:** Exploratory testing is an approach where the tester explores the application, learns its behavior, designs tests, and executes them at the same time.

Unlike scripted testing, we do not necessarily follow predefined test cases step by step.

In simple words: Explore the application like a real user and try to find unexpected problems.

**E-commerce example:**

You are testing the cart functionality.

Instead of following only predefined test cases, you explore:

- Add the same product multiple times.
- Increase quantity rapidly.
- Remove the product while checkout is open in another tab.
- Refresh the page after adding a product.
- Open the cart in multiple browser tabs.
- Change product availability while the product is already in the cart.
- Try applying multiple coupons.
- Navigate backward and forward during checkout.

You may discover unexpected defects that were not covered in existing test cases.

**What makes a good exploratory tester?**

- Understands the business flow.
- Thinks from the customer's perspective.
- Tries unusual combinations and edge cases.
- Observes unexpected behavior.
- Records steps and evidence when a defect is found.

**Interview answer:**

"Exploratory testing involves learning, designing, and executing tests simultaneously. For example, while testing the shopping cart, I may try adding the same product multiple times, refreshing the page, or modifying the cart in multiple tabs to identify unexpected defects."

## **8. Non-functional Testing**

**Definition:** Non-functional testing verifies how well an application works rather than only checking what the application does.

It evaluates quality attributes such as performance, security, usability, reliability, and compatibility.

**E-commerce examples:**

| Testing type  | What do we verify?                | Example                                                   |
| ------------- | --------------------------------- | --------------------------------------------------------- |
| Performance   | Speed and responsiveness          | Product page loads within the agreed response time        |
| Load          | Behavior under expected load      | 5,000 concurrent users browse products                    |
| Stress        | Behavior beyond expected capacity | Increase users until the application reaches its limits   |
| Security      | Protection against attacks        | Verify users cannot access another user's order           |
| Usability     | Ease of use                       | Customer can easily complete checkout                     |
| Compatibility | Behavior across environments      | Application works on Chrome, Firefox, and Safari          |
| Reliability   | Consistent operation over time    | Application continues working without unexpected failures |
| Scalability   | Ability to handle growth          | Application can support increasing users and transactions |

Let's understand some important non-functional types in more detail.

### **8.1 Performance Testing**

Checks application speed, responsiveness, stability, and resource usage.

Example:

- Product search response time.
- Checkout API response time.
- Page loading time.

### **8.2 Load Testing**

Checks application behavior under expected or anticipated user load.

Example:

- 1,000 users browsing products.
- 2,000 users adding products to cart.
- 500 users completing checkout.

### **8.3 Stress Testing**

Checks application behavior beyond its expected capacity to understand its breaking point and recovery behavior.

Example:

- Expected load is 5,000 concurrent users.
- Gradually increase to 10,000 or more.
- Observe failures, response times, and recovery.

### **8.4 Security Testing**

Checks whether application data and functionality are protected from unauthorized access and attacks.

E-commerce examples:

- User A should not access User B's order.
- Passwords should not be exposed in responses.
- Payment details should be protected.
- Unauthorized users should not access admin APIs.

### **8.5 Compatibility Testing**

Checks whether the application works correctly across different browsers, devices, operating systems, and environments.

Example:

- Chrome on Windows.
- Firefox on Windows.
- Safari on macOS.
- Mobile browsers on Android and iOS.

### **8.6 Usability Testing**

Checks whether the application is easy to understand and use.

Example:

- Can a customer easily find the checkout button?
- Are error messages understandable?
- Is the payment process simple?

**Interview answer:**

"Non-functional testing verifies quality attributes of an application, such as performance, security, usability, compatibility, and reliability. For example, I would verify that the e-commerce application can handle expected concurrent users while keeping response times within agreed limits."

## **9. Important comparison: Functional vs Non-functional testing**

| Functional Testing                   | Non-functional Testing                          |
| ------------------------------------ | ----------------------------------------------- |
| Checks what the application does     | Checks how well the application works           |
| Focuses on business functionality    | Focuses on quality attributes                   |
| Login, cart, payment, order creation | Performance, security, usability, compatibility |
| Example: User can complete payment   | Example: Payment page responds within 2 seconds |

Example:

- Functional: Can 1,000 users place orders correctly?
- Non-functional: Can the system handle 1,000 concurrent users within the required response time?

Both are important. An application may be functionally correct but still fail because it is too slow or insecure.

## **10. How do these testing types work together in a real project?**

Let's imagine a new build is deployed after introducing a coupon feature.

1. **Smoke Testing**

   Check whether login, product listing, cart, and checkout are working on the new build.
2. **Functional Testing**

   Verify valid, invalid, and expired coupon behavior.
3. **Sanity Testing**

   Focus on the coupon calculation bug fix or changed functionality.
4. **Regression Testing**

   Verify that cart, pricing, checkout, payment, and order creation still work.
5. **Exploratory Testing**

   Explore unusual combinations, such as multiple coupons or changing quantities after applying a coupon.
6. **Non-functional Testing**

   Check performance, security, compatibility, and usability where required.

This is an illustrative workflow, not a mandatory fixed sequence. Actual execution depends on the release, risk, and team's test strategy.

## **11. Playwright perspective: How would an SDET automate these tests?**

Playwright is particularly useful for automating browser-based functional, smoke, sanity, and regression tests.

Example: A functional test for adding a product to the cart.

```
import { test, expect } from '@playwright/test';

test('user can add product to cart', async ({ page }) => {
  await page.goto('/products');

  await page.getByRole('button', {
    name: 'Add to Cart'
  }).click();

  await expect(
    page.getByText('Added to cart')
  ).toBeVisible();

  await expect(
    page.getByTestId('cart-count')
  ).toHaveText('1');
});
```

How can we classify this test?

- It is functional testing because it verifies the add-to-cart requirement.
- It can be part of a smoke suite if selected as a critical build-health check.
- It can be part of a regression suite if rerun after code changes.
- It can be part of a sanity suite if it specifically verifies a recent add-to-cart fix.

**Important:** A test case is not permanently tied to only one testing type. Its classification depends on why, when, and how we execute it.

For non-functional testing, Playwright can help with certain browser-level checks, but dedicated tools are often more suitable for load and stress testing. For example, tools such as Apache JMeter and k6 are commonly used for performance testing.

## **12. Day 6 assignment – 15 real-world examples with answers**

Your roadmap asks you to prepare 15 examples of testing types. Here are 15 solved examples from our e-commerce application.

| #  | Scenario                                                                                                         | Testing type  | Why?                                      |
| -- | ---------------------------------------------------------------------------------------------------------------- | ------------- | ----------------------------------------- |
| 1  | Verify that a user can log in with valid credentials.                                                            | Functional    | Checks login behavior.                    |
| 2  | Verify that an invalid password is rejected.                                                                     | Functional    | Checks a negative business scenario.      |
| 3  | Verify that a customer can add an available product to cart.                                                     | Functional    | Checks add-to-cart behavior.              |
| 4  | After adding a coupon feature, verify that existing cart and checkout functionality still works.                 | Regression    | Checks for side effects of a new change.  |
| 5  | After a new build deployment, verify login, product listing, cart, and checkout pages open.                      | Smoke         | Checks basic build stability.             |
| 6  | After fixing a coupon calculation bug, verify the discount and final amount.                                     | Sanity        | Focuses on the changed functionality.     |
| 7  | Explore the cart by refreshing, opening multiple tabs, and changing quantities unexpectedly.                     | Exploratory   | Finds issues beyond predefined cases.     |
| 8  | Verify that product search returns results within the agreed response time.                                      | Performance   | Checks responsiveness.                    |
| 9  | Verify that the system handles 5,000 concurrent users as expected.                                               | Load          | Checks behavior under expected load.      |
| 10 | Increase concurrent users beyond expected capacity to find the system's breaking point.                          | Stress        | Checks behavior under excessive load.     |
| 11 | Verify that one customer cannot access another customer's order details.                                         | Security      | Checks authorization and data protection. |
| 12 | Verify that checkout works on Chrome, Firefox, Safari, and mobile browsers.                                      | Compatibility | Checks different browsers and devices.    |
| 13 | Verify that customers can easily understand checkout instructions and error messages.                            | Usability     | Checks ease of use.                       |
| 14 | Verify that the application continues processing orders reliably over an extended period.                        | Reliability   | Checks consistent operation over time.    |
| 15 | Verify that the application can support increasing traffic by adding resources without unacceptable degradation. | Scalability   | Checks its ability to grow.               |

Note: These are the primary classifications for each scenario. Some scenarios can belong to more than one category depending on the objective and execution context.

### **Assignment takeaways**

- Examples 1–3: Functional testing.
- Example 4: Regression testing.
- Example 5: Smoke testing.
- Example 6: Sanity testing.
- Example 7: Exploratory testing.
- Examples 8–15: Non-functional testing.

You now have all 15 examples with their expected classifications and reasoning.

## **13. Common SDET interview questions**

**Q1. What is the difference between smoke and sanity testing?**

Smoke testing checks critical functionalities of a new build to determine whether further testing can begin. Sanity testing focuses on a specific changed area after a small change or bug fix.

**Q2. What is regression testing and when do you perform it?**

Regression testing verifies that existing functionality is not broken after code changes, new features, or bug fixes. It is commonly performed before releases and after significant changes.

**Q3. What is exploratory testing?**

It is an approach where the tester learns, designs, and executes tests at the same time, exploring the application to find unexpected defects.

**Q4. What is the difference between functional and non-functional testing?**

Functional testing checks what the application does. Non-functional testing checks how well it performs, such as speed, security, usability, and reliability.

**Q5. What is the difference between load and stress testing?**

Load testing checks the application under expected user load. Stress testing pushes it beyond expected capacity to understand its limits and recovery behavior.

**Q6. Can smoke testing be automated?**

Yes. We can automate critical test cases using Playwright and execute them after every deployment to check basic application health.

**Q7. Can regression testing be automated?**

Yes. Regression testing is one of the strongest candidates for automation because existing test cases need to be executed repeatedly after code changes.

**Q8. Is sanity testing always performed after smoke testing?**

No. Smoke and sanity testing have different purposes. Their execution order depends on the team's workflow and the type of change.

**Q9. Can one test case belong to both smoke and regression testing?**

Yes. For example, a login test can be included in a smoke suite to check build stability and in a regression suite to verify that login remains unaffected by changes.

**Q10. Is performance testing functional or non-functional?**

Performance testing is non-functional because it checks application responsiveness, speed, stability, and resource usage.

## **14. Today's 60-minute study schedule**

| Duration  | Activity                                            |
| --------- | --------------------------------------------------- |
| 0–10 min  | Functional and regression testing                   |
| 10–20 min | Smoke vs sanity testing                             |
| 20–30 min | Exploratory testing                                 |
| 30–40 min | Non-functional testing                              |
| 40–50 min | Revise the 15 solved examples                       |
| 50–60 min | Practice interview questions and Playwright example |

**20-minute coding/revision focus:** Write one Playwright functional test and explain how you would reuse it in smoke and regression suites.

## **15. One-minute revision notes**

**Functional:** Does the feature work?

**Regression:** Did the new change break existing functionality?

**Smoke:** Is the new build stable enough for further testing?

**Sanity:** Does the specific change or bug fix work?

**Exploratory:** Can I find unexpected defects by exploring?

**Non-functional:** How well does the application work?

**Load:** Can it handle expected traffic?

**Stress:** What happens beyond expected capacity?

**Security:** Is the application protected?

**Compatibility:** Does it work across browsers and devices?

## **16. Day 6 completion checklist**

Use this checklist to track your understanding.

**My progress**

## **0/8**

I understand functional testing with e-commerce examples.

I can explain regression testing and when to perform it.

I can differentiate smoke and sanity testing.

I understand exploratory testing.

I can explain functional vs non-functional testing.

I can differentiate load vs stress testing.

I reviewed all 15 solved assignment examples.

I can explain these concepts in an interview.

## **Next: Day 7 – Week 1 Revision & Assessment**

We'll consolidate the first six days:

1. Software testing fundamentals.
2. Seven testing principles.
3. SDLC – Waterfall, Agile, Scrum and V-Model.
4. STLC – Six testing phases.
5. Testing levels – Unit, Integration, System and Acceptance.
6. Testing types – Functional, Regression, Smoke, Sanity, Exploratory and Non-functional.

The roadmap's weekly assessment includes theory, interview explanations, practical work, coding/problem solving, and debugging/edge cases. Our target is **75/100** before moving forward.&#x20;

Pasted text(1).txt



**My recommendation:** Before Day 7, revise the 15 examples above once. They will help you distinguish testing types quickly during interviews.
