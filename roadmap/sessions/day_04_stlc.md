# **Week 1 – Day 4: STLC (Software Testing Life Cycle)**

Day 4

26 November 2026

60 minutes

**Today's goal:** Understand the complete testing process followed by a QA team, from analysing requirements to closing testing activities.

We'll use our e-commerce application as the continuous example.

## **1. What is STLC?**

**STLC stands for Software Testing Life Cycle.**

It is a step-by-step process followed by the QA team to perform software testing in a structured way.

In simple words:

- SDLC explains how software is developed.
- STLC explains how software testing is performed.

**Example:** Imagine developers have built an e-commerce checkout feature. Before releasing it, QA needs to understand requirements, plan testing, prepare test cases, execute them, report defects and complete testing.

STLC provides a structured process for doing this.

## **2. STLC phases**

There are six commonly used phases in the STLC.

## **STLC lifecycle**

**1**

**Requirement Analysis**

Understand what needs to be tested

**2**

**Test Planning**

Decide scope, resources, effort and schedule

**3**

**Test Case Development**

Prepare test scenarios, cases and test data

**4**

**Test Environment Setup**

Prepare the environment for testing

**5**

**Test Execution**

Execute test cases and report defects

**6**

**Test Cycle Closure**

Evaluate results and complete testing

Simplified STLC flow. Activities can overlap or repeat when requirements change or defects are fixed.

## **3. Understand each STLC phase with an e-commerce example**

### **Phase 1: Requirement Analysis**

In this phase, QA understands the business and functional requirements and identifies what needs to be tested.

**Example requirement:**

"When a customer clicks the Add to Cart button, the selected product should be added to the shopping cart."

**What does QA do?**

- Understand the requirement.
- Identify testable requirements.
- Clarify missing or ambiguous information.
- Identify positive and negative scenarios.
- Identify test data requirements.
- Discuss requirements with the Product Owner, Business Analyst and developers.

**Possible questions QA should ask:**

1. Can a customer add an out-of-stock product?
2. Can a guest user add products to the cart?
3. What happens if the customer adds the same product multiple times?
4. Is there a maximum quantity limit?
5. What happens if the product price changes after adding it to the cart?

**Output / Deliverables:**

- Requirement understanding.
- List of testable requirements.
- Requirement clarifications.
- Initial test scenarios.
- Requirement Traceability Matrix (RTM), as applicable.

Key point: QA should identify requirement gaps before testing begins.

### **Phase 2: Test Planning**

In this phase, the QA team decides how testing will be performed.

**Example:** We need to test the complete e-commerce checkout functionality.

**What does QA do?**

- Define testing scope.
- Identify testing types.
- Estimate testing effort.
- Assign resources and responsibilities.
- Decide testing tools.
- Define testing schedule.
- Identify risks and dependencies.
- Define entry and exit criteria.

**Example test planning decisions:**

| Area           | Decision                            |
| -------------- | ----------------------------------- |
| Scope          | Cart, checkout and payment          |
| Testing type   | Functional, integration, regression |
| Tool           | Playwright                          |
| Environment    | QA environment                      |
| Test data      | Valid and invalid customer accounts |
| Schedule       | 5 working days                      |
| Responsibility | Two QA engineers                    |

**Output / Deliverables:**

- Test Plan.
- Effort estimation.
- Resource allocation.
- Test schedule.
- Risk assessment.

Key point: Test planning defines what, how, when and who will test.

### **Phase 3: Test Case Development**

In this phase, QA converts requirements into test scenarios and detailed test cases.

**Example requirement:**

Customer should be able to add an available product to the cart.

**Test scenarios:**

1. Verify customer can add an available product.
2. Verify customer cannot add an out-of-stock product.
3. Verify correct product details appear in the cart.
4. Verify product quantity is updated correctly.
5. Verify the cart total is calculated correctly.

**Example test case:**

