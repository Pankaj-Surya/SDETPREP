# **Week 1 – Day 3: SDLC – Waterfall, Agile, Scrum & V-Model**

Day 3

25 November 2026

60 minutes

**Today's goal:** Understand how software is developed, where testing fits into the development process, and how different SDLC models work.

We'll use our e-commerce application as the common example throughout the lesson.

## **1. What is SDLC?**

**SDLC (Software Development Life Cycle)** is a step-by-step process followed by a team to develop and deliver software.

It defines how a software product is planned, designed, developed, tested, deployed, and maintained.

**Simple example:** Imagine we are building an e-commerce application where customers can search products, add items to a cart, make payments, and place orders.

The team needs a proper process to build and deliver this application. SDLC provides that process.

## **2. SDLC phases – Software Development Life Cycle**

A typical SDLC has six major phases.

## **SDLC lifecycle**

**1. Requirement Analysis**

Understand what the customer wants

**2. Planning**

Define scope, resources, cost and timeline

**3. Design**

Decide system architecture and technical design

**4. Development**

Develop the application based on requirements

**5. Testing**

Verify functionality and identify defects

**6. Deployment & Maintenance**

Release software and maintain it after release

Simplified lifecycle for learning. In real projects, activities can overlap and repeat.

### **Example: Building an e-commerce application**

| Phase                | What the team does                                                 |
| -------------------- | ------------------------------------------------------------------ |
| Requirement analysis | Understand product search, cart, checkout and payment requirements |
| Planning             | Estimate time, cost, resources and delivery                        |
| Design               | Design database, UI, APIs and architecture                         |
| Development          | Developers build the application                                   |
| Testing              | QA tests login, cart, checkout, payment and order creation         |
| Deployment           | Release the application to production                              |
| Maintenance          | Fix production defects and introduce enhancements                  |

**Where does QA come into the picture?**

QA should be involved from the requirement phase, not just after development.

For example:

- Requirement phase: Review requirements and identify missing scenarios.
- Design phase: Review workflows and identify design risks.
- Development phase: Prepare test cases and perform component testing.
- Testing phase: Execute functional, integration and regression tests.
- Production phase: Validate release and monitor reported issues.

**Interview point:** Testing is not only one phase at the end. Testing activities should start early and continue throughout the SDLC.

## **3. Waterfall model**

**Waterfall** is a traditional SDLC model where development activities happen sequentially, one phase after another.

A phase is generally completed before the next phase begins.

