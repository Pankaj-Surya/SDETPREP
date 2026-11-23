# **SDET Week 1 – Day 1: Final Revision Notes**

# **Software Testing Fundamentals**

Final Notes

Interview Preparation

23 November 2026

I've incorporated your practical scenarios into the e-commerce workflow. I've kept your original structure and added a few useful testing considerations so that you can use these notes for both revision and interview discussions.

## **Part 1 – What is software testing?**

Software testing is the process of checking whether an application works as expected and identifying defects before the software reaches the end user.

**Example: E-commerce application**

A customer should be able to:

1. Login.
2. Search for a product.
3. Add a product to cart.
4. Proceed to checkout.
5. Make payment.
6. Receive order confirmation.

As a tester, we verify whether all these functionalities work correctly.

Testing is not just checking the happy path. We also test negative scenarios, edge cases, integrations and business rules.

**Why do we need software testing?**

- Identify defects early.
- Improve software quality.
- Reduce business risks.
- Improve customer experience.
- Verify business requirements.
- Build confidence before release.

**Interview answer:**

Software testing is important because it helps identify defects, verify business requirements, reduce risks and ensure that the application works as expected. It improves customer confidence and prevents costly issues in production.

## **Part 2 – QA vs QC**

# **QA**

**Quality Assurance**

Process-oriented

- Prevents defects.
- Improves development processes.
- Defines standards and practices.

# **QC**

**Quality Control**

Product-oriented

- Identifies defects.
- Checks actual application.
- Performs testing and inspection.

**Example:**

- QA: Establishing proper code review and testing processes.
- QC: Executing test cases to verify whether login works correctly.

| QA                 | QC                     |
| ------------------ | ---------------------- |
| Prevents defects   | Identifies defects     |
| Process-oriented   | Product-oriented       |
| Proactive          | More reactive          |
| Improves processes | Checks product quality |

**Interview answer:**

QA focuses on improving the process to prevent defects, whereas QC focuses on checking the actual product to identify defects.

## **Part 3 – Verification vs Validation**

# **Verification**

**Are we building the product right?**

Checks whether work products meet specified requirements.

Examples:

- Requirement review
- Test case review
- Design review
- Code review

# **Validation**

**Are we building the right product?**

Checks whether the actual application meets user needs.

Examples:

- Login execution
- Checkout testing
- Payment testing
- User acceptance testing

| Verification                                | Validation                               |
| ------------------------------------------- | ---------------------------------------- |
| Are we building it right?                   | Are we building the right product?       |
| Checks work products against specifications | Checks actual product against user needs |
| Reviews and inspections                     | Actual execution and testing             |

**Example:** For a tax calculation requirement:

- Verification: Review requirement and test cases for correct tax rules.
- Validation: Execute checkout and verify the actual tax calculation.

## **Part 4 – Error vs Defect vs Failure**

Example:

Requirement:

- Product price = ₹1,000
- Discount = 10%
- Expected final price = ₹900

Developer writes incorrect logic and subtracts ₹10 instead of ₹100.

1. **Error – Human mistake**

   Developer misunderstands the discount calculation.
2. **Defect – Problem in software**

   Incorrect discount calculation logic is implemented.
3. **Failure – Incorrect behavior**

   Customer sees ₹990 instead of ₹900.

**Remember:**

Human makes a mistake → Defect gets introduced → Failure may occur during execution.

A defect does not always cause an observable failure. It may remain hidden until a particular condition occurs.

| Term    | Meaning                                               |
| ------- | ----------------------------------------------------- |
| Error   | Human mistake                                         |
| Defect  | Problem in software or work product                   |
| Failure | Software does not behave as expected during execution |

## **Part 5 – Practical Exercise: E-commerce End-to-End Testing**

**Goal:** Think like a tester and identify defects at every stage of the customer purchase journey.

We will use one continuous e-commerce workflow.

### **Complete purchase workflow**

## **Customer purchase journey**

1. **Add product to cart**

   Product selection and quantity
2. **Cart displays product price**

   Price, quantity and currency
3. **Proceed to checkout**

   User authentication and order summary
4. **Calculate taxes and total**

   Discount, tax and final amount
5. **Complete payment**

   Payment validation and processing