| Field           | Details                                              |
| --------------- | ---------------------------------------------------- |
| Test Case ID    | TC_001                                               |
| Scenario        | Add available product to cart                        |
| Precondition    | Customer is on product details page                  |
| Test Steps      | Click Add to Cart                                    |
| Expected Result | Product should be added to cart with correct details |

**What does QA do?**

- Write test scenarios.
- Prepare detailed test cases.
- Prepare test data.
- Review test cases.
- Map test cases to requirements.
- Identify positive and negative test cases.

**Output / Deliverables:**

- Test scenarios.
- Test cases.
- Test data.
- Updated RTM.

Key point: Convert requirements into executable test cases.

### **Phase 4: Test Environment Setup**

In this phase, QA prepares the environment required to execute test cases.

**Example:** We need a working e-commerce application in the QA environment.

**What does QA do?**

- Verify application deployment.
- Check database connectivity.
- Verify API availability.
- Configure test accounts.
- Prepare browser and device configurations.
- Configure required test data.
- Verify third-party integrations or suitable mocks.

**Example environment setup:**

| Component    | Example                   |
| ------------ | ------------------------- |
| Application  | E-commerce QA application |
| Browser      | Chrome                    |
| Database     | QA database               |
| API          | QA payment API            |
| Test account | Valid customer account    |
| Payment      | Test payment gateway      |

**Output / Deliverables:**

- Ready-to-use test environment.
- Configured test data.
- Environment readiness confirmation.

Key point: Make sure the testing environment is ready before execution.

### **Phase 5: Test Execution**

In this phase, QA executes the prepared test cases and compares actual results with expected results.

**Example:**

Test case: Add available product to cart.

| Step              | Expected Result           | Actual Result             | Status |
| ----------------- | ------------------------- | ------------------------- | ------ |
| Open product page | Product details displayed | Displayed correctly       | Pass   |
| Click Add to Cart | Product added             | Product added             | Pass   |
| Increase quantity | Quantity becomes 2        | Quantity remains 1        | Fail   |
| Check cart total  | Total reflects quantity 2 | Total reflects quantity 1 | Fail   |

**What does QA do?**

- Execute test cases.
- Compare actual and expected results.
- Mark test cases as Pass, Fail or Blocked.
- Log defects.
- Retest fixed defects.
- Perform regression testing.
- Update test execution reports.

**Output / Deliverables:**

- Test execution report.
- Defect reports.
- Updated test case status.
- Regression test results.

Key point: Execute test cases, identify defects and verify fixes.

### **Phase 6: Test Cycle Closure**

In this phase, QA evaluates the overall testing activities and prepares the final testing summary.

**Example:** We have completed testing of the e-commerce checkout feature.

**What does QA do?**

- Check whether testing objectives are achieved.
- Review executed and passed test cases.
- Review failed and blocked test cases.
- Analyse outstanding defects.
- Check requirement coverage.
- Prepare test summary report.
- Document lessons learned.
- Identify areas for improvement.

**Example test summary:**

| Metric                | Result |
| --------------------- | ------ |
| Total test cases      | 100    |
| Passed                | 85     |
| Failed                | 10     |
| Blocked               | 5      |
| Defects reported      | 12     |
| Critical open defects | 1      |

The team should communicate the remaining risks clearly. Test closure does not automatically mean the application is defect-free or ready for release.

**Output / Deliverables:**

- Test Summary Report.
- Defect summary.
- Test closure report.
- Lessons learned.

Key point: Evaluate testing results and communicate remaining risks.

## **4. SDLC vs STLC**

This is an important interview question.

| SDLC                                                               | STLC                                                                                                |
| ------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| Software Development Life Cycle                                    | Software Testing Life Cycle                                                                         |
| Focuses on software development                                    | Focuses on software testing                                                                         |
| Followed by the complete development team                          | Mainly followed by the QA/testing team                                                              |
| Includes requirement, design, development, testing and maintenance | Includes requirement analysis, planning, test development, environment setup, execution and closure |
| Goal is to develop and deliver software                            | Goal is to verify and validate software quality                                                     |

