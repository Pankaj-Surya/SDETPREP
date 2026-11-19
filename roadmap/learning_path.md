# **SDET Master Preparation Plan – Product-Based Companies**



Preparation plan finalized

Starting 20 November 2026



Our preparation will follow a simple rule:

**We will prepare to become an SDET who can design, code, automate, debug, and explain a complete testing solution independently.**

We will not rush to finish topics. The plan will be milestone-based, and we will move forward when the practical and interview-readiness criteria are met.

## **1. Final preparation strategy**

START DATE

**20 November 2026**

DAILY TIME

**60 minutes**

STUDY FORMAT

**Day-wise**

DURATION

**Flexible**

# **40 + 20 minutes**

40 min – QA / Theory / Practical

20 min – Coding / Revision

### **How each day will work**

| Time        | What we will do                    |
| ----------- | ---------------------------------- |
| 11:00–11:10 | Theory and concept understanding   |
| 11:10–11:25 | Practical implementation           |
| 11:25–11:40 | Interview explanation and scenario |
| 11:40–12:00 | Coding problem or revision         |

The 40-minute session will not always be theory. On practical-heavy days, most of those 40 minutes will be spent writing code.

### **Rules we will follow**

1. Every topic starts with fundamentals and progresses to advanced concepts.
2. Every concept must have a real-world SDET example.
3. We will use TypeScript as our default language, with JavaScript fundamentals covered first.
4. Playwright will be our primary UI and API automation tool.
5. We will use one e-commerce application as our continuous example.
6. Every week will have an assessment.
7. We will maintain notes, coding exercises, interview answers, and project deliverables.
8. We will revise previously covered topics every week.

# **2. Complete day-wise curriculum**

The curriculum is organized into 20 planned weeks. This is a learning sequence, not a deadline. A difficult week can take two weeks, and a familiar week may take less.

Calendar starts Friday, 20 November 2026. Each week contains seven sessions.

## **Phase 1 – Testing and programming foundations**

Foundation

### **Week 1 – Manual QA fundamentals**

**Goal:** Think like a tester before thinking like an automation engineer.

| Day        | 40-minute QA / theory / practical                                                    | 20-minute coding / revision                        |
| ---------- | ------------------------------------------------------------------------------------ | -------------------------------------------------- |
| 1 – Nov 20 | What is software testing? QA vs QC, verification vs validation, error/defect/failure | Write 5 real-world software defect examples        |
| 2 – Nov 21 | Seven testing principles, why exhaustive testing is impossible                       | Explain all seven principles in your own words     |
| 3 – Nov 22 | SDLC: Waterfall, Agile, Scrum, V-model                                               | Draw SDLC lifecycle                                |
| 4 – Nov 23 | STLC: requirement analysis, planning, design, execution, closure                     | Map STLC to an e-commerce feature                  |
| 5 – Nov 24 | Testing levels: unit, integration, system, acceptance                                | Identify testing levels for checkout               |
| 6 – Nov 25 | Testing types: functional, regression, smoke, sanity, exploratory, non-functional    | Prepare 15 examples of testing types               |
| 7 – Nov 26 | Weekly revision and assessment                                                       | Explain QA vs QC, SDLC vs STLC, and testing levels |

**Interview questions**

- What is the difference between QA and QC?
- Explain SDLC and STLC.
- What is verification vs validation?
- Can testing prove that software is defect-free?
- Difference between smoke and sanity testing?
- When would you perform regression testing?

**Weekly deliverable:** One-page testing fundamentals notes and an SDLC/STLC diagram.

### **Week 2 – Test design and documentation**

**Goal:** Convert requirements into meaningful test scenarios and test cases.

| Day | 40-minute QA / theory / practical                             | 20-minute coding / revision                  |
| --- | ------------------------------------------------------------- | -------------------------------------------- |
| 1   | Test Plan vs Test Strategy, contents and ownership            | Draft a mini test strategy                   |
| 2   | Test scenarios vs test cases, positive and negative scenarios | Write 10 login scenarios                     |
| 3   | Equivalence Partitioning                                      | Create input partitions for age and password |
| 4   | Boundary Value Analysis                                       | Find boundary test cases for amount 1–10,000 |
| 5   | Decision Table and State Transition Testing                   | Create a payment-status transition table     |
| 6   | RTM, test coverage, bug lifecycle, severity vs priority       | Prepare an RTM and one bug report            |
| 7   | Assessment: test a login and checkout feature                 | Present your test design in interview format |

