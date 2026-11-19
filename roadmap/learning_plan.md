# **SDET Interview Preparation Roadmap – Product-Based Companies**

Starting **20 November 2026**, our objective is to prepare for SDET interviews at top product-based companies, focusing on strong fundamentals, hands-on automation engineering, problem-solving, system design, and modern AI-driven testing.

I would recommend approaching this as an **SDET Engineer preparation program, not just an automation testing course**.

Product companies generally expect an SDET to demonstrate five capabilities:

1. **Testing knowledge:** Can you identify what to test, how to test, and what can go wrong?
2. **Programming skills:** Can you write clean, maintainable JavaScript/TypeScript code and solve coding problems?
3. **Automation engineering:** Can you build a scalable UI and API automation framework?
4. **Engineering knowledge:** Can you understand CI/CD, cloud, databases, networking, Docker, and system design?
5. **Modern testing:** Can you use AI tools effectively while maintaining quality, security, and reliability?

## **1. Preparation structure**

START DATE

**20 November 2026**

DAILY COMMITMENT

**60 minutes**

PRIMARY LANGUAGE

**JavaScript + TypeScript**

AUTOMATION TOOL

**Playwright**

**Recommended duration: 20 weeks**

# **140+ hours**

Approximately 7 days × 60 minutes × 20 weeks

We will use weekly milestones, practical assignments, revision, and mock interviews rather than only completing theory.

### **Daily 60-minute structure**

| Duration | Activity                          |
| -------- | --------------------------------- |
| 10 min   | Theory and interview explanation  |
| 25 min   | Practical coding / implementation |
| 15 min   | Assignment or problem-solving     |
| 10 min   | Revision and interview questions  |

**Important:** Your proposed 11:00–11:40 QA slot can be used for the first 40 minutes, with the remaining 20 minutes dedicated to coding, assignments, or revision.

### **Overall learning sequence**

**Phase 1 – Foundation**

Manual QA · JavaScript · TypeScript · Git · MySQL

**Phase 2 – Automation Engineering**

Playwright UI · API · Framework design · Test data · Parallelism

**Phase 3 – Engineering Skills**

DSA · Design patterns · CI/CD · AWS · AI testing

**Phase 4 – Advanced Preparation**

System design · Linux · Networking · Docker · Performance · AI Evals

**Phase 5 – Interview Readiness**

Framework project · Coding rounds · Mock interviews

## **2. Detailed syllabus – topic by topic**

### **Module 1 – Manual QA / Testing Fundamentals**

Mandatory · High interview priority

**Objective:** Develop the ability to think like a QA engineer before writing automation scripts.

| Topic                | Detailed syllabus                                                                       |
| -------------------- | --------------------------------------------------------------------------------------- |
| Testing fundamentals | Why testing, verification vs validation, QA vs QC, defect vs failure vs error           |
| Testing principles   | Seven principles of software testing, early testing, exhaustive testing limitations     |
| SDLC                 | Waterfall, Agile, Scrum, V-model                                                        |
| STLC                 | Requirement analysis, test planning, test design, environment setup, execution, closure |
| Testing levels       | Unit, integration, system, acceptance                                                   |
| Testing types        | Functional, non-functional, regression, smoke, sanity, exploratory                      |
| Testing techniques   | BVA, Equivalence Partitioning, Decision Table, State Transition, Pairwise Testing       |
| Test documentation   | Test Plan, Test Strategy, Test Scenario, Test Case, RTM                                 |
| Defect management    | Bug lifecycle, severity vs priority, defect triage, root cause analysis                 |
| Agile QA             | Sprint planning, refinement, daily standup, retrospective, Definition of Done           |
| Risk-based testing   | Risk identification, impact, probability, test prioritization                           |
| Testing estimation   | Test effort estimation, coverage, exit criteria                                         |

**Theory – Interview preparation**

You should be able to explain:

- Difference between Test Strategy and Test Plan.
- How do you prepare test scenarios from requirements?
- What is RTM? How do you maintain traceability?
- Difference between smoke, sanity, and regression testing.
- How do you decide what to test when time is limited?
- How would you test an application without requirements?
- How do you handle a developer rejecting a bug?
- How do you identify test cases for a new feature?

**Practical assignments**

1. Take an e-commerce application.
2. Prepare test scenarios for login, registration, search, cart, checkout, and payment.
3. Write detailed test cases for login and checkout.
4. Create an RTM mapping requirements to test cases.
5. Prepare a Test Plan for the application.
6. Create a bug report with severity, priority, steps, expected and actual results.
7. Perform exploratory testing on a shopping cart.

**Interview task:** You are given a login page with username, password, CAPTCHA, and Forgot Password. Explain your test scenarios and prioritization.

### **Module 2 – JavaScript and TypeScript**

Mandatory · Very high priority

**Objective:** Become comfortable writing production-quality automation code, not just basic scripts.

#### **JavaScript syllabus**

| Topic            | Detailed concepts                                                        |
| ---------------- | ------------------------------------------------------------------------ |
| Fundamentals     | Variables, let, const, data types, operators, type coercion              |
| Functions        | Function declaration, expression, arrow functions, callbacks             |
| Scope            | Global, function, block scope, lexical scope                             |
| Closures         | Closure, lexical environment, practical use cases                        |
| Objects          | Object creation, destructuring, spread, rest, optional chaining          |
| Arrays           | map, filter, reduce, find, some, every, sort                             |
| Strings          | String methods, template literals, immutability                          |
| Async JavaScript | Synchronous vs asynchronous, event loop                                  |
| Promises         | Promise states, then, catch, finally, Promise.all, allSettled, race, any |
| Async/Await      | Error handling, sequential vs parallel execution                         |
| Error handling   | try/catch/finally, custom errors                                         |
| OOP              | Classes, objects, inheritance, encapsulation, polymorphism, abstraction  |
| Advanced JS      | Prototype, prototypal inheritance, this, bind, call, apply               |
| Modules          | CommonJS vs ES Modules, import/export                                    |
| Memory           | Stack, heap, garbage collection basics                                   |