**Simple way to remember:**

SDLC = How we build software.

STLC = How we test software.

## **5. How STLC works in a real Agile project**

In an Agile project, STLC activities are not necessarily completed once for the entire application.

They are performed repeatedly for each feature or Sprint.

**Example: Two-week Sprint – Checkout feature**

1. **Requirement Analysis**

   QA reviews checkout user stories and acceptance criteria.
2. **Test Planning**

   QA identifies scope, risks and testing effort for checkout.
3. **Test Case Development**

   QA prepares scenarios for address, discount, tax and payment.
4. **Environment Setup**

   QA verifies the checkout build, test accounts and payment gateway.
5. **Test Execution**

   QA executes tests, logs defects and performs retesting and regression.
6. **Test Cycle Closure**

   QA shares test results and outstanding risks with the team.

In Agile, some of these activities overlap and happen continuously throughout the Sprint.

## **6. Interview questions and answers**

**Q1. What is STLC?**

STLC stands for Software Testing Life Cycle. It is a structured process followed by the QA team to perform testing activities. It includes requirement analysis, test planning, test case development, test environment setup, test execution and test cycle closure.

**Q2. What are the different phases of STLC?**

There are six commonly used phases:

1. Requirement Analysis.
2. Test Planning.
3. Test Case Development.
4. Test Environment Setup.
5. Test Execution.
6. Test Cycle Closure.

**Q3. What happens during requirement analysis?**

QA understands the requirements, identifies testable requirements, clarifies doubts, identifies possible test scenarios and checks whether requirements are complete and testable.

**Q4. What is the difference between test planning and test case development?**

Test planning defines the testing approach, scope, resources, effort, tools and schedule.

Test case development focuses on converting requirements into test scenarios, test cases and test data.

**Q5. What is a test environment?**

A test environment is the setup required to execute test cases. It includes the application, database, APIs, test data, browsers and other required configurations.

**Q6. What happens during test execution?**

QA executes test cases, compares actual results with expected results, reports defects, retests fixed defects and performs regression testing.

**Q7. What is test cycle closure?**

Test cycle closure is the final STLC phase where QA evaluates testing results, reviews defects and test coverage, prepares a test summary report and documents lessons learned.

**Q8. What is the difference between SDLC and STLC?**

SDLC focuses on the complete software development process, whereas STLC focuses specifically on the testing process. STLC is part of the broader SDLC.

## **7. Practical assignment – Map STLC to an e-commerce feature**

Imagine your team has received this requirement:

"Customers should be able to place an order using a valid credit card. After successful payment, the system should create an order and send an order confirmation."

Map the activities to the six STLC phases.

**STLC practical exercise**

1 / 6

**Phase 1: Requirement Analysis**

What questions would you ask about payment and order creation?

PreviousNext phase

You can save partial answers and return to previous phases before submitting.

## **8. Today's 60-minute schedule**

| Activity                           | Time   |
| ---------------------------------- | ------ |
| Understand STLC and its six phases | 15 min |
| E-commerce examples for each phase | 15 min |
| SDLC vs STLC                       | 5 min  |
| Interview questions                | 10 min |
| Practical assignment and revision  | 15 min |

## **9. One-minute revision**

- **Requirement Analysis:** Understand what needs to be tested.
- **Test Planning:** Decide how testing will be performed.
- **Test Case Development:** Prepare test cases and test data.
- **Test Environment Setup:** Prepare the environment for testing.
- **Test Execution:** Execute test cases and report defects.
- **Test Cycle Closure:** Review results and complete testing activities.

**Day 4 completion target:** You should be able to explain all six STLC phases with an e-commerce example and clearly differentiate SDLC from STLC.

Once you complete the practical assignment, I'll review your answers like an interviewer. Then we'll move to **Day 5 – Testing Levels: Unit, Integration, System and Acceptance Testing.**