**Interview questions**

- What is RTM and why is it needed?
- How do you derive test cases from requirements?
- Explain BVA with an example.
- Severity vs priority?
- What happens when requirements change?
- How do you prioritize testing with limited time?

**Weekly deliverable:** Test Plan, RTM, test cases, and bug reports for login and checkout.

### **Week 3 – JavaScript fundamentals**

**Goal:** Build programming fundamentals for automation.

| Day | 40-minute QA / theory / practical                     | 20-minute coding / revision     |
| --- | ----------------------------------------------------- | ------------------------------- |
| 1   | Variables, let, const, var, data types, type coercion | Reverse a string                |
| 2   | Operators, conditions, loops                          | Find even and odd numbers       |
| 3   | Functions, arrow functions, callbacks                 | Find factorial                  |
| 4   | Arrays, objects, destructuring, spread/rest           | Remove duplicate array elements |
| 5   | map, filter, reduce, find, some, every                | Group users by role             |
| 6   | Scope, hoisting, closures, lexical environment        | Write a closure counter         |
| 7   | Weekly JavaScript assessment                          | Solve 3 problems without help   |

**Interview questions**

- var vs let vs const?
- What is hoisting?
- What is a closure?
- What is lexical scope?
- map vs forEach?
- filter vs find?
- What is the difference between shallow and deep copy?

**Weekly deliverable:** JavaScript fundamentals exercise repository.

### **Week 4 – Advanced JavaScript and asynchronous programming**

**Goal:** Understand asynchronous execution, which is critical for Playwright.

| Day | 40-minute QA / theory / practical                 | 20-minute coding / revision                    |
| --- | ------------------------------------------------- | ---------------------------------------------- |
| 1   | Call stack, event loop, microtasks and macrotasks | Predict execution order                        |
| 2   | Promise states, then, catch, finally              | Create a Promise                               |
| 3   | async/await and error handling                    | Convert Promise chain to async/await           |
| 4   | Promise.all, allSettled, race, any                | Execute 5 API-like promises in parallel        |
| 5   | Sequential vs parallel execution                  | Compare sequential and parallel duration       |
| 6   | this, bind, call, apply, prototypes               | Implement a simple class and prototype example |
| 7   | Async JavaScript assessment                       | Explain event loop and solve 3 async exercises |

**Interview questions**

- Explain the event loop.
- What is the difference between Promise and async/await?
- Promise.all vs Promise.allSettled?
- How do you handle errors in async functions?
- Why does await inside a loop execute sequentially?
- What is a race condition?

**Weekly deliverable:** Reusable retry, timeout, and asynchronous utility functions.

### **Week 5 – TypeScript and object-oriented programming**

**Goal:** Write strongly typed and maintainable automation code.

| Day | 40-minute QA / theory / practical                       | 20-minute coding / revision             |
| --- | ------------------------------------------------------- | --------------------------------------- |
| 1   | Type annotations, inference, primitive types, functions | Convert JS functions into TS            |
| 2   | Type vs interface, union and intersection               | Model User and Admin types              |
| 3   | Classes, constructors, access modifiers, inheritance    | Implement User and Admin classes        |
| 4   | Encapsulation, abstraction, polymorphism, composition   | Build a payment-method example          |
| 5   | Generics, constraints, utility types                    | Create generic API response type        |
| 6   | Narrowing, keyof, typeof, enums, strict mode, tsconfig  | Create a typed API response validator   |
| 7   | Weekly assessment                                       | Build a small TypeScript utility module |

**Interview questions**

- Type vs interface?
- What are generics?
- What is the difference between abstract class and interface?
- Explain composition vs inheritance.
- What is type narrowing?
- What are Partial, Pick and Omit?

**Weekly deliverable:** A reusable TypeScript utilities library.

## **Phase 2 – Playwright automation engineering**

Core SDET skills

### **Week 6 – Playwright UI fundamentals**

**Goal:** Automate real browser interactions.

