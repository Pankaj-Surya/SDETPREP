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