6. **Create order**

   Order creation and order status
7. **Send confirmation**

   Email, SMS and order history

### **1. Customer adds a product to cart**

**Expected behavior:** Customer should be able to add an available product to the cart and see the correct quantity.

| Test scenario                                                   | Expected result                                                          |
| --------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Add to Cart button is disabled even though product is available | Button should be enabled.                                                |
| Click Add to Cart but product is not added                      | Product should be added successfully.                                    |
| Increase product quantity but cart displays incorrect quantity  | Cart should display the correct quantity.                                |
| Add the same product multiple times                             | Quantity should update according to the application's intended behavior. |
| Add an out-of-stock product                                     | Product should not be added for purchase.                                |

**Important testing areas:** Button state, product availability, quantity updates, cart synchronization.

### **2. Cart displays product price**

**Expected behavior:** Cart should display the correct product price and calculate the total based on quantity.

| Test scenario                              | Expected result                                                                                |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| Wrong product price displayed in cart      | Correct product price should be displayed.                                                     |
| Wrong calculation based on quantity        | Unit price × quantity should be correct.                                                       |
| Currency mismatch                          | Correct currency should be displayed.                                                          |
| Decimal or rounding issues                 | Amount should follow the defined currency precision and rounding rules.                        |
| Product price changes after adding to cart | Application should follow its defined price-update policy and clearly show the updated amount. |

**Example:**

- Product price = ₹1,000
- Quantity = 2
- Expected cart total = ₹2,000

**Important testing areas:** Price accuracy, quantity calculation, currency formatting, decimal precision.

### **3. Customer proceeds to checkout**

**Expected behavior:** Customer should be redirected to checkout and see the correct order summary.

| Test scenario                                       | Expected result                                                                   |
| --------------------------------------------------- | --------------------------------------------------------------------------------- |
| Clicking Checkout does not redirect                 | Customer should navigate to checkout.                                             |
| Wrong product details displayed                     | Correct product details should be displayed.                                      |
| Wrong quantity displayed                            | Correct quantity should be displayed.                                             |
| Wrong price displayed                               | Correct price should be displayed.                                                |
| Guest user tries to proceed when login is mandatory | User should be redirected to login or asked to authenticate.                      |
| User session expires during checkout                | User should be asked to authenticate again without losing valid cart information. |

**Important testing areas:** Navigation, authentication, session management, cart data consistency.

### **4. System calculates taxes and total**

**Expected behavior:** System should calculate the correct final amount based on product price, quantity, discount, applicable taxes and currency.

| Test scenario                          | Expected result                                                     |
| -------------------------------------- | ------------------------------------------------------------------- |
| Incorrect tax calculation              | Tax should follow applicable tax rules.                             |
| Incorrect calculation based on country | Correct country-specific tax rules should apply.                    |
| Currency mismatch                      | Correct currency and conversion rules should apply where supported. |
| Incorrect discount calculation         | Correct discount should be applied.                                 |
| Incorrect quantity calculation         | Total should reflect the correct quantity.                          |
| Decimal and rounding issues            | Final amount should follow defined rounding rules.                  |
| Incorrect grand total                  | Final amount should equal the sum of all applicable components.     |

**Example:**

Assume:

- Product price = ₹1,000
- Quantity = 2
- Discount = ₹200
- Tax = ₹144

**Expected price calculation**

Product total

₹2,000

Discount

-₹200

Tax

+₹144

**Final amount**

# **₹1,944**

*This is an illustrative calculation; actual tax rules and whether tax is calculated before or after discount depend on the business requirements.*

**Important testing areas:** Tax rules, country, currency, discounts, quantity, rounding and total calculation.

### **5. Customer completes payment**

**Expected behavior:** System should process payment securely, charge the correct amount only once and communicate the payment result.

| Test scenario                            | Expected result                                                                 |
| ---------------------------------------- | ------------------------------------------------------------------------------- |
| No notification after successful payment | Customer should receive payment success confirmation.                           |
| No notification after payment failure    | Customer should receive payment failure information.                            |
| Out-of-stock product payment accepted    | System should prevent purchase according to inventory rules.                    |
| Expired card                             | Payment should be rejected.                                                     |
| Invalid card details                     | Payment should be rejected.                                                     |
| Unsupported payment method               | Appropriate validation message should be displayed.                             |
| Third-party payment gateway failure      | System should handle the failure gracefully.                                    |
| Incorrect amount debited                 | Only the actual payable amount should be charged.                               |
| Amount debited twice                     | Only one successful charge should occur for the intended payment.               |
| Payment timeout or network interruption  | System should verify payment status before retrying to avoid duplicate charges. |