| Day | 40-minute QA / theory / practical                             | 20-minute coding / revision       |
| --- | ------------------------------------------------------------- | --------------------------------- |
| 1   | Playwright architecture, browser, context, page, installation | Create first test                 |
| 2   | Locators: role, text, label, placeholder, test ID             | Automate login                    |
| 3   | CSS, XPath, locator chaining, filtering                       | Automate dynamic product search   |
| 4   | Auto-waiting, actionability, assertions                       | Automate form validation          |
| 5   | Hooks, test.describe, test.step, test annotations             | Write structured tests            |
| 6   | Browser selection, headed/headless, debugging                 | Run tests in Chromium and Firefox |
| 7   | Assessment: automate login and search                         | Explain locator strategy          |

**Interview questions**

- What is BrowserContext?
- Difference between Browser and Page?
- Why prefer getByRole over XPath?
- What is auto-waiting?
- What is the difference between locator and elementHandle?
- How do you handle flaky tests?

**Weekly deliverable:** First Playwright UI test suite.

### **Week 7 – Advanced Playwright UI**

| Day | 40-minute QA / theory / practical                     | 20-minute coding / revision |
| --- | ----------------------------------------------------- | --------------------------- |
| 1   | Multiple tabs, popups, browser contexts               | Automate new-tab workflow   |
| 2   | Frames and iframe handling                            | Automate iframe interaction |
| 3   | Dialogs, alerts, dropdowns, calendars                 | Automate complex form       |
| 4   | Uploads, downloads, file chooser                      | Automate file upload        |
| 5   | Network requests, responses, route interception       | Mock an API response        |
| 6   | Cookies, localStorage, sessionStorage, authentication | Save and reuse login state  |
| 7   | Assessment: complete product purchase workflow        | Debug a failing test        |

**Interview questions**

- How do you handle multiple tabs?
- How do you interact with iframe?
- How do you mock API responses?
- How do you manage authentication?
- How do you handle file upload and download?

**Weekly deliverable:** Advanced UI automation examples.

### **Week 8 – Playwright framework architecture**

| Day | 40-minute QA / theory / practical                 | 20-minute coding / revision           |
| --- | ------------------------------------------------- | ------------------------------------- |
| 1   | Framework architecture, POM and component objects | Create LoginPage                      |
| 2   | Fixtures, built-in and custom fixtures            | Create custom page fixture            |
| 3   | Hooks, setup and teardown, global setup           | Implement authentication setup        |
| 4   | Reusable assertions, utilities, test annotations  | Create assertion utility              |
| 5   | Reporter, logging, screenshots, videos, traces    | Generate HTML report                  |
| 6   | Folder structure, SOLID, dependency injection     | Refactor test code                    |
| 7   | Assessment: framework architecture review         | Explain framework design in interview |

**Interview questions**

- Why use POM?
- What are fixtures?
- Fixture vs hooks?
- How do you avoid duplicate code?
- How do you design a scalable framework?
- How would you implement dependency injection?

**Weekly deliverable:** Initial reusable UI automation framework.

### **Week 9 – Environment and test data management**

Critical real-world framework topic

| Day | 40-minute QA / theory / practical                       | 20-minute coding / revision                  |
| --- | ------------------------------------------------------- | -------------------------------------------- |
| 1   | Environment management, local/QA/staging/prod           | Create environment config                    |
| 2   | .env, dotenv, secrets, config validation                | Run same test in two environments            |
| 3   | Static vs dynamic test data, Faker                      | Generate unique users                        |
| 4   | Test data factory, builders, JSON/CSV data              | Build data factory                           |
| 5   | API-based test data creation and cleanup                | Create and delete test user                  |
| 6   | Data isolation, test independence, worker-specific data | Generate independent data for parallel tests |
| 7   | Assessment: design test data strategy                   | Explain test data architecture               |

**Interview questions**

- How do you manage multiple environments?
- Where do you store credentials?
- How do you generate unique test data?
- How do you avoid shared test data problems?
- How do you clean up test data?
- How do you manage test data in parallel execution?

**Weekly deliverable:** Environment manager and test data factory.

### **Week 10 – Parallel execution, sharding and reliability**

Critical product-company topic