[What Is a Systems Development Life Cycle (SDLC)?](https://images.openai.com/static-rsc-4/8gRJ6ObTe9TYm5PDYw_YRsdgaD75YRfPV2ipjzwS2tYQNM1-AJTdichOAIOtdHmkq3C8JMK7akZIzB_7_Byvwxk_6srwOJ5e7Lf8WRLgwGHSaq9id-uljyg5frFFk2JYAzgNeL0SAdExExpHGGndYRCr0gecnF2_1pHewZrq1AI?purpose=inline)

[wallarm.com](https://www.wallarm.com/what/what-is-a-systems-development-life-cycle-sdlc?utm_source=chatgpt.com)

### **Example – E-commerce application**

Imagine a company wants to build an e-commerce application.

1. First, the business team finalizes all requirements.
2. The team completes the application design.
3. Developers develop the complete application.
4. QA receives the application and starts testing.
5. Once testing is completed, the application is deployed.

### **Advantages**

- Simple and easy to understand.
- Clear phases and documentation.
- Easy to manage when requirements are stable.
- Suitable for projects with well-defined requirements.

### **Disadvantages**

- Changes in requirements are difficult and costly.
- Testing happens relatively late in the process.
- Customer feedback comes late.
- Defects discovered late can require significant rework.

**Remember:** Waterfall = Complete one phase, then move to the next phase.

## **4. Agile model**

**Agile** is an iterative and incremental approach to software development.

Instead of developing the entire application in one long cycle, the team develops and delivers smaller pieces of functionality in iterations.

The team gets feedback regularly and improves the product continuously.

[Agile Compliance Management for QA/RA in Medical Device](https://images.openai.com/static-rsc-4/Tg17M4QyJ9Y1LV6BqcVwV3qBP9UkxNr2DPHdOSIHzy2qgyvb1v-omD7nkMnTf7dUVKmrscWw7V8ziaVObLVBqFV0DORmeBuoddKSzumAE32aN738OpJVudhjNE2cpBpazGncEYEe1ZjhwpBXl0rT0r-6A3dYxIRtBR2eL3CC7CA?purpose=inline)

[matrixone.health](https://matrixone.health/blog/agile-compliance-agile-software-development-explained-for-raqa?utm_source=chatgpt.com)

### **Example – E-commerce application**

Instead of building the entire application at once, the team divides the work into smaller features.

| Iteration   | Features                        |
| ----------- | ------------------------------- |
| Iteration 1 | User registration and login     |
| Iteration 2 | Product listing and search      |
| Iteration 3 | Add to cart and cart management |
| Iteration 4 | Checkout and payment            |
| Iteration 5 | Order history and notifications |

After each iteration, the team reviews the functionality and collects feedback.

### **Advantages**

- Faster delivery of working software.
- Easy to accommodate changing requirements.
- Continuous customer feedback.
- Testing happens continuously.
- Better collaboration between developers and testers.

### **Disadvantages**

- Requires continuous team communication.
- Frequent requirement changes can affect planning.
- Scope can become difficult to control.
- Requires active customer or product-owner involvement.

**Remember:** Agile = Build small, test, get feedback, improve, and repeat.

## **5. Scrum – An Agile framework**

An important interview point:

**Agile is an approach or set of values and principles. Scrum is a framework used to implement Agile.**

Scrum organizes work into fixed-length iterations called **Sprints**.

A Sprint is typically one to four weeks.

[¿Qué es Scrum? | Scrum.org](https://images.openai.com/static-rsc-4/mcWFsvEkXRhP8vLn9v8adbOu7cYnUFokC3X9ZU0LxVbQhO9ekRUEgZdI-HvMzaPKyRgme0MteYQCBKdACUI7-0GdIhKFpniJTnGfOVNSHGuXzwRfeFzgjHLqoRt0vf8K3lujBjQyU3kbmCADGO0J5H5jeXEFDVAq4G0DSnSjzY8?purpose=inline)

[scrum.org](https://www.scrum.org/resources/blog/que-es-scrum?utm_source=chatgpt.com)

### **Scrum roles**

| Role          | Responsibility                                                           |
| ------------- | ------------------------------------------------------------------------ |
| Product Owner | Manages Product Backlog and prioritizes product work                     |
| Scrum Master  | Helps the team understand Scrum and removes impediments                  |
| Developers    | Build the product increment; includes testing and other engineering work |

Note: QA engineers commonly work as part of the Developers group in Scrum.

### **Scrum events**

| Event                | Purpose                                             |
| -------------------- | --------------------------------------------------- |
| Sprint Planning      | Decide what work to complete in the Sprint          |
| Daily Scrum          | Inspect progress and plan the next day's work       |
| Sprint Review        | Demonstrate the completed work and collect feedback |
| Sprint Retrospective | Discuss how to improve the team's process           |

### **Scrum artifacts**

1. **Product Backlog:** Ordered list of everything that may be needed in the product.
2. **Sprint Backlog:** Selected work for the current Sprint, plus the plan to deliver it.
3. **Increment:** Usable product outcome created during the Sprint.

### **Example – E-commerce application**

Suppose the team is working on the cart feature in a two-week Sprint.

- Product Owner prioritizes the cart feature.
- During Sprint Planning, the team selects cart-related work.
- Developers implement add-to-cart functionality.
- QA tests adding products, changing quantities and removing products.
- During Daily Scrum, the team discusses progress and blockers.
- During Sprint Review, the team demonstrates the feature.
- During Retrospective, the team discusses what went well and what needs improvement.

**Remember:** Scrum = Agile framework + Sprints + Roles + Events + Artifacts.

## **6. V-Model (Verification and Validation Model)**

V-Model is a software development model where each development phase has a corresponding testing phase.

It is called the V-Model because the development activities are represented on the left side and corresponding testing activities on the right side of a V-shaped diagram.

[CSE 370 | Learning Activities](https://images.openai.com/static-rsc-4/ZEsQyQMfICoED0q30zuMnQxIM5O0hqHJzKmqsJsp72FTeoLwN-2tjzfb67Oip9nHtjm3NFjDtjW8KVffbkEv0N45hGc_87EW2Ys-XEgFFajXGgOPX-0x15PiCge4gReamg4DtdLfECxfPenKAGi6hCkkgjoc5E1tLf1RynSSsqM?purpose=inline)

[byui-cse.github.io](https://byui-cse.github.io/cse370-course/unit01/prepare.html?utm_source=chatgpt.com)

### **V-Model phases**

| Development phase                 | Corresponding testing phase |
| --------------------------------- | --------------------------- |
| Requirement analysis              | Acceptance testing          |
| System design                     | System testing              |
| Architectural / high-level design | Integration testing         |
| Module / low-level design         | Unit testing                |
| Coding                            | Implementation              |

### **Example – E-commerce application**

Suppose we are developing a checkout feature.

| Development activity                                                            | Testing activity                                             |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| Requirement: Customer must pay using a valid payment method                     | Acceptance testing checks the complete payment requirement   |
| System design: Design checkout workflow                                         | System testing validates the complete checkout               |
| Architecture design: Define communication between checkout and payment services | Integration testing validates communication between services |
| Module design: Design payment calculation and validation modules                | Unit testing validates individual functions                  |
| Coding: Developers implement the feature                                        | Code-level testing and reviews begin during development      |

### **Advantages**

- Testing activities are planned early.
- Clear relationship between development and testing.
- Helps identify missing requirements and test coverage gaps.
- Suitable for projects with stable and well-defined requirements.

### **Disadvantages**

- Changes can be costly.
- Less flexible when requirements change frequently.
- Working software and customer feedback may arrive late.

**Remember:** V-Model = Every development phase has a corresponding testing phase.

## **7. Waterfall vs Agile vs Scrum vs V-Model**

| Area                | Waterfall           | Agile                     | Scrum                                       | V-Model                               |
| ------------------- | ------------------- | ------------------------- | ------------------------------------------- | ------------------------------------- |
| Approach            | Sequential          | Iterative and incremental | Agile framework                             | Sequential with corresponding testing |
| Delivery            | Usually at the end  | Frequent increments       | Sprint increments                           | Usually after planned phases          |
| Testing             | Often later         | Continuous                | Within each Sprint                          | Planned alongside development         |
| Requirement changes | Difficult           | Flexible                  | Managed through backlog and Sprint planning | Difficult                             |
| Feedback            | Usually later       | Frequent                  | Sprint Review and ongoing collaboration     | Usually at defined stages             |
| Best suited for     | Stable requirements | Evolving requirements     | Teams using Scrum practices                 | Stable, well-defined requirements     |

**Important interview clarification:**

- Agile is not the same as Scrum.
- Scrum is one framework under Agile.
- Waterfall and V-Model are traditional sequential approaches.
- V-Model emphasizes corresponding testing activities for development phases.

## **8. How does a tester work in different models?**

This is particularly important for SDET interviews.

| Activity              | Waterfall                   | Agile / Scrum                                            | V-Model                               |
| --------------------- | --------------------------- | -------------------------------------------------------- | ------------------------------------- |
| Requirement review    | Before development          | During backlog refinement                                | During requirement analysis           |
| Test planning         | Usually early               | Sprint-wise and continuously                             | Alongside development planning        |
| Test case preparation | Before test execution phase | During the Sprint, often before or alongside development | Alongside corresponding design phases |
| Test execution        | Often after development     | Continuously within each Sprint                          | At corresponding testing stages       |
| Feedback              | Later                       | Frequent                                                 | At planned checkpoints                |

**Interview answer:**

"In Waterfall, QA generally gets involved in requirement analysis and prepares test cases, but execution often happens after development. In Agile, QA works closely with developers throughout the Sprint and testing is continuous. In V-Model, testing activities are planned alongside each development phase, so every development activity has a corresponding testing activity."

## **9. Interview questions and answers**

**Q1. What is SDLC?**

SDLC stands for Software Development Life Cycle. It is a structured process followed by a team to develop software. It includes requirement analysis, planning, design, development, testing, deployment and maintenance.

**Q2. What are the different SDLC models?**

Some commonly used models are:

- Waterfall
- Agile
- V-Model
- Iterative
- Spiral

**Q3. What is the difference between Waterfall and Agile?**

Waterfall follows a sequential approach where one phase is completed before the next begins. Agile follows an iterative and incremental approach where smaller features are developed, tested and delivered continuously.

**Q4. What is Scrum?**

Scrum is an Agile framework where teams work in fixed-length iterations called Sprints. It defines accountabilities, events and artifacts to help teams deliver product increments.

**Q5. What is a Sprint?**

A Sprint is a fixed-length period during which a Scrum team works to achieve a specific goal and deliver a usable product increment. It is typically one to four weeks.

**Q6. What is the difference between Agile and Scrum?**

Agile is a set of values and principles for software development. Scrum is a framework that helps teams follow Agile principles through Sprints, accountabilities, events and artifacts.

**Q7. What is the V-Model?**

V-Model is an SDLC model in which every development phase has a corresponding testing phase. Testing is planned early, and the model emphasizes verification and validation.

**Q8. Which SDLC model is used in your project?**

A good answer should be based on your actual project experience. For example, if your team uses Scrum:

"Our team follows Agile methodology using Scrum. We work in two-week Sprints. The Product Owner prioritizes the backlog, the team selects stories during Sprint Planning, and QA works with developers to test features during the Sprint. We conduct Sprint Reviews and Retrospectives to gather feedback and improve."

Don't claim to use a process that your actual project did not follow.

## **10. Practical assignment – Draw the SDLC lifecycle**

Today's practical task from our study plan is to draw the SDLC lifecycle.&#x20;

Pasted text(2).txt



Try drawing this flow without looking at the notes:

**Your practice task**

Complete the missing phases in the correct order.

Phase 1

Choose SDLC phase

Phase 2

Choose SDLC phase

Phase 3

Choose SDLC phase

Phase 4

Choose SDLC phase

Phase 5

Choose SDLC phase

Phase 6

Choose SDLC phase

Phase 7

Choose SDLC phase

Check my answers

## **11. Today's 60-minute schedule**

| Activity                             | Time   |
| ------------------------------------ | ------ |
| Understand SDLC and lifecycle phases | 10 min |
| Waterfall, Agile, Scrum and V-Model  | 15 min |
| Practical e-commerce examples        | 15 min |
| Interview questions                  | 10 min |
| Draw SDLC and revision               | 10 min |

## **12. One-minute revision**

- **SDLC:** Process used to develop software.
- **Waterfall:** Complete one phase before moving to the next.
- **Agile:** Develop and deliver software in smaller increments.
- **Scrum:** Agile framework that uses Sprints.
- **V-Model:** Every development phase has a corresponding testing phase.

**Day 3 completion target:** You should be able to draw the SDLC lifecycle, explain all four approaches, and clearly differentiate Agile from Scrum and Waterfall from V-Model.

When you're ready, share your answers to the interview questions or say **"Day 3 notes"** and I'll prepare a concise revision sheet in the same style as Day 2.