#### **TypeScript syllabus**

| Topic             | Detailed concepts                                  |
| ----------------- | -------------------------------------------------- |
| Basics            | Type annotations, inference, primitive types       |
| Type vs Interface | Extending, declaration merging, use cases          |
| Advanced types    | Union, intersection, literal types                 |
| Functions         | Parameter types, return types, optional parameters |
| Objects           | Optional properties, readonly, index signatures    |
| Generics          | Generic functions, classes, constraints            |
| Utility types     | Partial, Pick, Omit, Required, Record, Readonly    |
| Narrowing         | typeof, instanceof, discriminated unions           |
| Enums             | Numeric and string enums                           |
| Advanced          | keyof, typeof, mapped types, conditional types     |
| Configuration     | tsconfig, strict mode, compiler options            |

**Theory – Interview preparation**

- Explain the JavaScript event loop with an example.
- What is the difference between Promise.all and Promise.allSettled?
- What happens when we use await inside a loop?
- Explain closures and their practical use.
- Difference between == and ===.
- Explain map, filter, and reduce.
- What is the difference between null and undefined?
- Explain the difference between type and interface.
- What are generics and where would you use them?
- How does JavaScript handle asynchronous operations?
- Explain shallow copy vs deep copy.

**Practical assignments**

1. Write a function to remove duplicate elements from an array.
2. Find the second-largest number in an array.
3. Group an array of objects by a property.
4. Flatten a nested array.
5. Implement a retry function using Promise.
6. Execute five asynchronous API calls in parallel.
7. Implement sequential API calls.
8. Create a generic API response type in TypeScript.
9. Build a reusable wait/retry utility.
10. Create a TypeScript class for User, Product, and Order.

**SDET-specific practical task**

Build a small reusable utility library:

```
utils/
  arrayUtils.ts
  stringUtils.ts
  retry.ts
  wait.ts
  logger.ts
  apiResponse.ts
```

The goal is to understand how we write reusable utilities in a real automation framework.

### **Module 3 – Playwright UI Automation**

Mandatory · Highest priority

This should be one of our strongest areas because product companies expect SDETs to build and maintain automation frameworks.

#### **A. Playwright fundamentals**

- Playwright architecture.
- Browser, BrowserContext, Page.
- Chromium, Firefox, WebKit.
- Headless vs headed execution.
- Locators and locator strategies.
- getByRole, getByText, getByLabel, getByTestId.
- CSS and XPath selectors.
- Locator chaining and filtering.
- Auto-waiting and actionability.
- Assertions and expect.
- Web-first assertions.
- Handling flaky tests.
- Timeout management.
- Screenshots and videos.
- Trace viewer.
- Debugging with Playwright Inspector.

#### **B. Advanced UI automation**

| Topic            | Detailed syllabus                                                 |
| ---------------- | ----------------------------------------------------------------- |
| Browser handling | Multiple browsers, tabs, popups, new pages                        |
| Frames           | iframe, frameLocator                                              |
| Windows          | Multiple tabs and browser contexts                                |
| Dialogs          | Alert, confirm, prompt                                            |
| Network          | Request, response, route, intercept, abort                        |
| Authentication   | Storage state, session reuse, login setup                         |
| Cookies          | Cookies, localStorage, sessionStorage                             |
| Files            | Upload, download, file chooser                                    |
| UI elements      | Dropdown, checkbox, radio, calendar, autocomplete                 |
| Tables           | Static and dynamic web tables                                     |
| Shadow DOM       | Shadow DOM interaction                                            |
| Visual testing   | Screenshot comparison, visual regression                          |
| Accessibility    | Accessibility snapshots, role-based testing, accessibility checks |
| Mobile           | Device emulation, viewport, touch                                 |
| WebSockets       | Basic real-time application testing                               |

#### **C. Playwright framework design**

This is particularly important for senior SDET interviews.

- Playwright Test Runner architecture.
- Fixtures and fixture inheritance.
- Page Object Model (POM).
- Component Object Model.
- Custom fixtures.
- Hooks: beforeAll, beforeEach, afterEach, afterAll.
- Test annotations and tagging.
- Test.describe and test.step.
- Data-driven testing.
- Reusable assertions.
- Custom reporters.
- Logging and reporting.
- Test configuration.
- Environment-specific configuration.
- Framework folder structure.
- Reusable services and utilities.
- SOLID principles in automation.
- Design patterns in automation.

**Suggested framework structure**

```
playwright-framework/
│
├── tests/
│   ├── ui/
│   ├── api/
│   └── integration/
│
├── pages/
├── components/
├── fixtures/
├── services/
├── clients/
├── test-data/
├── schemas/
├── utils/
├── config/
├── reports/
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

#### **D. Environment management – Important practical topic**

Learn how to execute the same automation framework across different environments.

| Environment | Example             |
| ----------- | ------------------- |
| Local       | localhost:3000      |
| QA          | qa.example.com      |
| Staging     | staging.example.com |
| Production  | app.example.com     |

Learn:

- Environment variables.
- `.env` files.
- dotenv.
- baseURL configuration.
- Environment-specific test data.
- Environment-specific credentials.
- Secrets management.
- Configuration validation.
- Avoiding hardcoded URLs and credentials.
- How to run tests using environment variables.

Example:

```
// config/env.ts