| Day | 40-minute QA / theory / practical                   | 20-minute coding / revision         |
| --- | --------------------------------------------------- | ----------------------------------- |
| 1   | Sequential vs parallel execution, workers           | Run tests using multiple workers    |
| 2   | Fully parallel, serial execution, test isolation    | Find shared-state problems          |
| 3   | Worker fixtures vs test fixtures                    | Create worker-specific resources    |
| 4   | Sharding, distributed execution                     | Execute four shards                 |
| 5   | Merging reports, retries, flaky test management     | Generate merged report              |
| 6   | Test tagging, smoke/regression, selective execution | Build tagged suites                 |
| 7   | Assessment: optimize 1,000-test suite               | Explain parallel execution strategy |

**Interview questions**

- Workers vs shards?
- How does Playwright execute tests in parallel?
- Why do tests fail only in parallel execution?
- How do you manage shared test data?
- How do you merge sharded reports?
- How do you identify and manage flaky tests?

**Weekly deliverable:** Framework supporting parallelism, sharding, reporting, and independent test data.

### **Week 11 – API testing fundamentals and Playwright API**

| Day | 40-minute QA / theory / practical                          | 20-minute coding / revision |
| --- | ---------------------------------------------------------- | --------------------------- |
| 1   | API fundamentals, REST, client-server architecture         | Identify API methods        |
| 2   | HTTP methods, status codes, headers, query/path parameters | Write HTTP request examples |
| 3   | Authentication, authorization, JWT, OAuth basics           | Generate and use token      |
| 4   | Playwright APIRequestContext, request fixture              | Automate GET and POST       |
| 5   | PUT, PATCH, DELETE, request payloads                       | Automate CRUD               |
| 6   | Response validation, JSON schema, negative testing         | Validate response schema    |
| 7   | Assessment: automate user CRUD APIs                        | Explain API test strategy   |

**Interview questions**

- PUT vs PATCH?
- 401 vs 403?
- Authentication vs authorization?
- What is statelessness?
- What is idempotency?
- How do you validate an API response?

### **Week 12 – Advanced API automation**

| Day | 40-minute QA / theory / practical                | 20-minute coding / revision        |
| --- | ------------------------------------------------ | ---------------------------------- |
| 1   | API chaining and dependent APIs                  | Create user → login → create order |
| 2   | Reusable API clients and service abstraction     | Build API client                   |
| 3   | API authentication reuse and token expiry        | Implement token manager            |
| 4   | Pagination, sorting, filtering, rate limits      | Test pagination                    |
| 5   | Contract testing and compatibility               | Validate contract                  |
| 6   | API/UI integration, database validation, cleanup | Create via API, verify via UI      |
| 7   | Assessment: complete e-commerce API flow         | Explain API framework architecture |

**Interview questions**

- How do you handle token expiry?
- How do you test idempotency?
- How do you validate pagination?
- What is contract testing?
- How do you test APIs across microservices?
- How do you avoid test data dependency?

**Weekly deliverable:** Reusable Playwright API framework.

## **Phase 3 – Software engineering and infrastructure**

Engineering skills

### **Week 13 – MySQL and database testing**

| Day | 40-minute QA / theory / practical                  | 20-minute coding / revision      |
| --- | -------------------------------------------------- | -------------------------------- |
| 1   | Relational DB, tables, primary and foreign keys    | Create sample tables             |
| 2   | SELECT, WHERE, ORDER BY, LIMIT, filtering          | Write 10 SQL queries             |
| 3   | INSERT, UPDATE, DELETE, constraints                | CRUD queries                     |
| 4   | INNER, LEFT, RIGHT, CROSS JOIN                     | Solve join exercises             |
| 5   | GROUP BY, HAVING, aggregate functions, subqueries  | Find duplicate records           |
| 6   | Transactions, ACID, indexes, CTE, window functions | Solve SQL interview problems     |
| 7   | Assessment: database testing of e-commerce         | Validate API-created order in DB |

**Interview questions**

- What are ACID properties?
- Primary key vs unique key?
- DELETE vs TRUNCATE vs DROP?
- INNER JOIN vs LEFT JOIN?
- What is an index?
- What is normalization?
- How do you find the second-highest salary?
- How do you identify duplicate records?

**Weekly deliverable:** SQL practice repository and database validation utilities.

### **Week 14 – Git, DSA, SOLID and design patterns**

This is a dense week. If coding or design-pattern understanding is weak, split it into two weeks.

