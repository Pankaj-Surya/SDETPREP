# **Week 1 – Day 2: Seven Principles of Software Testing**

Revision Notes

24 November 2026

Manual Testing

## **1. Introduction**

Software testing principles are basic guidelines that help testers perform testing effectively.

There are **seven fundamental principles of software testing**. These principles help us understand the limitations of testing and how to approach testing in real-world projects.

## **2. Seven Principles of Software Testing**

### **1. Testing shows the presence of defects, not their absence**

- Testing helps us identify defects in the application.
- Even if all test cases pass, we cannot guarantee that the application is defect-free.
- Testing increases confidence in the application but cannot prove that no defects exist.

**Example – E-commerce:**

All checkout test cases pass, but an international customer may still face an incorrect tax calculation.

**Remember:** Testing can show that defects exist, but cannot prove that defects do not exist.

### **2. Exhaustive testing is impossible**

- Testing every possible input, scenario, and combination is not practically possible.
- We have limited time and resources.
- Therefore, we should prioritize test cases based on risk, business impact, and priority.

**Example – E-commerce:**

A product purchase can involve:

- Different product quantities.
- Different currencies.
- Different discount coupons.
- Different delivery addresses.
- Different payment methods.

Testing every possible combination is impractical.

**Remember:** We cannot test everything, so we must prioritize testing.

### **3. Early testing saves time and money**

- Testing should start as early as possible in the SDLC.
- We should identify defects during requirement gathering and design instead of waiting until development is completed.
- Fixing defects early generally costs less than fixing them after release.

**Example – E-commerce:**

During requirement review, QA identifies that tax calculation rules for different countries are missing.

The team can clarify and fix the requirement before development starts.

**Remember:** Find defects early to reduce the cost of fixing them.

### **4. Defects cluster together**

- Most defects are generally found in a small number of modules.
- Some application modules are more complex and defect-prone than others.
- Testers should focus more attention on modules where defects are frequently found.

**Example – E-commerce:**

Suppose most defects are reported in:

- Checkout module.
- Payment module.

QA should spend more testing effort on these modules because they have a higher possibility of containing defects.

**Remember:** A small number of modules may contain most of the defects.

### **5. Pesticide paradox**

- Running the same test cases repeatedly may stop finding new defects.
- Existing test cases may not cover new scenarios or hidden defects.
- We should regularly review and update test cases and introduce new scenarios.

**Example – E-commerce:**

Initially, we test successful credit card payments repeatedly.

Later, we introduce new scenarios:

- Expired credit card.
- Invalid card details.
- Payment gateway failure.
- Payment timeout.
- Duplicate payment.
- Unsupported payment method.

These additional scenarios can help identify defects that existing test cases missed.

**Remember:** Don't keep repeating the same test cases. Improve and update them regularly.

### **6. Testing is context-dependent**

- Testing approach depends on the type of application.
- Different applications have different business requirements and risks.
- We cannot follow exactly the same testing approach for every application.

**Example:**

| E-commerce application    | Banking application              |
| ------------------------- | -------------------------------- |
| Add to cart               | Fund transfer                    |
| Product price calculation | Interest calculation             |
| Discount validation       | Account balance validation       |
| Order placement           | Transaction processing           |
| Payment gateway testing   | Security and transaction testing |

**Remember:** Testing depends on the application and its business requirements.

### **7. Absence-of-errors fallacy**

- Even if an application has very few defects, it does not mean the application is successful.
- The application must also meet business requirements and user expectations.
- Software that does not solve the user's problem is not useful, even if it works technically.

**Example – E-commerce:**

The checkout process works perfectly, but the application does not support a payment method commonly used by its target customers.

There may be no technical defect in checkout, but the application still fails to meet user expectations.

**Remember:** Defect-free software does not necessarily mean a successful product.

## **3. Quick Revision – All Seven Principles**

| No. | Principle                             | Simple meaning                       |
| --- | ------------------------------------- | ------------------------------------ |
| 1   | Testing shows the presence of defects | Testing cannot prove zero defects    |
| 2   | Exhaustive testing is impossible      | Cannot test everything               |
| 3   | Early testing saves time and money    | Start testing early                  |
| 4   | Defects cluster together              | Most defects may be in a few modules |
| 5   | Pesticide paradox                     | Update and improve test cases        |
| 6   | Testing is context-dependent          | Testing depends on the application   |
| 7   | Absence-of-errors fallacy             | Defect-free does not mean useful     |

## **4. Interview Questions and Answers**

**Q1. What are the seven principles of software testing?**

The seven principles are:

1. Testing shows the presence of defects, not their absence.
2. Exhaustive testing is impossible.
3. Early testing saves time and money.
4. Defects cluster together.
5. Pesticide paradox.
6. Testing is context-dependent.
7. Absence-of-errors fallacy.

**Q2. What is the pesticide paradox?**

Pesticide paradox means that if we keep executing the same test cases repeatedly, eventually those test cases may stop finding new defects.

To overcome this, we should review existing test cases and introduce new test scenarios.

**Q3. Why is exhaustive testing impossible?**

Exhaustive testing is impossible because an application can have many combinations of inputs, users, devices, browsers, and business scenarios.

Testing every possible combination requires unlimited time and resources. Therefore, we prioritize test cases based on risk and business impact.

**Q4. What is defect clustering?**

Defect clustering means most defects are generally found in a small number of modules.

For example, if checkout and payment modules have historically reported more defects, we should focus more testing effort on these modules.

**Q5. What is the absence-of-errors fallacy?**

It means that even if software has very few defects, it can still fail if it does not meet user expectations or business requirements.

For example, an e-commerce application may work perfectly but fail to support a payment method expected by its customers.

**Q6. Why should testing start early?**

Early testing helps identify defects during requirement gathering and design.

Fixing defects at an early stage generally takes less time and money compared to fixing them after development or production release.

**Q7. Can testing guarantee defect-free software?**

No. Testing can identify defects and increase confidence in software quality, but it cannot guarantee that the application is completely defect-free.

## **5. One-Minute Interview Revision**

If an interviewer asks you to explain the seven principles, you can say:

"Software testing has seven fundamental principles. First, testing shows the presence of defects but cannot prove their absence. Second, exhaustive testing is impossible, so we prioritize test cases. Third, early testing saves time and money because defects can be identified earlier. Fourth, defects cluster together, meaning most defects are generally found in a few modules. Fifth, pesticide paradox means we should regularly update our test cases instead of repeating the same tests. Sixth, testing is context-dependent, so our approach changes based on the application. Finally, absence-of-errors fallacy means even defect-free software can fail if it does not meet business requirements or user expectations."


0 of 6 completed

**Next topic – Day 3:** SDLC (Software Development Life Cycle): Waterfall, Agile, Scrum, and V-Model.

My recommendation: Before moving to Day 3, try answering the seven principles in your own words without looking at these notes. This will help prepare you for actual SDET interviews.