export const config = {
  baseURL: process.env.BASE_URL,
  username: process.env.TEST_USERNAME,
  password: process.env.TEST_PASSWORD
};
```

Practical task: Execute the same test against QA and Staging by changing the environment configuration, without modifying the test script.

#### **E. Test data management – Important practical topic**

This is often overlooked but very important in real-world automation.

Learn:

- Static vs dynamic test data.
- Test data factories.
- Faker library.
- Unique test data generation.
- Test data creation using API.
- Test data cleanup.
- Database-driven test data.
- JSON, CSV, and Excel data.
- Data isolation between tests.
- Shared data vs independent data.
- Test data seeding.
- Test data versioning.
- Sensitive data handling.
- Avoiding test data dependencies.

Example:

```
import { faker } from '@faker-js/faker';

const user = {
  name: faker.person.fullName(),
  email: faker.internet.email(),
  password: 'Test@123'
};
```

Practical task:

- Create 100 unique users.
- Register them using API.
- Use those users in UI tests.
- Clean up created data after execution.
- Ensure parallel tests do not use the same account.

#### **F. Parallel execution and sharding**

Very important for product companies

Understand the difference between:

- Sequential execution.
- Parallel execution.
- Worker processes.
- Fully parallel vs serial execution.
- Test isolation.
- Shared state problems.
- Worker-scoped and test-scoped fixtures.
- Worker-specific test data.
- Resource contention.
- Flaky tests caused by parallel execution.
- Sharding across CI machines.
- Load balancing.
- Combining shard reports.

Example:

```
# Run tests using 4 workers
npx playwright test --workers=4

# Run first shard
npx playwright test --shard=1/4

# Run second shard
npx playwright test --shard=2/4
```

**Interview question:** If 1,000 tests take 60 minutes to execute sequentially, how would you reduce execution time?

You should explain worker parallelism, CI machines, sharding, test isolation, data independence, and resource limitations.

#### **G. Advanced execution and reliability**

- Retries and retry strategy.
- Flaky test detection.
- Quarantine strategy.
- Test tagging.
- Smoke and regression suites.
- Selective test execution.
- Test dependency management.
- Global setup and teardown.
- Authentication reuse.
- CI-friendly reports.
- Failure artifacts.
- Test execution optimization.
- Test pyramid and test distribution.

**Practical project**

Build a complete e-commerce UI automation framework covering:

1. Login and registration.
2. Product search.
3. Product details.
4. Add to cart.
5. Checkout.
6. Payment simulation.
7. Order history.

The framework must support QA and Staging, multiple browsers, parallel execution, sharding, reporting, and dynamic test data.

### **Module 4 – Playwright API Automation**

Mandatory · Highest priority

Our goal should be to become equally strong in API and UI automation.

#### **A. API testing fundamentals**

- What is an API?
- Client-server architecture.
- REST architecture.
- HTTP methods: GET, POST, PUT, PATCH, DELETE.
- HTTP request and response.
- Headers, query parameters, path parameters.
- Request body.
- Response body.
- HTTP status codes.
- Authentication vs authorization.
- Statelessness.
- Idempotency.
- REST vs SOAP.
- JSON and XML.
- API versioning.
- Pagination, filtering, sorting.
- Rate limiting.

#### **B. API testing techniques**

- Positive and negative testing.
- Boundary testing.
- Schema validation.
- Contract testing.
- Data validation.
- Response time validation.
- Authentication testing.
- Authorization testing.
- Error handling.
- Security testing basics.
- API chaining.
- CRUD testing.
- Idempotency testing.
- Pagination testing.
- Rate-limit testing.

#### **C. Playwright API automation**

- APIRequestContext.
- request fixture.
- APIRequest.newContext.
- GET, POST, PUT, PATCH, DELETE.
- Request headers.
- Query parameters.
- Authentication tokens.
- Response validation.
- JSON schema validation.
- API fixtures.
- API client abstraction.
- Reusable API methods.
- API setup and teardown.
- API reporting.
- API/UI integration.

Example:

```
import { test, expect } from '@playwright/test';