**Important testing areas:**

- Payment gateway integration.
- Positive and negative payment scenarios.
- Card validation.
- Payment timeout.
- Duplicate payment prevention.
- Payment status verification.
- Correct amount deduction.

**SDET interview point:** Payment is a critical integration. We should test successful, failed, pending and timeout scenarios, including retry and idempotency behavior.

### **6. System creates an order**

**Expected behavior:** After successful payment, the system should create exactly one order with correct details and update its status.

| Test scenario                                  | Expected result                                                 |
| ---------------------------------------------- | --------------------------------------------------------------- |
| Payment successful but order creation fails    | System should recover or reconcile the payment and order state. |
| Order created but not updated in order history | Order should appear in customer order history.                  |
| Order created but customer not notified        | Appropriate confirmation should be triggered.                   |
| Order created with wrong product details       | Correct product information should be stored.                   |
| Order created with wrong quantity              | Correct quantity should be stored.                              |
| Order created with wrong price                 | Correct order amount should be stored.                          |
| Order created despite payment failure          | Order should not be marked as successfully paid.                |
| Multiple orders created for a single checkout  | Only one intended order should be created.                      |

**Important testing areas:**

- Payment and order integration.
- Order data consistency.
- Order status.
- Duplicate order prevention.
- Transaction handling and recovery.
- Order history synchronization.

**Important distinction:** An order may have a pending or failed status depending on the system design. The critical requirement is that a failed payment must not result in an order incorrectly marked as paid or confirmed.

### **7. Customer receives confirmation**

**Expected behavior:** Customer should receive accurate order confirmation through the configured communication channels.

| Test scenario                        | Expected result                                                                  |
| ------------------------------------ | -------------------------------------------------------------------------------- |
| Confirmation email not sent          | Confirmation email should be triggered.                                          |
| Confirmation SMS not sent            | SMS should be triggered if required.                                             |
| Wrong email address used             | Notification should go to the correct registered or checkout email.              |
| Wrong order details in email         | Email should contain correct order information.                                  |
| Wrong payment amount in confirmation | Correct amount should be displayed.                                              |
| Duplicate confirmation messages      | Customer should not receive unintended duplicate notifications.                  |
| Notification service unavailable     | System should retry or handle notification failure without corrupting the order. |

**Important testing areas:** Email/SMS integration, notification content, correct recipient, delivery and retry mechanisms.

### **Practical revision: What should we cover at each stage?**

| Workflow stage | Main testing focus                       |
| -------------- | ---------------------------------------- |
| Add to cart    | Functionality, availability, quantity    |
| Cart           | Price, currency, calculations            |
| Checkout       | Navigation, authentication, data         |
| Tax and total  | Business rules, calculation              |
| Payment        | Integration, security, duplicate charges |
| Order creation | Data consistency, order status           |
| Confirmation   | Email, SMS, notification                 |

### **How to explain this practical exercise in an interview**

"If I were testing an e-commerce purchase journey, I would divide the complete workflow into different stages, starting from adding a product to the cart and ending with order confirmation.

At each stage, I would cover positive scenarios, negative scenarios, boundary conditions and integration failures.

For example, during payment, I would verify successful and failed transactions, invalid cards, payment gateway timeouts, correct amount deduction and duplicate payment prevention.

After payment, I would also verify that exactly one correct order is created and that the customer receives the right confirmation.

My objective would be to validate the complete customer journey, data consistency and business-critical functionality rather than testing each screen independently."

## **Part 6 – Coding / Revision: Five real-world defects**

For today's exercise, select five scenarios from Part 5 and document them.

Use this format:

| Field           | Example                                       |
| --------------- | --------------------------------------------- |
| Defect          | Duplicate payment                             |
| Expected result | One payment deduction                         |
| Actual result   | Two deductions                                |
| Business impact | Financial loss and customer dissatisfaction   |
| Test approach   | Click Pay once and verify transaction records |
| Priority        | High                                          |