| Day | 40-minute QA / theory / practical                          | 20-minute coding / revision                |
| --- | ---------------------------------------------------------- | ------------------------------------------ |
| 1   | Git fundamentals, branch, commit, merge, rebase            | Create feature branch and merge            |
| 2   | Merge conflicts, PR, code review, revert vs reset          | Resolve a conflict                         |
| 3   | Big O, arrays, strings, HashMap, Set                       | Two Sum and frequency count                |
| 4   | Two pointers, sliding window, stack, queue                 | Valid parentheses                          |
| 5   | SOLID using credit card example and automation framework   | Identify SOLID violations                  |
| 6   | Factory, Strategy, Builder, Singleton, Adapter, Facade, DI | Implement Factory and Strategy             |
| 7   | Assessment: code review and design discussion              | Solve 2 DSA problems and explain a pattern |

**Interview questions**

- Merge vs rebase?
- Reset vs revert?
- Explain time and space complexity.
- What is SOLID?
- Explain OCP with a real-world example.
- Factory vs Strategy?
- Why should we avoid unnecessary design patterns?

**Weekly deliverable:** GitHub repository with clean commits and a small SOLID-based automation utility.

### **Week 15 – CI/CD and GitHub Actions**

| Day | 40-minute QA / theory / practical                    | 20-minute coding / revision     |
| --- | ---------------------------------------------------- | ------------------------------- |
| 1   | CI, CD, build pipelines, pipeline stages             | Draw CI/CD architecture         |
| 2   | GitHub Actions workflow, events, jobs, steps         | Create first workflow           |
| 3   | Dependencies, browser installation, caching          | Run Playwright tests in CI      |
| 4   | Environment variables, secrets, QA/Staging selection | Configure environment selection |
| 5   | Parallel jobs, matrix strategy, sharding             | Run 4 shards                    |
| 6   | Artifacts, reports, traces, failure notifications    | Publish test report             |
| 7   | Assessment: design complete regression pipeline      | Explain pipeline in interview   |

**Interview questions**

- CI vs CD?
- What is a pipeline?
- What are jobs and steps in GitHub Actions?
- How do you execute tests on every pull request?
- How do you manage secrets?
- How do you execute tests across multiple machines?
- How do you publish reports when a test fails?

**Weekly deliverable:** Complete Playwright UI + API GitHub Actions pipeline.

### **Week 16 – AWS Cloud and AI-assisted development**

| Day | 40-minute QA / theory / practical                           | 20-minute coding / revision                        |
| --- | ----------------------------------------------------------- | -------------------------------------------------- |
| 1   | Cloud fundamentals, regions, AZs, IAM, EC2                  | Draw AWS testing architecture                      |
| 2   | S3, RDS, VPC, security groups                               | Design report storage in S3                        |
| 3   | Lambda, CloudWatch, ECS, ECR, Secrets Manager               | Explore logs and secret management                 |
| 4   | Claude Code, Copilot, agentic coding, context and prompts   | Ask AI to explain a codebase                       |
| 5   | Playwright MCP vs CLI, MCP client/server architecture       | Explore a website using MCP and write a CLI script |
| 6   | AI agents, Skills, tool calling, guardrails, human approval | Create a testing skill with safe boundaries        |
| 7   | Assessment: AI-assisted test creation and AWS architecture  | Explain MCP vs CLI and AWS services                |

**Interview questions**

- What is IAM and why is least privilege important?
- EC2 vs Lambda?
- S3 vs RDS?
- What is CloudWatch?
- What is MCP?
- MCP vs CLI?
- Agent vs chatbot?
- What is a guardrail?
- How do you prevent an AI agent from modifying production data?

**Weekly deliverable:** AI-assisted testing workflow and AWS architecture diagram.

## **Phase 4 – Advanced engineering**

Advanced preparation

### **Week 17 – Low-Level Design (LLD)**

| Day | 40-minute QA / theory / practical                           | 20-minute coding / revision          |
| --- | ----------------------------------------------------------- | ------------------------------------ |
| 1   | OOP design, class relationships, composition vs inheritance | Design User and Order classes        |
| 2   | UML, class diagrams, sequence diagrams                      | Draw payment sequence diagram        |
| 3   | Parking lot design                                          | Implement core classes               |
| 4   | ATM and payment system design                               | Implement payment abstraction        |
| 5   | Notification system using Strategy and Factory              | Implement notification factory       |
| 6   | Design a test framework and test data manager               | Implement small framework components |
| 7   | LLD mock interview                                          | Explain design and trade-offs        |