test('Create user', async ({ request }) => {
  const response = await request.post('/users', {
    data: {
      name: 'Test User',
      email: 'test@example.com'
    }
  });

  expect(response.status()).toBe(201);

  const body = await response.json();

  expect(body.name).toBe('Test User');
});
```

#### **D. Advanced API automation scenarios**

| Topic               | Practical scenario                                 |
| ------------------- | -------------------------------------------------- |
| API chaining        | Create user → Login → Create order                 |
| Authentication      | Generate and reuse access token                    |
| Authorization       | Verify user cannot access another user's order     |
| Data validation     | Validate response against schema                   |
| Negative testing    | Invalid payload and missing fields                 |
| Idempotency         | Repeat payment request with same idempotency key   |
| Pagination          | Validate page size and next-page behavior          |
| Contract testing    | Verify API contract compatibility                  |
| Database validation | Verify API-created data in MySQL                   |
| API/UI integration  | Create product through API and purchase through UI |
| Cleanup             | Delete test data through API                       |
| Parallel execution  | Isolate data for concurrent API tests              |

**Practical project**

Build a reusable API automation framework for an e-commerce backend.

```
api-framework/
│
├── tests/
├── clients/
├── services/
├── fixtures/
├── schemas/
├── test-data/
├── utils/
├── config/
└── playwright.config.ts
```

**Interview tasks**

1. How would you test a login API?
2. How do you validate a 401 vs 403 response?
3. How do you handle token expiration?
4. How would you test an API that creates duplicate orders?
5. How do you test pagination?
6. How do you validate API response against MySQL?
7. How would you automate an API workflow with 10 dependent APIs?
8. How would you handle rate limiting?
9. How do you test idempotency?
10. How do you design a reusable API client?

### **Module 5 – Basic DSA and Problem Solving**

Mandatory · High priority

Product-based companies can ask coding questions even for SDET roles. We should target easy and medium-level problems.

| Topic          | Concepts and tasks                              |
| -------------- | ----------------------------------------------- |
| Complexity     | Big O, time and space complexity                |
| Arrays         | Reverse, duplicates, missing number, two sum    |
| Strings        | Palindrome, anagram, character frequency        |
| HashMap        | Frequency count, grouping, lookup               |
| Sets           | Unique elements, intersection, union            |
| Two pointers   | Pair sum, reverse array, sorted arrays          |
| Sliding window | Longest substring, maximum sum                  |
| Stack          | Valid parentheses, next greater element         |
| Queue          | Basic queue, BFS introduction                   |
| Sorting        | Bubble, selection, insertion, merge sort basics |
| Searching      | Linear search, binary search                    |
| Recursion      | Factorial, Fibonacci, tree traversal basics     |
| Linked list    | Reverse, cycle detection basics                 |
| Trees          | Traversal, height, BST basics                   |

**SDET-specific coding exercises**

- Find duplicate test cases.
- Compare two JSON objects.
- Find missing test IDs.
- Group test results by status.
- Find the most frequently failing test.
- Merge two test execution reports.
- Retry failed test cases.
- Find tests taking longer than a threshold.
- Compare expected and actual API response objects.

**Target:** Around 50–70 carefully selected coding problems, with emphasis on explaining complexity and edge cases.

### **Module 6 – Design Patterns and SOLID Principles**

Mandatory · High priority

We should learn patterns from an automation framework perspective, not just memorize definitions.

#### **SOLID principles**

Use one consistent credit-card example to explain all five principles, following this interview storytelling flow:

Requirement → problematic code → future-change problem → gradual refactoring → principle → trade-off.

| Principle | What to learn         | Automation example                                                         |
| --------- | --------------------- | -------------------------------------------------------------------------- |
| SRP       | Single Responsibility | Separate login, reporting, and test-data responsibilities                  |
| OCP       | Open/Closed           | Add new browser or reporting support without modifying existing core logic |
| LSP       | Liskov Substitution   | Browser implementations must respect their common contract                 |
| ISP       | Interface Segregation | Split large automation interfaces into smaller capabilities                |
| DIP       | Dependency Inversion  | Tests depend on abstractions rather than concrete services                 |

#### **Design patterns**

| Pattern              | Practical use                               |
| -------------------- | ------------------------------------------- |
| Singleton            | Shared configuration or controlled resource |
| Factory              | Browser or API client creation              |
| Builder              | Complex test data and request payloads      |
| Strategy             | Different authentication mechanisms         |
| Adapter              | Integrating external APIs                   |
| Decorator            | Adding logging or retry behavior            |
| Observer             | Event-driven notifications                  |
| Facade               | Simplifying complex automation operations   |
| Template Method      | Common test workflow                        |
| Dependency Injection | Fixture-based dependency management         |
| Page Object Model    | UI page abstraction                         |

**Practical task:** Design an automation framework using Factory, Strategy, Builder, Facade, and Dependency Injection.

**Interview preparation:** Be able to explain why you selected a particular pattern, what problem it solves, and when it would be unnecessary.

### **Module 7 – MySQL and Database Testing**

Mandatory · High priority

**Theory**

- Relational database fundamentals.
- Tables, rows, columns.
- Primary key and foreign key.
- Constraints.
- Normalization.
- Relationships: one-to-one, one-to-many, many-to-many.
- Indexes.
- Transactions.
- ACID properties.
- Joins.
- Views.
- Stored procedures and triggers basics.

**SQL syllabus**

| Category     | Topics                                  |
| ------------ | --------------------------------------- |
| Basic        | SELECT, INSERT, UPDATE, DELETE          |
| Filtering    | WHERE, LIKE, IN, BETWEEN                |
| Sorting      | ORDER BY, LIMIT                         |
| Aggregation  | COUNT, SUM, AVG, MIN, MAX               |
| Grouping     | GROUP BY, HAVING                        |
| Joins        | INNER, LEFT, RIGHT, CROSS               |
| Subqueries   | Nested queries, correlated subqueries   |
| Advanced     | CTE, window functions, ROW_NUMBER, RANK |
| Transactions | COMMIT, ROLLBACK                        |
| Performance  | Indexes, EXPLAIN, query optimization    |

**Database testing practicals**

1. Validate data after user registration.
2. Verify order records after checkout.
3. Validate payment status.
4. Verify foreign-key relationships.
5. Check duplicate records.
6. Validate data consistency between API and DB.
7. Test rollback scenarios.
8. Find orphan records.
9. Verify data cleanup.

**SDET practical task:** Create a user through API, verify the record in MySQL, update the user through UI, and verify the updated database record.

### **Module 8 – Git and GitHub**

Mandatory · Medium priority

**Theory**

- Git vs GitHub.
- Repository.
- Working directory, staging area, local repository.
- Branching.
- Merge vs rebase.
- Pull request.
- Merge conflicts.
- Commit history.
- Tags and releases.
- .gitignore.
- Git workflow.
- Code review.

**Commands to master**

```
git init
git clone
git status
git add
git commit
git push
git pull
git fetch
git branch
git checkout
git switch
git merge
git rebase
git stash
git log
git reset
git revert
git cherry-pick
```

**Practical task**

- Create a Playwright repository.
- Create feature branches.
- Work on separate UI and API test modules.
- Raise pull requests.
- Resolve merge conflicts.
- Review code.
- Merge changes into main.

### **Module 9 – CI/CD Pipeline**

Mandatory · Very high priority

**Theory**

- What is CI/CD?
- Continuous Integration vs Continuous Delivery vs Continuous Deployment.
- Build pipeline.
- Pipeline stages.
- Build artifacts.
- Environment variables.
- Secrets.
- Pipeline triggers.
- Scheduled execution.
- Quality gates.
- Test reports.
- Failure handling.
- Pipeline optimization.
- Deployment strategies: blue-green, rolling, canary.
- GitHub Actions.
- Jenkins fundamentals.

**Practical syllabus**

Build a GitHub Actions pipeline:

```
Developer Push
      |
      v
