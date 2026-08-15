# API Testing Interview Prep Notes
For: Senior SDET / Staff SDET / Principal SDET / Test Architect
Languages: RestAssured (Java), Playwright (TypeScript)
Covers: FAANG, FinTech, Healthcare, SaaS, Startup expectations

---

## 1. HOW SENIORITY CHANGES WHAT YOU'RE JUDGED ON

- Senior SDET: expected to write robust, maintainable tests, own a test suite/module, debug flaky tests, understand HTTP deeply, contribute to framework code, mentor juniors.
- Staff SDET: expected to design test strategy for a whole service or set of services, make build-vs-buy tool decisions, drive quality metrics, influence engineering culture, handle cross-team test ownership (contract testing, shared frameworks).
- Principal SDET: expected to set quality strategy org-wide, define testing standards, evaluate risk at the architecture level, be the escalation point for "how do we test this system", influence roadmap and headcount.
- Test Architect: expected to design the overall test pyramid/strategy across many services, choose tooling company-wide, define non-functional testing strategy (perf, security, resilience), and be able to defend decisions to senior engineering leadership.

Interviewers scale questions up the ladder like this:
- Senior: "How would you test this endpoint?"
- Staff: "How would you test this microservice and its contracts with 5 dependent services?"
- Principal/Architect: "How would you design a test strategy for 200 microservices with 10 teams, some in different tech stacks, with regulatory audit requirements?"

Always answer at the level you're interviewing for. If you only give Senior-level answers in a Staff interview, you will look underqualified even if technically correct.

---

## 2. HTTP / REST FUNDAMENTALS (rapid fire, must be automatic)

- HTTP methods: GET (idempotent, safe, cacheable), POST (not idempotent, creates), PUT (idempotent, replaces), PATCH (not guaranteed idempotent, partial update), DELETE (idempotent), HEAD, OPTIONS.
- Idempotency: calling the same request N times has the same effect as calling it once. GET/PUT/DELETE should be idempotent. POST usually is not, unless you add an idempotency key.
- Status codes:
  - 2xx success: 200 OK, 201 Created, 202 Accepted (async), 204 No Content
  - 3xx redirect: 301 permanent, 302 temporary, 304 not modified (caching)
  - 4xx client error: 400 bad request, 401 unauthenticated, 403 unauthorized/forbidden, 404 not found, 405 method not allowed, 409 conflict, 422 unprocessable entity, 429 too many requests
  - 5xx server error: 500 internal error, 502 bad gateway, 503 service unavailable, 504 gateway timeout
- 401 vs 403: 401 means "I don't know who you are" (missing/invalid auth). 403 means "I know who you are but you can't do this" (authorization failure).
- Headers to know: Content-Type, Accept, Authorization, Cache-Control, ETag, If-None-Match, X-Request-Id, X-Correlation-Id, Retry-After.
- REST constraints: statelessness, uniform interface, client-server separation, cacheability, layered system, resource-based URLs.
- REST vs RPC vs GraphQL vs gRPC — be ready to compare testing approach for each:
  - REST: test per resource/endpoint, status codes, schema.
  - GraphQL: single endpoint, test query/mutation shape, over/under fetching, N+1 problems, error format is different (200 OK with errors array).
  - gRPC: binary protocol (protobuf), need grpc client tools, test service contracts via .proto files, use tools like ghz or grpcurl.
  - Webhooks/async APIs: test callback delivery, retries, signature verification, out-of-order delivery.

---

## 3. WHAT "GOOD API TEST COVERAGE" MEANS (core interview theme)

Interviewers love asking "what would you test for this endpoint" — structure your answer using these buckets:

1. Functional / happy path — valid request returns correct response, correct status code, correct schema.
2. Negative / validation — missing required fields, wrong data types, invalid enum values, boundary values (empty string, null, max length, negative numbers), malformed JSON.
3. Authentication — no token, expired token, invalid token, wrong signature, token for wrong audience.
4. Authorization — valid user without permission, cross-tenant access (user A accessing user B's data — very important in FinTech/Healthcare), role-based access control (RBAC), scope-based access (OAuth scopes).
5. Contract / schema — response matches OpenAPI/Swagger spec, backward compatibility, no breaking changes for consumers.
6. Idempotency — repeat the same POST/PUT and confirm behavior is safe/expected, especially with idempotency keys.
7. Concurrency / race conditions — two requests modifying the same resource at once (e.g., double withdrawal in a bank API).
8. Data integrity — data actually persisted correctly, side effects verified (e.g., check DB or downstream event was published, not just the response).
9. Performance/load — latency SLAs, throughput, behavior under load, timeout handling.
10. Rate limiting / throttling — 429 responses, Retry-After header, quota reset behavior.
11. Pagination — correct page size, cursor vs offset, edge cases (last page, empty results, out-of-range page).
12. Error handling — consistent error response shape (error code, message, correlation id), no leaking of internal stack traces/PII.
13. Security — SQL/NoSQL injection, XSS in stored fields returned by API, IDOR (insecure direct object reference), mass assignment, sensitive data exposure.
14. Backward compatibility / versioning — old clients still work when new fields are added, deprecated fields handled properly.
15. Localization/internationalization — unicode handling, date/time formats, currency formats (important in FinTech).
16. Resilience — behavior when a downstream dependency is slow/down (circuit breakers, fallback, retries, timeouts).
17. Observability hooks — correlation IDs propagate correctly through logs/traces (important at Staff+ for debugging distributed systems).

For Staff/Principal, always add: "I'd also validate this against the contract our consumers depend on, and think about blast radius if this endpoint breaks."

---

## 4. AUTHENTICATION / AUTHORIZATION TESTING (very common, especially FinTech/Healthcare)

Know these deeply, they get asked constantly:

- Basic Auth vs API Key vs OAuth2 vs JWT vs SAML vs mTLS.
- OAuth2 flows: Authorization Code (with PKCE for public clients), Client Credentials (service-to-service), Refresh Token flow. Know what to test for each — token expiry, refresh token rotation, revocation.
- JWT structure: header.payload.signature. Test: expired exp claim, tampered payload (signature should fail), wrong issuer/audience, algorithm confusion attack (alg: none).
- Token storage/leakage — tokens should not appear in logs, URLs (GET query params), or error messages.
- Session handling for APIs backing web apps — cookie flags (HttpOnly, Secure, SameSite).
- Multi-tenancy testing — this is a favorite in FinTech/Healthcare/SaaS interviews: "How do you test that tenant A cannot see tenant B's data?" Answer: automated test suite that creates two tenants, attempts cross-tenant reads/writes via ID manipulation (IDOR), asserts 403/404, run this as a mandatory gate in CI, not just manual audit.
- HIPAA relevance (Healthcare): PHI (Protected Health Info) must never appear in logs, error messages must not leak PHI, access must be audit-logged, encryption in transit (TLS) and at rest.
- PCI-DSS relevance (FinTech): card data (PAN) must be masked/tokenized, no full card numbers in logs or responses, strict scope isolation.

---

## 5. CONTRACT TESTING (Staff+ favorite topic)

- Why: in microservices, integration tests across all services are slow and brittle. Contract testing lets each team verify their service meets the contract without spinning up the whole system.
- Consumer-Driven Contracts (CDC) — consumer defines expectations, provider verifies against them. Tool: Pact.
- Pact workflow: consumer writes a test defining expected interaction -> generates a pact file (JSON) -> pact file published to a Pact Broker -> provider runs verification tests against the pact file in their CI -> "can-i-deploy" check gates deployment.
- Schema-based contract testing: OpenAPI/Swagger spec validation — validate live responses against the spec (tools: Dredd, Schemathesis, restassured-json-schema-validator, or custom).
- When to use contract testing vs full integration/E2E: contract testing scales better with many services; E2E should be minimal ("test pyramid" — most tests unit, fewer integration/contract, very few E2E).
- How to explain in interview: "I'd use consumer-driven contracts so each team can deploy independently without waiting on a full integration environment, and gate deploys with a can-i-deploy check in the pipeline."

---

## 6. TEST PYRAMID / STRATEGY (Staff/Principal/Architect favorite)

Standard pyramid, bottom to top:
1. Unit tests — fast, isolated, majority of tests.
2. Component/service tests — test a service in isolation with mocked dependencies (WireMock, MockServer, Nock for TS).
3. Contract tests — verify interactions between services without full integration.
4. Integration tests — real service + real (or test) dependencies, fewer of these.
5. E2E tests — full system, very few, only for critical user journeys.
6. Exploratory/manual — small slice, for UX/edge cases automation can't catch easily.

Also mention the "Testing Trophy" concept (Kent C. Dodds) — more weight on integration tests for systems where integration bugs are more common than unit-level bugs — good to mention you know multiple models and pick based on context, not dogma.

For Architect-level: also discuss non-functional test strategy layer sitting alongside the pyramid — performance, security, chaos/resilience, accessibility, and how these are gated in the pipeline (some block release, some are informational).

---

## 7. MOCKING / SERVICE VIRTUALIZATION

- Why mock: isolate the system under test, test failure scenarios you can't reproduce with real dependencies (timeouts, 500s, malformed responses), speed up test execution, remove flakiness from unstable dependencies.
- Tools: WireMock (Java), MockServer, Nock (Node/TS), MSW (Mock Service Worker, great for Playwright/frontend), Prism (mocks from OpenAPI spec), Postman Mock Servers.
- Playwright specific: use route interception (page.route() or context.route()) to mock API responses for E2E tests, or APIRequestContext for pure API testing.
- Know the tradeoff: mocks drift from real behavior over time ("mock rot") — mitigate with contract tests validating that mocks still match reality.

---

## 8. TEST DATA MANAGEMENT (comes up a lot at Staff+, especially FinTech/Healthcare)

- Strategies: fixture data, factory pattern (build test data programmatically), synthetic data generation, data anonymization/masking of production data for test envs (critical in Healthcare/FinTech due to compliance), self-cleaning tests (setup/teardown via API, not DB manipulation, to test through the real interface).
- Test isolation — each test should create its own data, not depend on shared state, to avoid flaky/order-dependent tests.
- Idempotent test setup — tests should be safely re-runnable.
- Ephemeral environments — spin up isolated environment per PR/test run (common at Staff+/Architect discussions: "how do you avoid environment contention across teams").

---

## 9. FLAKY TESTS (guaranteed interview question at every level)

Causes:
- Timing/race conditions (not waiting for async operations properly)
- Shared/mutable test data
- Environment instability (network, third-party dependency)
- Test order dependency
- Improper waits (hardcoded sleep instead of polling/waiting for condition)
- Resource leaks (leftover data breaking subsequent runs)

How to fix (say this in interview):
- Replace hardcoded sleeps with explicit waits/polling on a condition.
- Retry only at the CI level with quarantine tagging, not by silently re-running until green.
- Track flakiness metrics — flag tests that fail intermittently, quarantine them, assign owners to fix, don't let them rot in the suite.
- Ensure test data isolation (unique IDs per test run, e.g., UUID-based emails).
- Use idempotency keys and clean up state in teardown or use ephemeral environments.

Principal/Architect-level answer: "I'd build flaky test detection into CI — track pass/fail history per test, auto-quarantine tests that fail intermittently above a threshold, and make flakiness a visible team metric, not something individual engineers silently work around."

---

## 10. PERFORMANCE / LOAD TESTING OF APIs

- Types: load testing (expected traffic), stress testing (beyond expected, find breaking point), soak testing (sustained load over long time, find memory leaks), spike testing (sudden traffic surge).
- Metrics: latency (p50/p90/p95/p99 — always mention percentiles not just average), throughput (RPS), error rate, resource utilization (CPU/memory/DB connections).
- Tools: k6, Gatling, JMeter, Locust, Artillery.
- How SDETs typically own this: define SLAs/SLOs with the team, build performance tests into CI (smoke-level on every build, full load test on a schedule or before major releases), fail the build if p95 latency regresses beyond a threshold.
- Common trap question: "What's the difference between load testing and performance testing?" — performance testing is the umbrella term including load/stress/soak/spike; load testing specifically tests expected concurrent traffic.

---

## 11. SECURITY TESTING FOR APIs (know OWASP API Top 10 conceptually)

- BOLA / IDOR (Broken Object Level Authorization) — user can access another user's object by changing an ID in the URL/body. #1 most common API vuln — always mention this first.
- Broken authentication.
- Broken object property level authorization (mass assignment — user sends extra fields like "isAdmin": true and server accepts them).
- Unrestricted resource consumption (no rate limiting -> DoS).
- Broken function level authorization (regular user can call admin endpoints).
- Server-side request forgery (SSRF).
- Security misconfiguration (verbose errors, default credentials, unnecessary HTTP methods enabled).
- Injection (SQL, NoSQL, command injection).
- Improper inventory management (old/shadow API versions still live and unprotected).
- Unsafe consumption of third-party APIs.

SDETs should at least run basic security checks in CI: dependency scanning (Snyk/OWASP Dependency-Check), basic fuzzing on inputs, automated IDOR checks, header checks (no sensitive data in headers, HSTS present, CORS not overly permissive).

---

## 12. ASYNC / EVENTUAL CONSISTENCY TESTING

- Common in event-driven/microservices architectures (Kafka, SQS, SNS, Pub/Sub).
- Problem: API call returns 202 Accepted but actual processing happens async — how do you assert success?
- Approaches: poll an endpoint until state changes (with timeout), listen to the event/queue directly in the test and assert the event payload, use test hooks/webhooks the system exposes for testing, avoid arbitrary sleep() calls.
- Also test: message ordering guarantees, at-least-once vs exactly-once delivery, dead-letter queue handling, retry/backoff behavior, duplicate message handling (idempotent consumers).

---

## 13. RESTASSURED (JAVA) — CORE PATTERNS

### 13.1 Basic GET with assertions
```java
given()
    .baseUri("https://api.example.com")
    .header("Authorization", "Bearer " + token)
.when()
    .get("/users/{id}", 123)
.then()
    .statusCode(200)
    .contentType(ContentType.JSON)
    .body("id", equalTo(123))
    .body("email", notNullValue())
    .time(lessThan(2000L));
```

### 13.2 POST with request body (POJO or map)
```java
UserRequest payload = new UserRequest("john@example.com", "John Doe");

Response response = given()
    .contentType(ContentType.JSON)
    .header("Authorization", "Bearer " + token)
    .body(payload)
.when()
    .post("/users")
.then()
    .statusCode(201)
    .extract().response();

String userId = response.jsonPath().getString("id");
```

### 13.3 Schema validation (contract-style check)
```java
given()
    .header("Authorization", "Bearer " + token)
.when()
    .get("/users/123")
.then()
    .statusCode(200)
    .body(matchesJsonSchemaInClasspath("schemas/user-schema.json"));
```

### 13.4 Reusable RequestSpecification (avoid duplication across a large suite — Staff-level design point)
```java
RequestSpecification baseSpec = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .setContentType(ContentType.JSON)
    .addFilter(new RequestLoggingFilter())
    .addFilter(new ResponseLoggingFilter())
    .build();

given()
    .spec(baseSpec)
    .header("Authorization", "Bearer " + token)
.when()
    .get("/orders")
.then()
    .statusCode(200);
```

### 13.5 Data-driven testing with TestNG/JUnit5 parameterized tests
```java
@ParameterizedTest
@CsvSource({
    "'', 400",
    "'not-an-email', 400",
    "'valid@example.com', 201"
})
void createUser_validatesEmail(String email, int expectedStatus) {
    given()
        .contentType(ContentType.JSON)
        .body(Map.of("email", email))
    .when()
        .post("/users")
    .then()
        .statusCode(expectedStatus);
}
```

### 13.6 Chained requests (dependent test flow — extract and reuse)
```java
// create order, then verify via GET, then cancel it
String orderId = given().spec(baseSpec).body(orderPayload)
    .post("/orders")
    .then().statusCode(201).extract().path("id");

given().spec(baseSpec)
    .get("/orders/" + orderId)
    .then().statusCode(200).body("status", equalTo("CREATED"));

given().spec(baseSpec)
    .delete("/orders/" + orderId)
    .then().statusCode(204);
```

### 13.7 Testing idempotency keys
```java
String idempotencyKey = UUID.randomUUID().toString();

Response first = given().spec(baseSpec)
    .header("Idempotency-Key", idempotencyKey)
    .body(paymentPayload)
    .post("/payments");

Response second = given().spec(baseSpec)
    .header("Idempotency-Key", idempotencyKey)
    .body(paymentPayload)
    .post("/payments");

assertThat(first.jsonPath().getString("id"))
    .isEqualTo(second.jsonPath().getString("id")); // same payment, not duplicated
```

### 13.8 Points to raise about RestAssured design at Staff+ level
- Wrap RestAssured calls behind a fluent internal API/DSL (e.g., `UsersApi.createUser(payload)`) so tests read like business language, not raw HTTP calls — improves maintainability at scale.
- Centralize auth/token management (token caching, refresh) in a base client rather than per-test.
- Use filters for consistent logging/correlation ID injection across all requests.
- Integrate schema validation into a shared assertion utility so every response is contract-checked without repeating boilerplate.
- Parallelize test execution (JUnit5 parallel execution or TestNG parallel="methods") and be careful about test data isolation when doing so.

---

## 14. PLAYWRIGHT (TYPESCRIPT) — CORE PATTERNS

Playwright has a built-in `APIRequestContext` for pure API testing (not just browser automation), which many teams now prefer over supertest/axios-based frameworks because it shares tooling with UI/E2E tests.

### 14.1 Basic setup with request fixture
```typescript
import { test, expect } from '@playwright/test';

test('get user returns correct data', async ({ request }) => {
  const response = await request.get('/users/123', {
    headers: { Authorization: `Bearer ${token}` },
  });

  expect(response.status()).toBe(200);
  const body = await response.json();
  expect(body.id).toBe(123);
  expect(body.email).not.toBeNull();
});
```

### 14.2 Custom APIRequestContext with base URL and default headers (reusable across suite)
```typescript
import { request } from '@playwright/test';

const apiContext = await request.newContext({
  baseURL: 'https://api.example.com',
  extraHTTPHeaders: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${token}`,
  },
});