Remember: As an SDET, we should think about how to reproduce the defect, validate the expected behavior, identify the root cause and automate the scenario where appropriate.

## **Part 7 – Interview Practice: Questions with Answers**

These are the seven questions from today's study session, with simple, interview-ready answers.

### **Q1. What is software testing, and why do we need it?**

**Answer:**

Software testing is the process of verifying and validating an application to ensure that it works as expected and meets business requirements.

We need software testing to:

- Identify defects before production.
- Ensure business requirements are met.
- Improve application quality.
- Reduce business and security risks.
- Provide a better customer experience.

**Example:** In an e-commerce application, we test whether a customer can add a product to the cart, make payment and successfully place an order.

### **Q2. What is the difference between QA and QC?**

**Answer:**

QA stands for Quality Assurance, whereas QC stands for Quality Control.

- QA focuses on preventing defects by improving processes.
- QC focuses on identifying defects by checking the actual product.

**Example:**

- QA: Defining coding standards and code review processes.
- QC: Executing test cases to identify defects in the application.

In simple words, QA is process-oriented, while QC is product-oriented.

### **Q3. What is verification vs validation?**

**Answer:**

Verification checks whether we are building the product right, whereas validation checks whether we are building the right product.

- Verification involves reviewing requirements, design documents, test cases and code.
- Validation involves checking the actual application against user requirements.

**Example:**

For an e-commerce checkout feature:

- Verification: Reviewing the requirement and test cases for tax calculation.
- Validation: Executing checkout and checking whether the actual tax is calculated correctly.

### **Q4. Explain error, defect and failure with an example.**

**Answer:**

These three terms represent different stages of a problem.

- Error: A human makes a mistake.
- Defect: That mistake introduces an issue into the software.
- Failure: The application produces incorrect behavior during execution.

**Example:**

Requirement: Product price is ₹1,000 with a 10% discount.

Expected price = ₹900.

- Error: Developer misunderstands the discount calculation.
- Defect: Incorrect calculation logic is implemented.
- Failure: Application displays ₹990 instead of ₹900.

### **Q5. Can software have defects without producing failures?**

**Answer:**

Yes. A defect does not always produce an observable failure.

A defect may remain hidden until a particular condition or input triggers it.

**Example:**

Suppose a tax calculation defect exists only for international customers.

For Indian customers, the application works correctly. But when an international customer places an order, the incorrect tax calculation becomes visible.

Therefore, the defect existed even before the failure occurred.

### **Q6. Can testing guarantee defect-free software?**

**Answer:**

No. Testing cannot guarantee that software is completely defect-free.

There are several reasons:

- Exhaustive testing is impossible.
- We cannot test every possible input and scenario.
- Some defects occur only under specific conditions.
- Requirements may be incomplete or misunderstood.
- New defects can be introduced when existing functionality changes.

Our objective is to identify as many important defects as possible, reduce risks and build confidence in the software.

### **Q7. What is the difference between positive and negative testing?**

**Answer:**

- Positive testing verifies whether the application works correctly with valid inputs.
- Negative testing verifies how the application behaves with invalid inputs or unexpected conditions.

**Example: Login functionality**

| Positive testing            | Negative testing     |
| --------------------------- | -------------------- |
| Valid username and password | Invalid username     |
| Correct credentials         | Incorrect password   |
| Successful login            | Empty username       |
| Valid email format          | Invalid email format |

Both are important to ensure application reliability.

## **Day 1 – Final revision cheat sheet**

## **Remember these concepts**

**Software Testing**

Checking whether software works as expected.

**QA**

Process-oriented – Prevent defects.

**QC**

Product-oriented – Identify defects.

**Verification**

Are we building the product right?

**Validation**

Are we building the right product?

**Error**

Human mistake.

**Defect**

Problem in software.

**Failure**

Incorrect behavior during execution.

**Positive Testing**

Test with valid inputs.

**Negative Testing**

Test with invalid inputs.

## **Day 1 – Learning outcome**

Your first day's focus is not just understanding definitions. It is learning to break down a real business workflow into testable scenarios.

Day 1 material completed