Checkout Code
      |
      v
Install Dependencies
      |
      v
Install Playwright Browsers
      |
      v
Lint + Type Check
      |
      v
Run API Tests
      |
      v
Run UI Tests
      |
      v
Parallel / Sharded Execution
      |
      v
Merge Reports
      |
      v
Publish Test Results
```

Learn to implement:

- UI and API test execution.
- QA/Staging environment selection.
- Parallel workers.
- Sharding across CI jobs.
- Scheduled regression execution.
- Smoke tests on pull requests.
- Retry and failure handling.
- HTML report publishing.
- Trace and screenshot artifacts.
- Pipeline notifications.
- Secure credential management.

**Interview questions**

- How would you integrate Playwright with Jenkins?
- How do you execute tests on every PR?
- How do you execute 2,000 tests within a limited time?
- How do you handle flaky tests in CI?
- What happens if a test fails in a pipeline?
- How do you separate smoke and regression pipelines?

### **Module 10 – AWS Cloud**

Mandatory · Medium-high priority

Focus on concepts that an SDET needs rather than becoming an AWS administrator.

| Service / Concept  | Syllabus                                      |
| ------------------ | --------------------------------------------- |
| Cloud fundamentals | IaaS, PaaS, SaaS, regions, availability zones |
| IAM                | Users, roles, policies, least privilege       |
| EC2                | Virtual machines, instances, security groups  |
| S3                 | Buckets, objects, artifact storage            |
| VPC                | Subnets, route tables, networking basics      |
| Lambda             | Serverless execution                          |
| RDS                | Managed relational database                   |
| CloudWatch         | Logs, metrics, alarms                         |
| ECS                | Container orchestration basics                |
| ECR                | Docker image registry                         |
| Secrets Manager    | Credential management                         |
| SQS                | Message queues                                |
| Load balancing     | ALB, health checks                            |

**Practical tasks**

1. Store Playwright reports in S3.
2. Understand how tests can execute on EC2.
3. Understand how to securely manage credentials.
4. Read application logs from CloudWatch.
5. Understand how to run containerized tests.
6. Learn the architecture of a distributed test execution environment.

### **Module 11 – AI for SDET**

Mandatory · Emerging high-priority area

This deserves dedicated preparation because AI-assisted development and testing are becoming important engineering skills.

#### **A. AI coding assistants**

- Claude Code.
- GitHub Copilot.
- Agent-based coding.
- Prompt engineering.
- Context management.
- Codebase exploration.
- Planning vs execution.
- Multi-file code generation.
- Debugging with AI.
- Test-case generation.
- Code review using AI.
- Refactoring using AI.
- Human-in-the-loop development.

#### **B. Playwright MCP vs CLI**

Understand:

- What is MCP?
- What is an MCP server?
- MCP client and server architecture.
- Playwright MCP.
- Playwright CLI.
- Browser automation through MCP.
- Browser automation through scripts.
- When to use MCP vs CLI.
- Limitations of AI-driven browser automation.
- Deterministic automation vs agent-driven automation.

#### **C. AI agents**

- What is an AI agent?
- Agent vs chatbot.
- Tool calling.
- Planning and execution.
- Agent memory.
- Context window.
- Multi-agent workflows.
- Agent orchestration.
- Human approval.
- Failure recovery.
- Observability.

#### **D. Skills and guardrails**

- What are AI Skills?
- Reusable skill instructions.
- Skill structure.
- Project-specific skills.
- Tool permissions.
- Guardrails.
- Prompt injection.
- Sensitive data protection.
- Human approval before destructive operations.
- Preventing unauthorized production changes.
- Limiting agent scope.
- Validation of generated code.
- AI output verification.

**Practical tasks**

1. Use Claude Code to explore an existing Playwright framework.
2. Generate test cases from a requirement.
3. Ask an AI assistant to create UI automation scripts.
4. Use Playwright MCP to explore a website.
5. Compare the same task using Playwright CLI and MCP.
6. Create a reusable testing skill.
7. Add guardrails to prevent deletion of production data.
8. Use AI to analyze flaky test reports.
9. Review AI-generated test cases for missing coverage.

**Interview question:** Would you allow an AI agent to execute tests directly against production? Explain the guardrails you would implement.

## **3. Advanced syllabus**

These topics will help differentiate an SDET with automation experience from an SDET with strong software engineering knowledge.

### **Module 12 – System Design: LLD and HLD**

Advanced · Very high priority for senior SDET

We should prepare both traditional software system design and test automation architecture.

#### **A. Low-Level Design (LLD)**

**Core concepts**

- Object-oriented design.
- SOLID principles.
- Interfaces and abstract classes.
- Composition vs inheritance.
- Encapsulation.
- Dependency injection.
- Design patterns.
- UML class diagrams.
- Sequence diagrams.
- State diagrams.
- Class relationships.
- Extensibility and maintainability.

**Design problems to practice**

| Problem             | What to design                     |
| ------------------- | ---------------------------------- |
| Parking lot         | Vehicles, slots, floors, tickets   |
| ATM                 | Accounts, transactions, withdrawal |
| Elevator            | Requests, scheduling, movement     |
| Library management  | Books, members, borrowing          |
| Payment system      | Payment methods, transactions      |
| Rate limiter        | Request limits, time windows       |
| Notification system | Email, SMS, push                   |
| Test framework      | Test runner, test cases, reporters |
| Test data manager   | Data creation, isolation, cleanup  |

**SDET-specific LLD problems**

- Design a reusable API automation framework.
- Design a test data management library.
- Design a retry mechanism.
- Design a parallel test execution manager.
- Design a test reporting system.
- Design a browser factory.
- Design a configuration manager.

#### **B. High-Level Design (HLD)**

**Core concepts**

- Client-server architecture.
- Monolithic vs microservices.
- Load balancing.
- Horizontal vs vertical scaling.
- Caching.
- Database scaling.
- SQL vs NoSQL.
- Message queues.
- Event-driven architecture.
- API gateway.
- CDN.
- Service discovery.
- Fault tolerance.
- High availability.
- Replication.
- Sharding.
- CAP theorem.
- Consistency models.
- Rate limiting.
- Observability.
- Distributed systems.

**Important interview topics**

- TCP vs UDP.
- HTTP vs HTTPS.
- HTTP/1.1 vs HTTP/2 vs HTTP/3.
- DNS resolution.
- TCP handshake.
- IP address and port.
- Stateless vs stateful systems.
- Synchronous vs asynchronous communication.

**HLD problems**

1. Design URL Shortener.
2. Design WhatsApp.
3. Design Instagram.
4. Design an e-commerce application.
5. Design a payment gateway.
6. Design a notification service.
7. Design a distributed test execution platform.
8. Design a centralized test reporting system.
9. Design a scalable test data service.

**SDET interview angle:** How would you test a distributed application containing 20 microservices? Discuss service-level tests, contract testing, integration testing, test environments, observability, and failure isolation.

### **Module 13 – Linux, Networking, Operating Systems and Shell Scripting**

Advanced · High priority

#### **A. Linux**

- Linux architecture basics.
- Filesystem structure.
- File permissions.
- Users and groups.
- Process management.
- Environment variables.
- File operations.
- Searching and filtering.
- Logs.
- Package management.
- SSH.
- Cron jobs.
- Disk and memory monitoring.

**Commands to practice**

```
pwd
ls
cd
mkdir
touch
cat
less
grep
find
awk
sed
head
tail
chmod
chown
ps
top
kill
df
du
free
curl
wget
ssh
```

#### **B. Networking**

- OSI model.
- TCP/IP model.
- TCP vs UDP.
- DNS.
- DHCP.
- IP address.
- Public vs private IP.
- Ports.
- Sockets.
- HTTP and HTTPS.
- SSL/TLS handshake.
- Proxy.
- Reverse proxy.
- Firewall.
- Load balancer.
- NAT.
- WebSocket.
- REST communication.
- Network troubleshooting.

**Practical commands**

```
ping
curl
nslookup
dig
traceroute
netstat
ss
lsof
ip
telnet
nc
```

#### **C. Operating Systems**

- Process vs thread.
- Multithreading.
- Concurrency vs parallelism.
- Race conditions.
- Deadlocks.
- Mutex and semaphore.
- Context switching.
- CPU scheduling.
- Memory management.
- Stack vs heap.
- Virtual memory.
- Synchronization.

#### **D. Shell scripting**

- Variables.
- Input/output.
- Conditions.
- Loops.
- Functions.
- Exit codes.
- Arguments.
- Environment variables.
- File processing.
- Scheduling scripts.
- Error handling.

**Practical tasks**

1. Write a script to execute Playwright tests.
2. Find failed tests from a report.
3. Extract error messages from logs.
4. Kill a process using a port.
5. Check whether a service is available.
6. Monitor a test execution process.
7. Create a shell script that runs smoke tests and stores reports.

### **Module 14 – Performance Testing**

Advanced · High priority

**Theory**

- Performance testing fundamentals.
- Load testing.
- Stress testing.
- Spike testing.
- Soak testing.
- Scalability testing.
- Capacity testing.
- Response time.
- Throughput.
- Latency.
- Concurrent users.
- Requests per second.
- Percentiles: P50, P90, P95, P99.
- Bottleneck identification.
- CPU and memory utilization.
- Database performance.
- Network latency.
- Performance baselines.

**Tools**

- k6.
- JMeter.
- Grafana.
- Prometheus.

**Practical tasks**

1. Write a simple k6 script.
2. Execute load tests against an API.
3. Increase virtual users gradually.
4. Compare P95 response time.
5. Identify performance degradation.
6. Generate performance reports.
7. Understand how to correlate API latency with application and database metrics.

**Interview questions**

- Difference between load and stress testing?
- What is the difference between throughput and latency?
- Why is P99 important?
- How do you identify an API bottleneck?
- How would you test an API handling 10,000 requests per second?

### **Module 15 – Docker and Containerization**

Advanced · High priority

**Theory**

- Virtual machines vs containers.
- Containerization.
- Docker architecture.
- Docker engine.
- Images vs containers.
- Dockerfile.
- Docker Hub.
- Docker registry.
- Volumes.
- Networks.
- Docker Compose.
- Container lifecycle.
- Port mapping.
- Environment variables.
- Container logs.
- Multi-stage builds.
- Resource limits.
- Container health checks.

**Practical tasks**

1. Write a Dockerfile for Playwright.
2. Build an automation Docker image.
3. Run UI tests inside a container.
4. Run API tests inside a container.
5. Configure environment variables.
6. Mount test reports as volumes.
7. Use Docker Compose for an application and its database.
8. Execute tests in a CI pipeline using Docker.
9. Run multiple test containers.

**Interview question:** Why would you run Playwright tests inside Docker instead of directly on a CI server?

### **Module 16 – AI Evals**

Advanced · Emerging priority

This is different from traditional functional testing. We need to understand how to evaluate the quality and reliability of AI-powered applications.

**Theory**

- What are LLMs?
- LLM application architecture.
- AI evaluation fundamentals.
- Deterministic vs non-deterministic output.
- Hallucination.
- Prompt injection.
- Context relevance.
- Groundedness.
- Faithfulness.
- Relevance.
- Toxicity.
- Bias.
- Accuracy.
- Precision and recall.
- LLM-as-a-judge.
- Human evaluation.
- Golden datasets.
- Evaluation datasets.
- Regression evaluation.
- RAG evaluation.
- Agent evaluation.
- Tool-calling evaluation.
- Safety and security evaluation.

**Evaluation frameworks and tools to explore**

- DeepEval.
- Ragas.
- LangSmith.
- Promptfoo.

**Practical task**

Assume we have an AI customer support chatbot.

Create 50 test questions and expected evaluation criteria.

Evaluate:

1. Correctness of answer.
2. Relevance of answer.
3. Hallucination.
4. Response consistency.
5. Prompt injection resistance.
6. Sensitive data leakage.
7. Incorrect tool invocation.
8. Latency and token consumption.

Build an evaluation report that identifies failures and compares two versions of the chatbot.

### **Module 17 – Crio Advanced Topics**

Advanced · Depends on your course curriculum

I would keep this as an integration and problem-solving module instead of treating it as another isolated theory subject.

Recommended areas:

- Advanced JavaScript/TypeScript problem solving.
- Advanced Playwright framework architecture.
- Real-world API automation.
- Framework optimization.
- Advanced design patterns.
- LLD and HLD problem solving.
- Distributed systems.
- Coding assignments.
- Debugging complex automation failures.
- Engineering trade-offs.

We should map the exact Crio modules to our syllabus once the course material is available, avoiding duplicate study.

## **4. Recommended 20-week study calendar**

Here is the sequence I recommend. It prioritizes Playwright, programming, and testing fundamentals while ensuring that advanced engineering topics are not skipped.

Proposed calendar starting 20 November 2026. Each week has seven 60-minute study sessions.

| Week | Dates        | Primary focus                               |
| ---- | ------------ | ------------------------------------------- |
| 1    | Nov 20–26    | Manual QA fundamentals + SDLC/STLC          |
| 2    | Nov 27–Dec 3 | Test design + documentation + bug lifecycle |
| 3    | Dec 4–10     | JavaScript fundamentals                     |
| 4    | Dec 11–17    | Advanced JavaScript + async                 |
| 5    | Dec 18–24    | TypeScript + OOP                            |
| 6    | Dec 25–31    | Playwright fundamentals + locators          |
| 7    | Jan 1–7      | Advanced UI automation                      |
| 8    | Jan 8–14     | POM + fixtures + framework architecture     |
| 9    | Jan 15–21    | Environment + test data management          |
| 10   | Jan 22–28    | Parallelism + sharding + reliability        |
| 11   | Jan 29–Feb 4 | API testing fundamentals + Playwright API   |
| 12   | Feb 5–11     | Advanced API + API/UI integration           |
| 13   | Feb 12–18    | MySQL + database testing + Git              |
| 14   | Feb 19–25    | DSA + SOLID + design patterns               |
| 15   | Feb 26–Mar 4 | CI/CD + GitHub Actions + Jenkins            |
| 16   | Mar 5–11     | AWS + AI tools + MCP/CLI                    |
| 17   | Mar 12–18    | LLD + automation framework design           |
| 18   | Mar 19–25    | HLD + networking + Linux + OS               |
| 19   | Mar 26–Apr 1 | Docker + performance testing + AI Evals     |
| 20   | Apr 2–8      | Crio advanced + projects + mock interviews  |

Note: This is a first-pass calendar. Some topics will continue through revision and practical assignments after their primary week.

## **5. Weekly learning methodology**

We should not spend all seven days watching lectures or reading documentation.

Use this approach every week:

| Day   | Focus                    | Expected output                |
| ----- | ------------------------ | ------------------------------ |
| Day 1 | Theory                   | Understand concepts            |
| Day 2 | Interview preparation    | Explain concepts without notes |
| Day 3 | Practical implementation | Write code                     |
| Day 4 | Advanced practical       | Handle real-world scenarios    |
| Day 5 | Assignment               | Solve an independent problem   |
| Day 6 | Interview questions      | Answer 10–15 questions         |
| Day 7 | Revision + assessment    | Weekly test and review         |

**Weekly assessment criteria**

- Can I explain the concept without looking at notes?
- Can I implement it independently?
- Can I explain why I selected this approach?
- Can I identify edge cases?
- Can I debug a failure?
- Can I explain the trade-offs?

If the answer is no, revisit the topic rather than moving forward just to complete the calendar.

## **6. Three end-to-end projects (must-have)**

I strongly recommend building projects instead of completing isolated assignments. These projects will become talking points in interviews.

[Belo.Fur – Furniture E-commerce Website | Add to Cart Flow by Subash Chandra for Seative Digital on Dribbble](https://images.openai.com/static-rsc-4/Tiqkob_KzL-qk40uA-I9P6joVhSR600POqUp2gERrYAxJbKAFRuk9Nwn3ZF-5MtH0WBIx4SShJ0TseJ1Jq2MeIw144MyHUQrWvb9TbPOz4Eh83B-C9LzEZpVZgDxZXelMeAYW2PjBpo8gnMHGPFqvMJmm-7ldAcUYG1b5iNsEsc?purpose=inline)

**Project 1 – E-commerce Automation Framework**

Mandatory

- Playwright UI + API automation.
- POM, fixtures, SOLID and design patterns.
- Environment and test data management.
- Parallel execution and sharding.
- CI/CD and reporting.

[Self-Host Your API Testing with Hoppscotch: The open-source API Platform | by Kiran Johns | Medium](https://images.openai.com/static-rsc-4/73sNdkKpcsAiGvDzU-2g_oyMKnx69MEnR6qcrotSU9Ltqzw5-3quprp6B5l9AZzqUVo3HMo2uLCKflnzyCMOkX5VbO4H4ft9IeyZq3-jp7xcc1jcrIul4VPmhx-wyInKTNd569c2MQ316vdgHIaOUtRiXL_mkNWkkVm0rMqHH1s?purpose=inline)

**Project 2 – Enterprise API Testing Framework**

Mandatory

- Reusable API client.
- Authentication and authorization.
- Schema and contract validation.
- Database integration.
- API chaining and test data cleanup.
- Parallel execution.

[Alethia | AI-native E2E testing. Let agents test. Not break. | vitron.ai](https://images.openai.com/static-rsc-4/rS2FqObvuBV9FIo1_--IHAkS5dWxQgxwvMKdyb1_JKW_z-Q6Di4murKR90YUbuw8kR86hFFRAKpCM7-j-spp9qv_z3EWbVkEkwDFwW-HYLy-YUrgRD5CD_xsO1pv_Cw-1xeCOG3bAXY9g9A6K037p3STNPDgAphI_ynxOXNGe0w?purpose=inline)

**Project 3 – AI-Powered Testing Assistant**

Advanced

- Requirement-to-test-case generation.
- AI-assisted Playwright code generation.
- MCP browser exploration.
- Automated failure analysis.
- AI evaluation and guardrails.
- Human approval workflow.

## **7. Topics we should not ignore**

I would add the following topics to your original list because they are important for product-company SDET interviews.

| Additional topic      | Why it matters                                 |
| --------------------- | ---------------------------------------------- |
| API security testing  | Authentication, authorization, OWASP API risks |
| Contract testing      | Essential for microservices                    |
| Accessibility testing | Important for UI quality                       |
| Visual regression     | Useful for UI automation                       |
| Exploratory testing   | Demonstrates testing thought process           |
| Observability         | Logs, metrics, traces                          |
| Microservices testing | Service integration and failure isolation      |
| GitHub Actions        | Practical CI/CD implementation                 |
| Test architecture     | Framework scalability and maintainability      |
| Debugging             | Real-world engineering ability                 |
| Code review           | Understanding code quality                     |
| Flaky test management | Critical in large automation suites            |

## **8. Interview preparation priority**

Not every topic deserves equal time. This is how I would prioritize our preparation.

**Tier 1 – Must be very strong**

Highest

JavaScript/TypeScript, Playwright UI, Playwright API, Manual QA, Framework Design, API Testing, DSA basics.

**Tier 2 – Strong working knowledge**

High

SQL, SOLID, Design Patterns, Git, CI/CD, Parallelism, Sharding, Test Data Management, LLD, Networking.

**Tier 3 – Working knowledge**

Advanced

AWS, Docker, Linux, OS, HLD, Performance Testing, AI Evals, AI Agents, MCP and CLI.

One important recommendation: DSA, JavaScript coding, and interview explanation should continue throughout all 20 weeks. Don't restrict them to their dedicated weeks.

## **9. How we will track our progress**

We should maintain a topic-wise tracker. Here is a simple interactive tracker to start with.

**SDET preparation tracker**

# **13%**

3 of 24 topics completed

Manual QA

JavaScript

TypeScript

Playwright UI

Playwright API

Framework Design

Environment Management

Test Data Management

Parallelism and Sharding

DSA

SOLID and Design Patterns

MySQL

Git and GitHub

CI/CD

AWS

AI Tools and Agents

LLD and HLD

Linux, Networking and OS

Docker

Performance Testing

AI Evals

Crio Advanced

End-to-end Projects

Mock Interviews

Reset tracker

This is a session-level tracker. Use a persistent spreadsheet or project board for long-term progress.

## **10. My recommendations before we start**

There are three things I want us to follow throughout the preparation:

**1. Theory must be interview-ready.**

For every topic, prepare:

- Simple definition.
- Why do we need it?
- How does it work?
- Real-world example.
- Advantages and disadvantages.
- Interview questions.

**2. Practical must be production-oriented.**

For every automation topic, we should understand how it works in a real project, not just how to write a simple script.

For example, don't just learn Playwright workers. Understand how workers affect test data, database state, authentication, shared resources, and CI execution.

**3. Every week should produce something tangible.**

A coding assignment, framework feature, diagram, SQL exercise, design document, or interview explanation.

## **11. Let's finalize our preparation schedule**

There is one detail I want to clarify before creating the detailed day-wise syllabus.

You mentioned both 60 minutes daily and an 11:00–11:40 QA slot.

**1. Daily study time**

60 minutes total (11:00–12:00)

40 minutes QA (11:00–11:40) + 20 minutes coding/revision

60 minutes QA + additional coding time

**2. Syllabus format**

Day-wise (Day 1, Day 2, Day 3...)

Week-wise with detailed daily assignments

**3. Target duration**

20 weeks (recommended initial plan)

24 weeks (more practice and revision)

Flexible – continue until interview-ready

Finalize preparation plan

**My expectation:** By the end of this preparation, you should be able to independently design a Playwright UI + API automation framework, explain its architecture and engineering decisions, solve coding problems, debug real-world failures, and confidently handle SDET technical interviews at product-based companies.