const response = await apiContext.post('/orders', {
  data: { itemId: 'abc123', quantity: 2 },
});
expect(response.status()).toBe(201);
```

### 14.3 Schema validation (using a library like ajv alongside Playwright)
```typescript
import Ajv from 'ajv';
import userSchema from '../schemas/user-schema.json';

const ajv = new Ajv();
const validate = ajv.compile(userSchema);

test('user response matches schema', async ({ request }) => {
  const response = await request.get('/users/123');
  const body = await response.json();
  const valid = validate(body);
  expect(valid, JSON.stringify(validate.errors)).toBe(true);
});
```

### 14.4 Data-driven / parameterized tests
```typescript
const cases = [
  { email: '', expected: 400 },
  { email: 'not-an-email', expected: 400 },
  { email: 'valid@example.com', expected: 201 },
];

for (const { email, expected } of cases) {
  test(`create user with email="${email}" expects ${expected}`, async ({ request }) => {
    const response = await request.post('/users', { data: { email } });
    expect(response.status()).toBe(expected);
  });
}
```

### 14.5 Chained/dependent flow
```typescript
test('create, fetch, and cancel order', async ({ request }) => {
  const createRes = await request.post('/orders', { data: orderPayload });
  expect(createRes.status()).toBe(201);
  const { id } = await createRes.json();

  const getRes = await request.get(`/orders/${id}`);
  expect((await getRes.json()).status).toBe('CREATED');

  const cancelRes = await request.delete(`/orders/${id}`);
  expect(cancelRes.status()).toBe(204);
});
```

### 14.6 Mocking with route interception (useful for E2E tests where API is a dependency, not the SUT)
```typescript
test('shows error banner when API returns 500', async ({ page }) => {
  await page.route('**/api/orders', (route) =>
    route.fulfill({ status: 500, body: JSON.stringify({ error: 'Internal error' }) })
  );

  await page.goto('/orders');
  await expect(page.getByText('Something went wrong')).toBeVisible();
});
```

### 14.7 Combining API setup with UI tests (common Playwright pattern — speeds up E2E)
```typescript
test('user sees their order in the UI', async ({ page, request }) => {
  // seed data via API instead of clicking through the UI — faster, less flaky
  const res = await request.post('/orders', { data: orderPayload });
  const { id } = await res.json();

  await page.goto(`/orders/${id}`);
  await expect(page.getByTestId('order-status')).toHaveText('CREATED');
});
```

### 14.8 Points to raise about Playwright API design at Staff+ level
- Use API calls to set up/tear down state for E2E tests instead of driving the UI for setup — big speed and stability win, always mention this.
- Wrap `APIRequestContext` in a Page Object–style API client class per resource (UsersClient, OrdersClient) for reuse and readability.
- Use Playwright's built-in parallelization and sharding for CI speed, and isolate test data per worker (e.g., unique test user per worker index).
- Playwright can trace and generate HTML reports with request/response details automatically — useful for debugging flaky API failures in CI without re-running locally.

---

## 15. FRAMEWORK / ARCHITECTURE DESIGN QUESTIONS (Staff/Principal/Architect)

Typical prompt: "Design an API test automation framework from scratch for our company."

Structure your answer like this:

1. Requirements gathering — what's the tech stack, how many services, sync vs async APIs, compliance needs (HIPAA/PCI/SOC2), team size, CI/CD tooling already in place.
2. Layered framework design:
   - Core HTTP client layer (RestAssured spec or Playwright APIRequestContext wrapper)
   - Domain/API client layer (UsersApi, OrdersApi — business-readable methods)
   - Test data layer (builders/factories, cleanup utilities)
   - Assertion/utility layer (schema validators, custom matchers)
   - Reporting layer (Allure/ExtentReports/Playwright HTML report)
   - Config layer (environment-specific config, secrets management — never hardcode credentials)
3. CI/CD integration — run smoke tests on every PR, full regression nightly, contract tests gate deploys, performance tests on schedule, parallel execution to keep pipeline fast.
4. Environment strategy — ephemeral/preview environments vs shared staging, test data seeding strategy, service virtualization for unavailable dependencies.
5. Metrics/observability — track flaky test rate, test execution time trend, defect escape rate, coverage by risk area (not just % coverage — risk-based).
6. Governance — coding standards for tests, code review process for test code (yes, test code should be reviewed like production code), ownership model (who owns which test suites).
7. Scaling considerations — how this framework serves 10 teams without becoming a bottleneck, self-service tooling so other teams can write tests without SDET as a gatekeeper.

Principal/Architect add: talk about "shift-left" (testing earlier, e.g., contract tests before dev finishes, schema validation in the API design/review stage) and "shift-right" (testing in production — canary analysis, synthetic monitoring, feature flags with gradual rollout, chaos engineering).

---

## 16. CHAOS / RESILIENCE TESTING (Architect-level, increasingly asked at FAANG/FinTech)

- Concept: intentionally inject failure (kill a service, add latency, drop network) to verify the system degrades gracefully instead of cascading failure.
- Tools: Chaos Monkey/Chaos Toolkit, Gremlin, AWS Fault Injection Simulator, Toxiproxy (great for simulating latency/timeouts in API tests specifically).
- SDET's role: write automated resilience test suites — e.g., inject 5-second latency into a downstream dependency and assert the caller times out gracefully and returns a proper fallback/error rather than hanging or crashing.
- Circuit breaker testing — assert the circuit opens after N failures, requests fail fast while open, and it half-opens/recovers correctly.

---

## 17. COMPANY-TYPE SPECIFIC FOCUS (what interviewers in each vertical care about most)

### FAANG / Big Tech
- Scale: how do you test APIs serving millions of requests/sec.
- Distributed systems correctness (eventual consistency, idempotency, retries, race conditions).
- Deep framework/tooling design questions, internal tooling build vs buy tradeoffs.
- Heavy emphasis on automation-first mindset, CI/CD velocity, canary/production testing.
- Expect system design style questions even for SDET roles ("design a test strategy for our checkout API used by 3 different clients").

### FinTech
- Idempotency and exactly-once semantics are critical (money must never be double-charged/double-transferred).
- Strong compliance focus: PCI-DSS, SOX audit trails, immutable transaction logs.
- Precision with numbers — testing decimal/currency handling, rounding errors, floating point issues (should use BigDecimal in Java, not double, for money).
- Fraud/security testing emphasis — IDOR, rate limiting on sensitive endpoints (login, money transfer).
- Reconciliation testing — verifying data consistency between services (ledger matches transaction log).

### Healthcare
- HIPAA compliance — PHI handling, audit logging, access control testing.
- Data integrity is paramount — incorrect data could affect patient care, so strong emphasis on validation testing.
- Interoperability standards — HL7, FHIR APIs are common; know that FHIR is REST-based with specific resource schemas, and testing often includes conformance to FHIR spec.
- Slower release cycles due to regulation but still need strong automated regression to support that.

### SaaS
- Multi-tenancy testing is central — data isolation between customers.
- API versioning and backward compatibility (many external customers depend on your API, breaking changes are very costly).
- Rate limiting/quota testing per subscription tier.
- Public API documentation accuracy — contract tests against published OpenAPI spec matter a lot since external developers build on it.

### Startup
- Pragmatism over process — expect questions about how you'd bootstrap a test framework with limited resources/headcount.
- Risk-based prioritization — you can't test everything, how do you decide what matters most with a tiny team.
- Wearing multiple hats — SDET might also do some DevOps/CI pipeline work.
- Expect less depth on formal process, more on judgment and speed of delivering value.

---

## 18. STAFF / PRINCIPAL LEADERSHIP & BEHAVIORAL QUESTIONS (API-testing flavored)

- "Tell me about a time you found a critical bug that would have caused a major outage/financial loss." — structure with STAR (Situation, Task, Action, Result), emphasize the systemic fix (not just the bug, but how you prevented the class of bug going forward — e.g., added contract test gate).
- "Tell me about a time you influenced a team to adopt a testing practice they resisted." — emphasize how you built buy-in with data (e.g., showed defect escape rate before/after) rather than mandate.
- "How do you decide what NOT to test?" — answer with risk-based thinking: business impact x likelihood of failure, and mention explicit tradeoffs (e.g., skip exhaustive UI-level testing of a low-traffic internal tool, focus on payment flow).
- "How do you handle disagreement with a developer who thinks a bug is 'not a bug'?" — show data-driven, calm escalation, ownership without ego.
- "Describe how you scaled testing practices across multiple teams." — for Staff+/Architect, have a real or well-thought-out example: shared framework, self-service tooling, office hours/documentation, champions model.

---

## 19. RAPID-FIRE LIKELY QUESTIONS WITH SHORT ANSWERS

Q: What's the difference between API testing and UI testing?
A: API testing hits the service layer directly — faster, more stable, tests business logic without UI rendering concerns. UI testing validates the full user experience including rendering, but is slower and more fragile. Good strategy uses mostly API tests with a thin layer of critical-path UI tests.

Q: How do you test an endpoint that depends on a third-party API you don't control?
A: Mock/stub the third party for most tests (WireMock/Nock), have a small number of tests against a sandbox/staging environment of the third party if available, and monitor production for real-world contract drift.

Q: How would you test pagination?
A: Verify first page, last page, empty result set, page size boundaries, requesting a page beyond available data, consistent ordering across pages (no duplicates/missing items if data changes mid-pagination), and cursor-based vs offset-based edge cases.

Q: How do you test rate limiting?
A: Send requests exceeding the limit and confirm 429 is returned with correct Retry-After header, confirm limit resets after the window, confirm limit is enforced per correct scope (per user/per API key/per IP as designed).

Q: What is a false positive vs false negative in testing, and why do they matter?
A: False positive = test fails but there's no real bug (usually flaky/environment issue) — erodes trust in the suite. False negative = test passes but there is a real bug — dangerous because it gives false confidence. Both are addressed by good test design, proper assertions, and not over-mocking.

Q: How do you approach testing a breaking API change?
A: Version the API (URL versioning or header versioning), run contract tests against both old and new consumers, communicate deprecation timeline, keep old version running in parallel until consumers migrate, monitor usage of deprecated version before sunsetting.

Q: RestAssured vs Playwright for API testing — when would you pick one over the other?
A: RestAssured is Java-native, deeply integrated with JVM ecosystems (Spring Boot teams, Maven/Gradle pipelines), mature schema validation support. Playwright's APIRequestContext is a good pick when the team is already using Playwright for E2E/UI (shared tooling, one framework for both, TypeScript ecosystem, good parallelization/tracing/reporting out of the box). Neither is "better" universally — pick based on team's existing stack and skill set.

Q: How do you test webhooks?
A: Stand up a test receiver endpoint (or use a tool like ngrok/RequestBin in test env), trigger the event, assert payload shape/signature verification, test retry behavior on receiver failure, test duplicate delivery handling (idempotent processing).

Q: What would you do if a test suite takes too long and slows down CI?
A: Identify slow tests (usually E2E/UI or serial DB-heavy tests), parallelize execution, move redundant coverage down the pyramid (turn an E2E test into an API/unit test if it's testing logic not UI), use ephemeral/lightweight environments, and add fast smoke suite for PRs with fuller regression run async/nightly.

---

## 20. MOCK INTERVIEW EXERCISE PROMPTS (practice these out loud)

1. "Walk me through how you'd test a POST /transfer endpoint for a banking app that moves money between two accounts." (expect: idempotency, negative balance, concurrent transfer race condition, rounding, authorization on both accounts, audit logging, rollback on partial failure)
2. "We have a GraphQL API with a single endpoint — how does your test approach change vs REST?" (expect: query/mutation coverage, depth/complexity limits, error array format, N+1 query performance)
3. "Design a test strategy for a healthcare API exposing patient records via FHIR." (expect: HIPAA, FHIR schema conformance, access control per role, audit trail testing)
4. "Our checkout API has 5 downstream dependencies (inventory, payment, tax, shipping, notifications) — how do you test it without a fragile full integration suite every time?" (expect: contract testing per dependency, mocking for negative/failure scenarios, small number of true E2E smoke tests)
5. "You inherited a flaky test suite with a 30% failure rate not related to real bugs — what's your 90-day plan?" (expect: triage/quarantine, root cause categorization, fix top offenders, add flakiness tracking to CI, prevent regressions with code review standards for new tests)
6. "How would you convince leadership to invest in a dedicated test architecture team?" (expect: cost of quality argument, defect escape data, incident cost tied to test gaps, phased rollout plan)

---

## 21. QUICK CHECKLIST TO REVIEW NIGHT BEFORE THE INTERVIEW

- Can explain idempotency and give a money/payment example without hesitation.
- Can list OWASP API security top issues, especially BOLA/IDOR, from memory.
- Can write a RestAssured POST + schema validation test on a whiteboard/live coding from memory.
- Can write a Playwright APIRequestContext test with a data-driven loop from memory.
- Can explain contract testing (Pact) end to end in under 2 minutes.
- Can explain test pyramid vs testing trophy and when to deviate.
- Has 2-3 real STAR stories ready: a critical bug found, a framework/process you built or improved, a conflict you navigated.
- Can explain how they'd design a framework from scratch in a structured way (requirements -> layers -> CI integration -> metrics -> governance).
- Comfortable discussing tradeoffs out loud — interviewers want to hear reasoning, not just a memorized "correct" answer.

---

END OF NOTES