**Interview questions**

- How do you approach an LLD problem?
- Composition vs inheritance?
- Abstract class vs interface?
- How do you apply SOLID in your design?
- How would you design a test execution engine?
- How would you make a framework extensible?

**Weekly deliverable:** LLD design documents and working code for two design problems.

### **Week 18 – HLD, networking, Linux and OS**

This is another dense week. Give yourself additional time if necessary.

| Day | 40-minute QA / theory / practical                           | 20-minute coding / revision                      |
| --- | ----------------------------------------------------------- | ------------------------------------------------ |
| 1   | HLD fundamentals, scalability, availability, load balancing | Draw URL shortener architecture                  |
| 2   | Caching, DB replication, sharding, CAP theorem              | Design scalable test reporting                   |
| 3   | Microservices, API gateway, message queues, observability   | Draw microservices testing architecture          |
| 4   | Networking: DNS, TCP/UDP, HTTP/HTTPS, TLS, OSI              | Troubleshoot using curl and nslookup             |
| 5   | Linux: filesystem, permissions, processes, logs             | Practice grep, find, tail, chmod                 |
| 6   | OS: process/thread, concurrency, deadlocks, memory          | Write a simple shell script                      |
| 7   | Assessment: system design and troubleshooting               | Explain request lifecycle from browser to server |

**Interview questions**

- What happens when you enter a URL in a browser?
- TCP vs UDP?
- Explain DNS resolution.
- What is a reverse proxy?
- Process vs thread?
- What is a deadlock?
- Horizontal vs vertical scaling?
- SQL replication vs sharding?
- How would you investigate a slow API?

**Weekly deliverable:** One HLD diagram, one networking explanation, and one Linux troubleshooting script.

### **Week 19 – Docker, performance testing and AI Evals**

| Day | 40-minute QA / theory / practical                                 | 20-minute coding / revision         |
| --- | ----------------------------------------------------------------- | ----------------------------------- |
| 1   | Docker architecture, images, containers, Dockerfile               | Create Playwright Dockerfile        |
| 2   | Volumes, networks, Docker Compose, environment variables          | Run tests in Docker                 |
| 3   | Performance fundamentals, load/stress/spike/soak                  | Identify performance test scenarios |
| 4   | k6, virtual users, throughput, latency, P95/P99                   | Write basic k6 script               |
| 5   | Prometheus, Grafana, performance bottlenecks                      | Analyze performance report          |
| 6   | AI Evals: hallucination, groundedness, relevance, golden datasets | Build 10 AI evaluation cases        |
| 7   | AI evaluation, prompt injection, LLM-as-a-judge                   | Evaluate chatbot responses          |

**Interview questions**

- Image vs container?
- Docker vs virtual machine?
- What is Docker Compose?
- Load vs stress testing?
- Throughput vs latency?
- What is P99?
- What are hallucinations?
- How do you evaluate an LLM response?
- What is LLM-as-a-judge?

**Weekly deliverable:** Dockerized automation execution, k6 performance report, and a small AI evaluation dataset.

### **Week 20 – Crio advanced topics and final project**

We will map this week to the actual advanced Crio material. The following is the integration target.

| Day | 40-minute QA / theory / practical               | 20-minute coding / revision      |
| --- | ----------------------------------------------- | -------------------------------- |
| 1   | Advanced automation architecture review         | Refactor one framework component |
| 2   | Advanced API/UI integration and debugging       | Solve an integration failure     |
| 3   | Framework scalability and distributed execution | Optimize a test suite            |
| 4   | Advanced DSA and coding interview               | Solve one medium-level problem   |
| 5   | End-to-end project review                       | Improve project documentation    |
| 6   | SDET system design and project explanation      | Prepare architecture walkthrough |
| 7   | Full technical mock interview                   | Record gaps and revision plan    |

**Weekly deliverable:** A portfolio-ready automation project with documentation and a clear architecture explanation.

# **3. Continuous revision plan**

One problem with long preparation plans is forgetting topics that were studied earlier.

We will solve this using spaced revision.

