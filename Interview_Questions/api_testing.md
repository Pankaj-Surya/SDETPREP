## Q1. What happens if you PUT/PATCH a resource that doesn't exist?

**PATCH — straightforward**
- PATCH is a partial update of an *existing* resource, so if the resource doesn't exist, it must fail
- Correct expected status: `404 Not Found`
- Example: `PATCH /orders/9999` where order 9999 doesn't exist → should return 404, not silently create or succeed

**PUT — this is the nuance to know for an interview**
- Per REST/HTTP spec, PUT is defined as "replace the resource at this URI" — and the spec technically allows PUT to **create** the resource if it doesn't exist (this is called an "upsert")
- So the correct behavior actually depends on how the API is **designed and documented** — both are valid:
  - Strict design: PUT on non-existent ID → `404 Not Found` (resource must exist first, created only via POST)
  - Upsert design: PUT on non-existent ID → `201 Created` (PUT creates it since you're explicitly saying "this is what the resource at this URI should look like")
- Example: `PUT /users/123` where user 123 doesn't exist — if the API spec says "PUT creates if absent," a 404 here would actually be a **bug**; if the spec says "PUT only updates," then a 201 here would be the bug

**How to test this properly**
- First check the API contract/spec to know which behavior is intended
- Then write the test to assert *that specific* documented behavior — don't assume 4xx is always correct without checking the contract
- If the spec is silent/ambiguous, flag it as a spec gap to the team rather than guessing

## Q2. POST /orders called twice with the same payload — both return 201, two orders created. Is the API correct?

**Key concept to get right**
- Per HTTP spec, POST is explicitly **not required to be idempotent** — calling it twice creating two resources is technically "correct" HTTP behavior, since POST means "create a new thing," and two calls are two separate creation requests
- So from a pure HTTP-semantics view, the API isn't violating any rule

**But from a real-world/business correctness view, this is still a problem**
- If the two calls represent the *same user action* (e.g., a network retry, or user double-clicking "Place Order"), creating two separate orders is a business logic bug — duplicate orders, duplicate charges, unhappy customer
- The fix isn't "make POST idempotent" (that goes against HTTP semantics) — the standard solution is an **Idempotency-Key** pattern:
  - Client generates a unique key (e.g., UUID) per logical action and sends it in a header (`Idempotency-Key: abc-123`)
  - Server checks: if it's seen this key before, return the *original* response instead of creating a new record
  - Example: Stripe's payment API works exactly this way — retrying a payment call with the same idempotency key returns the original charge, not a duplicate

**How I'd test this as an SDET**
- Call POST twice with the same payload but no idempotency key → confirm two resources ARE created (expected HTTP behavior)
- Call POST twice with the same payload and same idempotency key → confirm only ONE resource is created and both responses reference the same order ID (business requirement)
- If the API doesn't support idempotency keys at all and duplicate submission is a known real risk (like payments/orders), flag it to the dev team as a design gap — this is where a good SDET adds value beyond just checking status codes

## Q3. How do you test that an auth token expires without waiting an hour?

**Your backdated-token approach is good — here's the full toolkit to mention**

- Ask backend to expose a test-only endpoint/config to issue tokens with a custom, shorter expiry (e.g., 30 seconds) for automation — most reliable and CI-friendly
- Example: instead of waiting 60 minutes, request a token with `exp` set to "issued 59 minutes 55 seconds ago," then wait 5 seconds and confirm the API returns `401 Unauthorized`
- If you can't get backend support, decode the JWT yourself (JWTs are just base64-encoded, not encrypted) and manually craft one with a modified `iat`/`exp` claim, signed with the test environment's secret key (only works if you legitimately have access to the test signing secret — never do this against real/prod secrets)
- Alternative: mock/manipulate system clock in the test environment so the app "thinks" more time has passed (libraries like `sinon` in JS, `Clock` mocking in Java) — useful when time logic is evaluated server-side using system time
- Alternative: reduce token TTL specifically in the test/staging environment config (e.g., 60 seconds instead of 1 hour) so real expiry can be tested end-to-end without any mocking

**What to actually assert once you have an expired token**
- Call a protected endpoint with the expired token → expect `401 Unauthorized`
- Confirm the error response body clearly indicates "token expired" (not a generic 401) so the client app can react correctly (e.g., trigger re-login)

## Q4. Login endpoint returns 200 with an error object inside the body — pass, fail, or skip?

- This should be marked as a **fail** in the test suite
- Reasoning: HTTP status codes exist specifically to communicate success/failure at the protocol level — returning `200 OK` while the body contains an error object breaks the API contract and misleads any client/consumer that checks status code first (which is standard practice)
- Correct expected design would be one of two things:
  - `4xx` status code (e.g., 401 for invalid credentials) with an error object in the body, OR
  - `200 OK` with no error object / error field is `null`, meaning login genuinely succeeded
- Example: `POST /login` with wrong password returning `200 OK` + `{ "error": "Invalid credentials" }` — any client/dashboard/monitoring tool that just checks `status === 200` would treat this as a successful login, which is a real bug with real consequences (broken client error-handling, false monitoring green status)
- As a tester, I wouldn't just fail the test silently — I'd flag it as a contract violation to the dev team with this exact reasoning, since it's a design issue, not just something to code around in my test

## Q5. Test creates a user, asserts, deletes in teardown — 8 parallel CI workers hit shared staging. What breaks?

**Root cause**
- This is a race condition caused by shared, non-isolated test data across parallel workers
- Example: if all 8 workers use the same hardcoded email (`test@example.com`) to create a user, the first worker's create call succeeds; the remaining 7 either fail on a uniqueness constraint (duplicate email) or one worker's teardown deletes the user while another worker is still mid-test using it

**What specifically breaks**
- Create step: only 1 of 8 create calls succeeds if using static/shared data (unique constraint violation for the other 7)
- Mid-test: one worker's teardown (delete) can remove a user that another worker is still actively asserting against, causing unrelated tests to fail with "user not found"
- Shared staging environment itself may have other automated/manual activity happening, adding more unpredictability

**Fix**
- Give each worker/test its own unique, dynamically generated test data — e.g., append a UUID, timestamp, or the CI worker index to the email/username at creation time (`test_${Date.now()}_${workerId}@example.com`)
- Each test's setup/teardown should only touch the data *it created* — never touch shared or global test data
- If truly independent data isn't possible (e.g., a shared reference table), consider test data namespacing per worker, or database transactions/rollback scoped per test where the environment supports it
- Longer-term: consider isolated environments per test run (e.g., ephemeral test DB or containerized backend spun up per CI run) instead of one shared staging environment for all parallel workers — this eliminates the race condition at the root

## Q6. Spec says responses are sorted by created_at, dev says "it usually is." How do you test this as a contract, not a coincidence?

- The key mindset: don't just eyeball that today's response looks sorted — write an assertion that would **fail** if the sort behavior ever silently breaks in the future
- Create multiple records (e.g., 10) with clearly distinguishable `created_at` timestamps — add a small deliberate delay between creations if the system generates timestamps too fast to differentiate (avoids flaky false-passes from identical timestamps)
- Call the actual **API endpoint** being tested (not the DB directly) to fetch the list — since the goal is verifying the API's contract/behavior, not just what's stored in the DB
- Extract the `created_at` field from each item in the response array in the order returned
- Assert programmatically that the array is sorted in the expected order (ascending or descending, per spec) — e.g., loop through and confirm each `created_at` is `>=` the previous one, rather than manually comparing values
- Run this as a repeatable, permanent regression test (not a one-time manual check) — so if a future code change accidentally breaks the sort order, this test catches it immediately instead of relying on "it usually is" from the dev
- Bonus: also explicitly test the *documented* sort field and order (e.g., confirm it's `created_at descending`, not `id ascending` that just happens to look similar) — this catches subtle contract mismatches that a superficial "looks sorted" check would miss

## Q14. What API validations have you performed in your project?

- Status code validation — confirm the right HTTP code returns for each scenario (200, 201, 400, 401, 404, 500)
- Response body validation — confirm correct fields, correct values, correct data types
- Response header validation — confirm headers like `Content-Type`, `Cache-Control`, custom headers are present and correct
- Schema validation — confirm the response structure matches the agreed contract (all expected fields present, correct types, no unexpected extra/missing fields)
- Response time validation — confirm the API responds within an acceptable time limit
- Negative/error validation — confirm invalid inputs return proper error codes and meaningful error messages, not a generic 500
- Data consistency validation — confirm data returned by the API actually matches what's stored in the DB
- Security validation — confirm unauthorized/unauthenticated calls are correctly rejected (401/403)

**Real-time example**
- For a banking `POST /transfer` API, I validate: status code is 201 on success, response body has correct `transactionId` and `status: SUCCESS`, headers include correct `Content-Type: application/json`, invalid account number returns 400 with a clear error message, and finally I cross-check in the DB that the transaction actually got recorded with the correct amount.

## Q15. How do you validate API response status codes?

- Check the returned status code matches what's expected for that specific scenario, not just "did it return something"
- Different scenarios expect different codes — success (200/201/204), client error (400/401/403/404/409), server error (500)

**Real-time example**
- `POST /orders` with valid payload → expect `201 Created`
- `GET /orders/9999` where order doesn't exist → expect `404 Not Found`
- `POST /orders` with missing required field like `productId` → expect `400 Bad Request`
- `GET /orders` without an auth token → expect `401 Unauthorized`
- In RestAssured: `.then().statusCode(201);`
- In Playwright: `expect(response.status()).toBe(201);`

## Q16. How do you validate API response body and response headers?

**Response body validation**
- Check specific field values, not just that a response came back
- Check data types match the contract (e.g., `price` should be a number, not a string)
- Check nested objects/arrays have correct structure
- Ignore fields that are expected to change every time (like timestamps, auto-generated IDs) unless specifically testing those

**Real-time example**
- For `GET /users/101`, I'd assert: `response.body.id == 101`, `response.body.email` is a valid email format, `response.body.status == "ACTIVE"`
- RestAssured: `.body("id", equalTo(101))`
- Playwright: `expect(json.id).toBe(101);`

**Response header validation**
- Check `Content-Type` matches expected format (`application/json`)
- Check security-related headers where relevant (`Cache-Control`, `X-RateLimit-Remaining`, custom auth headers)
- Check `Location` header on `201 Created` responses (should point to the newly created resource)

**Real-time example**
- After `POST /orders` returns 201, I'd check the `Location` header equals `/orders/{newOrderId}` — confirms the API correctly tells the client where to find the new resource

## Q17. Difference between Basic Authentication, Bearer Token, and OAuth

| Type | How it works | Real-time example |
|---|---|---|
| Basic Auth | Username and password combined, base64-encoded, sent in the `Authorization` header on every request | Older internal admin tools, some legacy enterprise APIs — simple but insecure over plain HTTP since the credentials are just encoded, not encrypted |
| Bearer Token | A token (often a JWT) is sent as `Authorization: Bearer <token>` after initial login; server trusts whoever holds the token | Most modern REST APIs — e.g., after logging into a banking app, every subsequent API call carries the token instead of re-sending username/password |
| OAuth 2.0 | A full authorization framework where a user grants a third-party app limited access without sharing their password; involves an authorization server issuing tokens | "Sign in with Google" on a shopping app — the shopping app never sees your Google password, it just gets a token with limited permission (e.g., access to your email only) |

**Simple way to remember it**
- Basic Auth = sending your actual password every time (weak)
- Bearer Token = "here's my ticket, let me in" (token proves who you are after one login)
- OAuth = "let this app act on my behalf, but only for these specific things" (delegated access, no password sharing)

## Q18. How do you handle authentication in API automation testing?

*(Already covered in detail earlier — quick recap in plain English)*
- Authenticate once at suite start, store the token, reuse it across dependent tests
- Refresh the token automatically if it's close to expiring mid-suite, instead of letting tests fail with 401
- Never hardcode credentials — pull from environment variables or a secrets manager
- For OAuth-based APIs, use the Client Credentials flow for automation since there's no real user logging in through a UI

## Q19. RestAssured code for a PATCH request

```java
given()
    .baseUri("https://api.example.com")
    .header("Authorization", "Bearer " + accessToken)
    .contentType(ContentType.JSON)
    .body("{ \"status\": \"SHIPPED\" }")
.when()
    .patch("/orders/{orderId}", orderId)
.then()
    .statusCode(200)
    .body("status", equalTo("SHIPPED"));
```

**Plain-English explanation**
- Set the base URL and auth token
- Set the request body to JSON with only the field you want to update (that's the whole point of PATCH — partial update, not sending the entire object like PUT would)
- Call `.patch()` with the endpoint and path variable
- Assert the status code and confirm the specific field actually got updated in the response

## Q20. How do you detect schema drift across 3 downstream consumers?

**What schema drift means in plain English**
- The API's response structure quietly changes over time (a field renamed, removed, or its type changed) and one or more consuming systems don't get updated to match — they start breaking or silently misreading data

**How to detect it**
- Maintain a single source-of-truth schema (OpenAPI/JSON Schema) for the API, versioned in a shared repo
- Run automated schema validation tests on every API response against this schema in CI, on every build — not just once
- For each of the 3 downstream consumers, maintain a **consumer contract** describing exactly what fields/structure *that consumer* expects (this is the idea behind consumer-driven contract testing, e.g., using Pact)
- Run all 3 consumer contracts against the API in CI before every deploy — if the API change breaks any one consumer's expected contract, that specific test fails and names exactly which consumer would be affected

**Real-time example**
- A banking API `GET /account/balance` is consumed by: (1) the mobile app, (2) an internal reporting dashboard, (3) a partner fintech integration
- If backend renames `availableBalance` to `balanceAvailable`, schema validation catches the structural change immediately, and the specific contract test for the partner fintech integration fails, telling you exactly which consumer will break — instead of finding out only after the partner calls to complain

## Q21. An optional field becomes required — production breaks — but your contract test passed. Why? Also: how do you validate a nested response with dynamic keys?

**Why the contract test missed it**
- This usually happens because the contract test only validates the fields it explicitly knows about — if the schema definition itself was updated to mark the field "required" but the actual test data used in the contract test always happened to include that field anyway, the test never exercised the "field is missing" case
- Another common cause: the contract test validates the **response schema** but not the **request schema** — if the field became required on the request side (something the client must now send) and the test always sent it anyway (because that's the "happy path" test data), the break only shows up in production when a real caller doesn't send that field
- Real example: `POST /orders` used to treat `couponCode` as optional; a backend change quietly makes it required. Contract test's stored request payload already included a `couponCode` field, so the test kept passing — but the mobile app's older version never sends `couponCode` for users without a coupon, so real production calls start failing with 400 errors that nobody caught in testing

**Fix**
- Contract tests need explicit negative cases too — deliberately omit optional fields and confirm the API still behaves as expected, not just test with a "complete" payload every time
- Whenever a field's requiredness changes in the schema, add a specific test for the "field missing" scenario, don't just rely on existing happy-path data continuing to work

## Q22. **How to validate a nested response when keys are dynamic**

*Plain English: dynamic keys means the key name itself changes each time (like a date, an ID, or a category name), so you can't hardcode `response.body.someFixedKeyName` in your assertion — you have to write logic to check contents regardless of what the key is called.*

**Real-time example — banking domain**
- A banking transaction summary API returns balances grouped by account number as the key:
```json
{
  "accountsSummary": {
    "AC1023456": { "balance": 5000, "currency": "INR" },
    "AC1023987": { "balance": 12000, "currency": "INR" }
  }
}
```
- Since account numbers differ per user/test run, you can't assert `response.accountsSummary.AC1023456.balance` directly
- Instead: iterate over the keys dynamically — `for (String key : response.accountsSummary.keySet())` — and assert generic rules for every entry: `balance` is a number and `>= 0`, `currency` is a valid 3-letter code, etc.

**Real-time example — e-commerce domain**
- A product catalog API returns pricing grouped dynamically by region:
```json
{
  "pricing": {
    "IN": { "amount": 999, "currency": "INR" },
    "US": { "amount": 15, "currency": "USD" }
  }
}
```
- Region codes vary depending on which regions are configured for that product, so instead of hardcoding `pricing.IN`, loop through all keys present and validate each one follows the same structural rule (amount is positive number, currency is valid ISO code)

## Q23. How do you seed test data without going through the UI?

- Seed data directly via API calls (call the backend's own create endpoints, e.g., `POST /users`) instead of clicking through UI forms — much faster and more reliable
- Seed data directly into the database using SQL scripts or a test data setup library, when even the API is too slow/complex for bulk data needs
- Use fake/dummy data generation instead of manually typing values — this is where Faker comes in

**Faker — plain English explanation**
- Faker is a library (available in Java as `JavaFaker`, in JS as `@faker-js/faker`) that generates realistic-looking random data — names, emails, addresses, phone numbers, dates — so you don't hardcode the same test data everywhere (which causes collisions in parallel runs, as covered in Q5 earlier)

**Real-time example**
- Instead of hardcoding `email: "test@example.com"` (which fails on the 2nd parallel run due to a uniqueness constraint), use:
```java
String email = Faker.instance().internet().emailAddress();
String name = Faker.instance().name().fullName();
```
- Then call `POST /users` with this generated data to seed a fresh, unique user for each test run — no UI interaction, no data collisions across parallel workers

## Q24. Two tests share an auth token. One mutates user state. How do you fix the flake?

**The problem in plain English**
- Both tests log in as the same user and share the same token — Test A changes something about that user (e.g., updates their email or locks their account), and now Test B, which assumed the user was in its original state, fails unpredictably depending on execution order

**Real-time example**
- Test A: "update profile name" test changes the shared test user's name to "Updated Name"
- Test B: "verify default profile name is 'John Doe'" test runs after Test A in parallel and fails, because the name is no longer "John Doe" — not because of a real bug, but because they were fighting over the same user

**Fix**
- Give each test its own dedicated user (created fresh via API/Faker as in Q22), so no two tests ever share mutable state
- Each test authenticates with its own token tied to its own user — not a single shared token across the whole suite
- If creating a user per test is too expensive/slow, at minimum ensure any test that **mutates** state uses its own isolated user, while read-only tests can safely share one
- Reset/teardown the user's state after each test that mutates it, so it doesn't leak into other tests even if reused

## Q25. How do you test an endpoint that depends on 4 other API calls executing in order?

**Two approaches, both correct — use depending on the goal**

**Approach 1 — Chaining (real integration test)**
- Call all 4 dependent APIs in the actual required sequence, pass the output of each as input to the next, and finally call the target endpoint
- Use this when you want to test the *real* end-to-end flow as it happens in production

**Real-time example — e-commerce**
- To test `POST /orders/{id}/confirm`, you first need: `POST /cart` (add item) → `POST /address` (set shipping address) → `POST /payment` (process payment) → then finally `POST /orders/{id}/confirm`
- Chain these calls, capturing `cartId`, `addressId`, `paymentId` from each response and passing them into the next call, ending with the confirm call under test

**Approach 2 — Mocking the dependencies**
- Instead of really calling all 4 APIs, mock/stub their responses so the target endpoint can be tested in isolation, faster and without depending on other services being up
- Use this when you specifically want to unit-test just the target endpoint's logic, or when the dependent services are slow/unstable/not always available in the test environment

**Real-time example**
- If testing `POST /orders/{id}/confirm` in isolation, mock the payment service response as `{ "paymentStatus": "SUCCESS" }` directly, so your test doesn't depend on the real payment gateway being available or slow

**Which to prefer**
- Use chaining for critical E2E business flows (checkout, fund transfer) where the real integration matters
- Use mocking for faster, more isolated tests of edge cases at the specific endpoint level (e.g., what happens if confirm is called but payment status is "FAILED" — easier to simulate via mock than to force a real payment failure)

## Q26. Your endpoint accepts JSON. You send XML. What's the right failure?

- Correct expected behavior: `400 Bad Request` (or `415 Unsupported Media Type`, which is technically more precise for this specific case) with a clear error message
- `415 Unsupported Media Type` is the more textbook-correct status code here — it specifically means "the server understands the content type you sent, but refuses to process it because it doesn't support that format for this endpoint"

**Real-time example**
- `POST /orders` expects `Content-Type: application/json` — if you send XML with `Content-Type: application/xml`, the API should reject it with `415 Unsupported Media Type` and a message like `"Unsupported content type. Expected application/json"`
- What should NOT happen: a `500 Internal Server Error` — that would mean the server crashed trying to parse XML as JSON instead of gracefully rejecting it, which is a robustness bug worth reporting

## Q27. The API returns 200 with an error in the body — bug or pass?

*(Already covered in detail in Q4 of the earlier set — same answer applies: this should be marked as a fail, since a `200 OK` status while returning an error object violates the API contract and misleads clients that check status codes first.)*

## Q28. Rate limiting kicks in at 100 req/sec — how do you test the 101st request's behavior?

**Correction on tooling**
- You're right that pure load simulation (sending massive concurrent traffic) is better suited to performance tools like k6, JMeter, or Gatling — but rate limiting specifically (checking that the 101st request in a short window gets rejected) **can absolutely be tested with Playwright/RestAssured too**, since you just need to fire 101 requests quickly, not simulate thousands of virtual users — this is a functional correctness check, not a load/performance test

**Two valid approaches**
- Functional check (RestAssured/Playwright): fire 101 requests as fast as possible within the rate limit window (e.g., using a loop with parallel/async calls), and assert the first 100 return success while the 101st returns `429 Too Many Requests` with a clear error message — this validates the rate-limiting *logic* is correctly implemented
- Load/performance check (k6/JMeter): simulate realistic concurrent user traffic at scale to validate rate limiting holds up under real production-like load, and that the system degrades gracefully rather than crashing — this validates *system behavior under load*, a different concern than just logic correctness

**Real-time example**
- Banking API allows 100 balance-check requests per second per API key. Functional test: fire 101 requests rapidly using a thread pool/async calls, confirm requests 1–100 return `200 OK`, and request 101 returns `429 Too Many Requests` with a `Retry-After` header telling the client when to try again
- Also worth checking: does the rate limit counter reset correctly after the time window passes — e.g., wait 1 second after hitting the limit, then confirm request 102 succeeds again

## Q29. What is the difference between encoded and encrypted?

**Encoded — plain English**
- Encoding just changes data into a different format so systems can transfer/store it properly — it's **not for security**, it's for compatibility
- Anyone can decode it instantly, no secret key needed — it's completely reversible by design
- Example (banking): a JWT auth token's payload is base64-**encoded** — anyone can copy the token, paste it into a site like jwt.io, and instantly read the raw claims inside (user ID, role, expiry) — no password needed to view it, just to view it as intended data
- Example (e-commerce): a product image URL with special characters gets URL-encoded (`%20` for space) so the browser can process it correctly

**Encrypted — plain English**
- Encryption scrambles data specifically so **only someone with the correct key** can read it — this is for security/confidentiality
- Without the decryption key, the data is unreadable gibberish

**Real-time examples**
- Banking: a customer's account number and CVV stored in the database are **encrypted** — even if someone steals the database file, they can't read the actual numbers without the decryption key
- E-commerce: when you enter your card details at checkout, they're **encrypted** (via HTTPS/TLS) while traveling from your browser to the server, so nobody intercepting the network traffic can read the raw card number

**Simple way to remember for the interview**
- Encoded = "changed format for convenience, anyone can reverse it" (like writing in shorthand)
- Encrypted = "locked with a key, only the key holder can reverse it" (like a locked safe)

## Q30. An endpoint returns user data. How do you verify it doesn't leak other users' data?

**This is testing for a real, serious vulnerability called IDOR (Insecure Direct Object Reference)**

Your instinct is right, here's the full test approach:

- Log in as User A, note their user ID
- Log in as User B, get User B's valid token
- Using **User B's token**, try to call the endpoint requesting **User A's ID** in the path — e.g., `GET /users/A123/profile` with User B's auth token
- Expected correct behavior: API should reject this with `403 Forbidden` (or `404 Not Found` to avoid revealing that the ID even exists) — never return User A's data to User B

**Real-time example — banking**
- `GET /accounts/{accountId}/statement` — log in as Customer B, then try requesting Customer A's account ID in the URL using Customer B's token
- If the API returns Customer A's statement (balance, transactions) to Customer B, that's a critical security bug — this is exactly the kind of bug that makes news headlines in banking apps

**Real-time example — e-commerce**
- `GET /orders/{orderId}` — try fetching another customer's order ID while logged in as yourself
- If it returns their order details (items, shipping address, payment last 4 digits), that's a data leak

**What to also check**
- Test with no auth token at all (should be 401)
- Test with an expired/invalid token (should be 401)
- Test with a valid token but wrong role/permission level (should be 403)
- This category of test should be a **standard, mandatory** part of every API test suite that returns user-specific data — not an afterthought

## Q31. Payment gateway is down. How do you continue your test suite?

**Your instinct (mocking + strategy pattern) is good — here's the practical breakdown**

- Use a **mock/stub of the payment gateway** in the test environment that returns realistic responses (success, failure, timeout) without depending on the real third-party gateway being up
- Design the code so switching between the real gateway and the mock is a simple config change — this is where the strategy/adapter pattern helps, since your application code calls a common interface, and swaps the actual implementation (real vs mock) based on environment

**Real-time example — e-commerce**
- Payment gateway (like Razorpay/Stripe) is down. Your automated checkout tests use a mocked payment service that returns `{ "paymentStatus": "SUCCESS", "transactionId": "TXN123" }` instantly, so the rest of the checkout flow (cart → address → confirm order) can still be tested end-to-end without waiting on the real gateway
- You can also simulate failure scenarios on demand — mock returns `{ "paymentStatus": "FAILED" }` to test how your app handles a declined payment, which is actually harder to trigger reliably with a real gateway anyway

**Real-time example — banking**
- Testing a fund transfer flow where the core banking payment processor is a third-party dependency — use a mocked processor in the test/staging environment so QA isn't blocked whenever that external system has downtime or maintenance windows

**Important caveat to mention**
- Mocked tests validate your app's logic, but you still need periodic real integration tests against the actual payment gateway (in a sandbox/test mode most gateways provide) to catch real-world drift — mocking alone isn't a complete substitute (ties into Q33 below)

## Q32. How do you mock a streaming response?

**Plain English — what a streaming response is**
- Instead of getting the whole response at once, data arrives in small chunks over time, while the connection stays open (e.g., live stock price updates, live order tracking status, chat messages)

**How to mock it**
- Instead of returning one fixed JSON blob immediately, the mock server needs to send data in **chunks**, with a delay between each chunk, simulating how the real streaming server would behave
- Tools: WireMock supports chunked/delayed responses; for WebSocket-based streaming, tools like `mock-socket` (JS) can simulate a WebSocket server sending messages over time; Node.js `http` server can manually write chunks using `res.write()` multiple times before `res.end()`

**Real-time example — banking**
- A live "transaction status" feature (e.g., "processing" → "verifying" → "completed") sent via Server-Sent Events (SSE) or WebSocket while a fund transfer is happening — to test the UI updates correctly at each stage, mock the stream to send `"processing"`, wait 1 second, send `"verifying"`, wait 1 second, send `"completed"` — and assert the UI updates correctly after each chunk arrives, not just at the final state

**Real-time example — e-commerce**
- A live order-tracking map showing the delivery agent's location updating every few seconds — mock the streaming endpoint to send a sequence of fake coordinates over time, and verify the UI marker moves correctly with each update

## Q33. Mock returns success. Real API returns failure. How do you catch this drift?

**The problem in plain English**
- Your automated tests all pass because they're running against a mock that was set up once and never updated — meanwhile the real API's actual behavior has changed (maybe it now fails under a condition the mock doesn't know about) — so your tests give false confidence

**How to catch it**
- Never rely on mocks alone — always run a smaller set of tests against the **real API** (in a sandbox/staging environment) periodically, even if the majority of the suite uses mocks for speed
- Use **contract testing** (as covered earlier, e.g., Pact) — this specifically verifies the mock's assumed behavior actually matches what the real provider does, and fails your CI if they drift apart
- Schedule periodic **mock validation checks** — a scheduled job that hits the real API with the same test cases used to build the mock, and alerts if the real response no longer matches what the mock assumes

**Real-time example — e-commerce**
- Your checkout tests mock the payment gateway to always return `SUCCESS`. Meanwhile, the real payment gateway changed its fraud-detection logic and now returns `FAILED` for certain test card numbers you're using. Your mocked tests keep passing, but real checkout in production starts failing for real users. A weekly scheduled test hitting the real sandbox gateway with the same test cases would have caught this drift before it hit production.

## Q34. Your API returns correct data but corrupts the database. How do you catch this in automation?

**The problem in plain English**
- The API response looks perfectly fine to the caller (correct status code, correct-looking data), but something is wrong in what actually got saved to the database — e.g., wrong data type stored, a related table not updated, a calculation stored incorrectly behind the scenes

**How to catch it**
- Never validate only the API response — also add a **direct DB-level assertion** after the API call, checking the actual stored data matches expectations (as covered in Q12 — validating UI/API/DB together)
- Check not just the primary record, but any **related tables** that should also be updated as part of the same operation (referential integrity)
- Check data types and constraints at the DB level, not just at the API's JSON level (e.g., API says `"balance": 500.00` but DB stored it as an integer `500`, losing decimal precision)

**Real-time example — banking**
- `POST /transfer` API returns `200 OK` with `{ "status": "SUCCESS" }`, correctly showing the transfer completed. But at the DB level, the source account's balance got deducted correctly, while the destination account's balance was never actually credited due to a bug in the backend transaction logic. A test that only checks the API response would miss this completely — you need a follow-up DB query confirming **both** accounts reflect the correct updated balances after the transfer.

**Real-time example — e-commerce**
- `POST /orders` returns success and a valid order ID, but the DB's inventory table wasn't decremented for the purchased item — leading to overselling stock later. Catch this by querying the inventory count in DB before and after the order call, and asserting it decreased by the correct quantity.

## Q35. How do you validate a JSON response dynamically?

**Plain English — what "dynamically" means here**
- Instead of hardcoding checks for specific values (which breaks the moment data changes), you write validation logic that checks **structure and rules** that should hold true regardless of the actual values

**Approaches**
- **Schema validation** — validate the response against a JSON Schema, confirming required fields exist, correct data types, correct format (e.g., email fields match email pattern) — without caring about the actual values
- **Rule-based/property assertions** — instead of asserting an exact value, assert a rule: `price >= 0`, `email matches valid email pattern`, `status is one of [ACTIVE, INACTIVE, PENDING]`
- **Dynamic key traversal** — as covered in Q21, loop through keys/arrays when the structure has variable-length lists or dynamic key names, applying the same rule to every item found

**Real-time example — banking**
- For `GET /transactions`, instead of hardcoding "the 5th transaction has amount 500," validate dynamically: every transaction in the returned array has a non-negative `amount`, a valid `date` format, and a `type` that's either `CREDIT` or `DEBIT` — this holds true no matter how many transactions come back or in what order

**Real-time example — e-commerce**
- For `GET /products`, validate dynamically that every product in the list has a `price > 0`, a non-empty `name`, and a valid `category` from an allowed list — rather than hardcoding checks for specific product names that will break the moment the catalog changes

## Q36. How do you chain multiple API requests?

**Plain English**
- Chaining means calling APIs in sequence where the output/data from one call becomes the input for the next call — simulating a real multi-step business flow

**How to do it (RestAssured/Playwright)**
- Call API 1, extract the needed value from its response (like an ID or token)
- Pass that extracted value into API 2's request (path, body, or header)
- Repeat for as many steps as the flow needs
- Use path/query parameters, request bodies, or headers to carry forward the values

**Real-time example — banking**
- Step 1: `POST /accounts` creates a new account, response returns `accountId`
- Step 2: `POST /accounts/{accountId}/deposit` uses that `accountId` to deposit initial funds
- Step 3: `GET /accounts/{accountId}/balance` uses the same `accountId` to verify the balance reflects the deposit correctly
- In RestAssured, you'd store `accountId` from step 1's response using `.extract().path("accountId")`, then reuse that variable in the next calls

**Real-time example — e-commerce**
- Step 1: `POST /cart` add item, get `cartId`
- Step 2: `POST /cart/{cartId}/checkout` using that `cartId`, get `orderId`
- Step 3: `GET /orders/{orderId}` to confirm order was placed correctly with the right items and total

## Q37. What are the main components of an API request?

- **Endpoint/URL** — the address of the resource being accessed (e.g., `https://api.bank.com/v1/accounts/123`)
- **HTTP Method** — the action being performed (GET, POST, PUT, PATCH, DELETE)
- **Headers** — metadata about the request, like `Content-Type`, `Authorization` token, `Accept` format
- **Path parameters** — values embedded directly in the URL path, usually identifying a specific resource (e.g., `123` in `/accounts/123`)
- **Query parameters** — key-value pairs added after `?` in the URL, usually for filtering/searching (e.g., `/orders?status=SHIPPED&limit=10`)
- **Request body/payload** — the actual data being sent, usually in JSON format, used with POST/PUT/PATCH (e.g., new order details)
- **Authentication** — credentials proving who's making the request (Bearer token, API key, Basic Auth)

**Real-time example — e-commerce**
- `POST https://api.shop.com/v1/orders/456/items?notify=true`
  - Endpoint: `/orders/456/items`
  - Method: `POST`
  - Path parameter: `456` (the order ID)
  - Query parameter: `notify=true` (whether to send a notification)
  - Header: `Authorization: Bearer <token>`, `Content-Type: application/json`
  - Body: `{ "productId": "P789", "quantity": 2 }`