| Revision interval     | What to revise                                   |
| --------------------- | ------------------------------------------------ |
| Same day              | Explain the topic without notes                  |
| Next day              | Recall important concepts and solve one question |
| After 7 days          | Revisit interview questions                      |
| After 30 days         | Reimplement one practical example                |
| Before mock interview | Revise architecture, edge cases and trade-offs   |

### **Daily 20-minute coding rotation**

The 20-minute coding session should not be limited to the current week's topic.

Use this rotation:

| Day       | Coding/revision activity       |
| --------- | ------------------------------ |
| Monday    | JavaScript/TypeScript coding   |
| Tuesday   | DSA problem                    |
| Wednesday | Playwright UI coding           |
| Thursday  | Playwright API coding          |
| Friday    | SQL / Git / shell practice     |
| Saturday  | Design pattern / system design |
| Sunday    | Weekly revision and assessment |

If a topic requires a longer practical exercise, we can use the full 20 minutes for it.

# **4. Weekly assessment and evaluation system**

We will treat each week like a small engineering milestone.

**Weekly assessment – 100 marks**

| Assessment area          | Marks   |
| ------------------------ | ------- |
| Theory understanding     | 15      |
| Interview explanation    | 20      |
| Practical implementation | 30      |
| Coding / problem solving | 20      |
| Debugging and edge cases | 15      |
| **Total**                | **100** |

**Passing criteria**

Minimum target

# **75/100**

A score of 75 is a learning checkpoint, not a guarantee of interview readiness.

### **Weekly assessment process**

Every seventh day, we will follow this process:

1. **Theory:** Answer 5–10 questions without referring to notes.
2. **Interview:** Explain one topic as if speaking to an interviewer.
3. **Practical:** Implement one task independently.
4. **Debugging:** Fix one intentionally introduced issue.
5. **Review:** Identify weak areas and plan revision.

### **Mastery gates**

We will not move forward just because seven days have passed.

| Skill            | Ready to move forward when...                                            |
| ---------------- | ------------------------------------------------------------------------ |
| Manual QA        | You can independently derive test cases from requirements                |
| JavaScript       | You can write and explain asynchronous code                              |
| TypeScript       | You can design reusable typed utilities                                  |
| Playwright UI    | You can automate complex UI workflows reliably                           |
| Framework design | You can explain fixtures, POM, configuration and reporting               |
| Test data        | You can create isolated data for parallel tests                          |
| Playwright API   | You can automate dependent API workflows                                 |
| DSA              | You can solve easy problems independently and approach medium problems   |
| SQL              | You can write joins, aggregations and subqueries                         |
| CI/CD            | You can create and debug an automation pipeline                          |
| System design    | You can explain architecture and design trade-offs                       |
| Advanced topics  | You can explain concepts and complete representative practical exercises |

If you score below 75, we will identify the weak areas and add revision sessions.

# **5. Project milestones**

The most important outcome of this program should be a portfolio of engineering work.

We will build one main project continuously and add advanced capabilities to it.

## **Main project: Enterprise E-commerce Test Automation Platform**

[Full-Stack E-Commerce Platform - Full Stack Web Development with JavaScript (MERN) & AI | Ostad](https://images.openai.com/static-rsc-4/eYTK6ZrInI43Lo76zuORARh2lTDa2ihRQ8F3jcaaumZe-x822Hs7d8X7BJnsRw-XwrQYMfr2UBqAcTtvbF9L76nQPoV4Ika1DQvZeGyM6NnVUn_ntgf44R3jWEWtNYJpfmxWuQ4d5JDJmliLQYP7hpoGcFdqqa_FtmN8AqqAv7c?purpose=inline)

[ostad.app](https://ostad.app/course/mern/projects/3?utm_source=chatgpt.com)

**Milestone 1 – Week 8**

UI framework

- Playwright + TypeScript.
- Page Object Model.
- Fixtures and reusable components.
- Reporting and screenshots.

**Milestone 2 – Week 10**

Framework engineering

- QA/Staging environments.
- Dynamic test data.
- Parallel execution.
- Sharding and test isolation.

**Milestone 3 – Week 12**

API framework

- API clients and services.
- Authentication.
- Schema validation.
- API/UI integration.
- Database verification.

**Milestone 4 – Week 15**

CI/CD integration

- GitHub Actions.
- Smoke and regression pipelines.
- Distributed execution.
- Test artifacts and reports.

**Milestone 5 – Week 19–20**

Engineering maturity

- Docker execution.
- Performance testing.
- AI-assisted test development.
- Documentation and architecture diagrams.

### **What should be present in our GitHub repository?**

```
enterprise-test-platform/
│
├── tests/
│   ├── ui/
│   ├── api/
│   └── integration/
│
├── pages/
├── components/
├── fixtures/
├── clients/
├── services/
├── test-data/
├── schemas/
├── utils/
├── config/
├── database/
│
├── docker/
├── .github/workflows/
├── docs/
│   ├── test-strategy.md
│   ├── architecture.md
│   ├── test-data-strategy.md
│   └── execution-strategy.md
│
├── playwright.config.ts
├── package.json
└── README.md
```

# **6. Interview preparation strategy**

We should prepare for the different rounds typically encountered in product-based SDET interviews.

| Interview round                | Preparation areas                                             |
| ------------------------------ | ------------------------------------------------------------- |
| Round 1 – Programming          | JavaScript, TypeScript, DSA, coding                           |
| Round 2 – Testing fundamentals | Manual QA, scenarios, test design, debugging                  |
| Round 3 – Automation           | Playwright UI, API, framework design                          |
| Round 4 – Engineering          | SOLID, design patterns, Git, CI/CD, SQL                       |
| Round 5 – System design        | LLD, HLD, automation architecture                             |
| Round 6 – Managerial           | Project explanation, ownership, conflict handling, trade-offs |

**We should also prepare 5 project stories:**

1. How did you design your automation framework?
2. How did you reduce automation execution time?
3. How did you solve flaky test problems?
4. How did you handle test data and environment dependencies?
5. How did you improve automation reliability and maintainability?

Each story should follow:

**Problem → Challenges → Approach → Implementation → Result → Learning**

Avoid claiming improvements or numerical results that you cannot demonstrate.

# **7. Daily preparation tracker**

We will track not just whether a topic was completed, but whether we can explain and implement it.

## **Daily SDET study log**

Current weekWeek 5

**Today's completion**

# **5/5**

Theory understood

Interview questions answered

Practical implemented

20-minute coding completed

Revision completed

Learning notes / blockers

&#x20;Copy daily study log

# **8. Day 1 – Let's start on 20 November 2026**

We should start with Manual QA because strong testing fundamentals will influence how we design our automation framework.

**Day 1 – Software Testing Fundamentals**

First session

20 November 2026 · 11:00 AM – 12:00 PM

**11:00–11:10 – Theory**

- What is software testing?
- Why do we need testing?
- QA vs QC.
- Verification vs validation.
- Error vs defect vs failure.

**11:10–11:25 – Practical**

Consider an e-commerce application:

- User adds a product to cart.
- Cart shows the wrong price.
- Checkout calculates a different total.
- Payment succeeds but the order is not created.

Identify the possible defects and their business impact.

**11:25–11:40 – Interview preparation**

Answer these questions aloud:

1. What is software testing?
2. Why is testing necessary?
3. QA vs QC?
4. Verification vs validation?
5. Difference between error, defect and failure?

**11:40–12:00 – Coding / revision**

- Write 5 real-world examples of software defects.
- Classify each defect.
- Explain its potential impact.

  **Day 1 expected output**

A short note explaining testing fundamentals and five real-world defect examples.

## **9. How we will work together during this preparation**

For each daily session, we will use a consistent format:

1. **Concept explanation:** Simple language, fundamentals first.
2. **Interview perspective:** How to explain it to an interviewer.
3. **Code examples:** JavaScript/TypeScript and Playwright wherever applicable.
4. **Real-world scenario:** How the concept is used in a product company.
5. **Practical assignment:** Something to implement independently.
6. **Interview questions:** Questions with expected answers.
7. **Assessment:** Review understanding and identify gaps.

When you come back for the next session, we can continue from the exact day in this roadmap. You can simply say `SDET Day 2`, `SDET Week 1 Assessment`, or share your implementation for review.

**One final recommendation:** Do not measure preparation by the number of topics completed. Measure it by how confidently you can explain a concept, write the code independently, debug it, and justify your engineering decisions.

Our target is not to finish 20 weeks. Our target is to become interview-ready.**Preparation principle: Learn → Implement → Explain → Debug → Revise.**

This will be our standard approach throughout the SDET journey.
