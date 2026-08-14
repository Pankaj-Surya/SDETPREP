## Q1. Your suite has 600 tests and takes 45 minutes. What do you do?

**Step 1 – Diagnose before fixing**
- First check *why* it's slow: bad/hard waits, flaky tests causing retries, or just genuine test volume
- Check if it's one flat suite — separate smoke/sanity (fast, run every commit) from full regression (can run nightly)

**Step 2 – Parallelize**
- Run tests in parallel instead of sequentially
- Decide parallel granularity: test level, class level, file level, or suite/describe block level
- Parallelization works best when:
  - Tests are independent of each other (no test depends on another test's outcome)
  - Test data is isolated per test to avoid race conditions, read-after-write lags, shared state conflicts

**Step 3 – Tool-specific implementation**
- Selenium: configure `parallel="methods/classes/tests"` and thread-count in **testng.xml**; use ThreadLocal for WebDriver instances so each thread gets its own driver; reporters also need thread-safe handling
- Playwright: set `workers` count in `playwright.config.ts` under the `defineConfig` function

**Step 4 – Scale beyond one machine**
- Shard tests across multiple machines/nodes for extra speed once a single machine maxes out
- Selenium: Selenium Grid distributes tests across multiple nodes in CI
- Playwright: built-in sharding via CLI flags (`--shard=1/3` etc.), merge blob reports after

## Q2. How do you decide what not to automate?

**Core principle**
- Automate based on cost vs. value: high repetition + stable requirement + deterministic outcome = automate

**Automate**
- Tests executed repeatedly every release cycle — smoke, sanity, regression

**Don't automate / handle differently**
- OTP, Captcha, MFA — can't automate the third-party challenge itself, but most teams disable/whitelist these in test environments so the rest of the flow can still be automated
- One-off test cases — rarely executed, not worth automation investment
- Requirements/UI still under active change — automating too early means high maintenance cost
- Cases needing human judgment — usability, visual polish, subjective UX checks
- E2E scripts duplicating coverage already done at unit/API/integration level

**Shared-resource / flaky scenario handling (not a reason to skip automation, it's a design fix)**
- Example: multiple test scripts (Rinaldo, Pankaj, etc.) transferring money through a shared suspense/temp holding account, run in parallel
- If tests share the same suspense account, they can collide when simulating locked/suspended states
- Fix: isolate test data — give each test its own dummy account/suspense instance — rather than skipping automation for that scenario

## Q3. How do you stop test coverage from rotting as the product evolves?

**Coverage rot has two causes — both need to be addressed**
- Gaps — new features shipped without new tests
- Staleness — old tests still passing but testing behavior nobody cares about anymore, or tests that don't meaningfully verify anything

**Tools/process to prevent it**
- Maintain an RTM (Requirement Traceability Matrix) mapping features to test cases
  - Example: schedule transfers (internal/external/FPS/DDA out) supporting multiple types with scheduling options (one-off/daily/weekly/monthly/yearly)
  - Use RTM to spot missing coverage → add new test scripts
  - Use RTM to spot tests for deprecated/no-longer-supported features → remove/retire them
- Tag/link test cases to requirement IDs or user stories so traceability isn't a manual spreadsheet exercise
- Periodic suite audits (each release/quarter) — review suite against current requirements, retire dead tests
- Track and triage flaky tests actively — don't let people just re-run to green, since that hides real rot
- Optional (shows depth): mutation testing to confirm tests still actually catch bugs, not just pass silently

## Q4. What is Headless Browser Testing and what are its use cases?

**Definition**
- A browser that runs without a GUI — same DOM rendering, JS execution, and network calls happen under the hood, just no visible window; controlled via CLI/API

**Use cases / benefits**
- Better performance and faster execution
- Less resource utilization — can run many instances in parallel on limited hardware
- Frees up the machine to do other QA activities while automation runs in the background
- Matches how CI pipelines actually execute — running headless locally helps catch pipeline-only failures early and prevents surprises in the pipeline

**Caveat worth mentioning**
- Headless and headed aren't always 100% identical — occasional rendering/font/screenshot differences
- Debugging is harder since you can't visually watch it fail live
- Best practice: run headed locally for debugging, headless in CI for speed

## Q5. How do you generate automation reports and share execution results with stakeholders?

**By audience**
- QA/Dev team: detailed reports with screenshots/logs on failure — Allure, Extent Reports
- CI Pipeline: JUnit/XML format so results integrate into the pipeline and can fail the build correctly
- Non-technical stakeholders: summarized results, not raw reports

**Delivery channels**
- Email notification with report attached (using an email service package, e.g., Nodemailer in Node.js)
- Slack/Teams notification with a summary posted to a channel/group
- Live dashboard with a shareable URL so the team can self-serve status anytime

**Report formats**
- Allure, Extent — rich HTML reports with history, screenshots, retries
- HTML, JSON, JUnit/XML — general-purpose and CI-friendly formats
- Playwright reporters specifically: `blob` (used to merge sharded run results into one report) and `line` (compact terminal output for quick local runs)

**What impresses interviewers — trend visibility**
- A single run's report only tells you pass/fail for that day
- Track trends over time — pass rate over multiple runs, recurring flaky tests, coverage drift
- Tools like Allure trend graphs or ReportPortal give stakeholders a sense of whether quality is improving, not just a snapshot

## Q6. How do you identify, reduce flaky tests, and stabilize highly flaky tests?

**Definition**
- A flaky test is a non-deterministic test — it passes and fails intermittently against the same code, with no actual functional change

**Identification approach**
- Re-run the suspected test 10–15 times in a loop to observe pattern and pinpoint the exact failure point
- Playwright: use `repeat-each` flag to re-run a test N times
- Selenium/TestNG: use `invocationCount` attribute — `@Test(invocationCount = 10)` — (not `count`)
- Run repeatedly in headless mode too, since flakiness often shows up more in CI/headless than local headed runs due to timing/rendering differences
- Track flaky tests over time in the reporting/dashboard (tag them, don't just re-run silently) — a test that needs re-runs to pass is a signal, not noise

**Common root causes to check**
- Timing/synchronization issues — most common cause
- Test data collisions — shared data between parallel tests
- Environment instability — third-party/API dependency slowness
- Order-dependency — test relies on another test's leftover state

**Reduction/stabilization strategies**
- Replace hard waits (`Thread.sleep`) with proper explicit/dynamic waits — wait for actual condition (element visible, network idle, API response) rather than fixed time
- Playwright's built-in auto-waiting and web-first assertions help reduce this significantly
- Ensure test isolation — each test creates/cleans its own data, doesn't depend on execution order
- Build the end-to-end flow correctly — proper setup/teardown, no assumptions about prior state
- Quarantine chronically flaky tests into a separate suite until fixed, so they don't block the pipeline or erode trust in the whole suite
- Stabilize environment/network dependencies — mock unstable third-party calls where appropriate

## Q7. How do you identify whether a failure is application-related or automation-related?

**Key concepts**
- False positive: test **fails** but the application actually works fine — automation/script issue (bad locator, timing, wrong assertion)
- False negative: test **passes** but the application actually has a bug — automation isn't verifying the right thing

**Approach to isolate root cause**
- Reproduce manually — try the same steps by hand in the UI; if it fails manually too, it's an application bug, not automation
- Check logs — automation logs, browser console logs, network logs, and backend/API/DB logs together
- Check backend/DB state directly — don't just trust the UI's confirmation message
- Review the test's assertions — confirm the test is actually validating backend state, not just a UI toast/message

**Applying it to the order example**
- Scenario: UI shows "order placed" but no order exists in the backend/DB
- This is an **application bug**, not an automation issue — the UI is showing a false success message to the real user, independent of any test script
- However, it also exposes an **automation gap**: if the test only asserted on the UI confirmation message and never verified the order in the DB/API response, that's a weak/incomplete test — it should be strengthened to validate backend state too
- So: primary fault = application (UI/backend mismatch is a real production bug affecting real users); secondary lesson = automation should be enhanced to catch this class of bug in future by asserting at the data layer, not just the UI layer

## Q8. What would be your strategy for testing a critical release under tight timelines?

**A. If it's a change to an existing feature**
- Smoke/sanity of the core feature to confirm nothing critical is broken
- Test flows directly related to the changed feature
- Regression on surrounding/dependent modules that interact with this feature

**B. If it's a new feature**
- Core happy-path flows first
- Basic unhappy/negative flows (invalid input, missing fields)
- Basic field-level and response-level validation for APIs (status codes, required fields, data types)
- If time permits, extend to important edge-case negative scenarios

**Additional points to add for a complete answer**
- Do a quick risk-based prioritization first — identify what's highest-impact/highest-usage before deciding what to test, rather than testing everything shallowly
- Communicate scope clearly to stakeholders — explicitly state what was tested and what was deliberately deprioritized due to time, so risk is visible and agreed upon, not hidden
- Run automated regression in parallel/background while doing manual exploratory testing on the new/changed area, to use time efficiently
- Do a quick rollback/monitoring plan check — confirm feature flags, rollback steps, and post-release monitoring/alerts are ready, since tight-timeline releases carry higher production risk

## Q9. Among all locators, which is the fastest?

**Correct framing**
- Speed differences between locators are usually negligible in practice — the better question to prioritize is **stability/reliability**, not raw speed
- A flaky-but-fast locator is worse than a slightly-slower-but-stable one, since re-runs from flakiness cost far more time than locator lookup speed

**General reliability ranking (most to least stable), applies to both tools**
- Dedicated test-id attributes (`data-testid`, `data-test`) — most stable, immune to UI/style/DOM changes
- ID (`#id`) — stable if unique and not auto-generated/dynamic
- CSS selector — reasonably fast and readable, but breaks if class names/structure change
- Role/text-based locators — stable for user-facing behavior, aligns with accessibility
- XPath (especially absolute XPath) — most fragile, breaks easily with DOM changes, but sometimes only option for complex relative traversal

**Selenium specifics**
- `By.id` and `By.cssSelector` are generally faster than `By.xpath`, since XPath uses the browser's XPath engine which is comparatively slower
- Avoid absolute XPath entirely; relative XPath only when no better option exists

**Playwright specifics**
- Recommended priority: `getByTestId()` > `getByRole()` > `getByText()` > `getByLabel()` > CSS > XPath
- Playwright's locators are auto-waiting and re-query the DOM on each action, so they're inherently more stable against timing issues than Selenium's static element reference — reducing "stale element" flakiness by design

**Answer to say in interview**
- "I'd prioritize stable locators — test IDs first, then role/ID, then CSS — over chasing raw speed, since flaky locators cost more time in re-runs than any speed difference saves."

## Q10. Handling memory leaks and random NullPointerExceptions when running tests in parallel using ThreadLocal

**Why this happens**
- WebDriver instances stored in `ThreadLocal` aren't automatically cleaned up when a thread finishes — if not removed explicitly, the driver reference lingers in memory even after the test completes
- In parallel execution with thread pools (common in TestNG/Selenium Grid), threads get reused across multiple tests — if `ThreadLocal.remove()` isn't called, a new test can accidentally pick up a stale/null reference from a previous test's thread

**Memory leak examples**
- WebDriver instance not quit/closed after test, and ThreadLocal reference not removed — browser process and thread reference both stay in memory across the suite run
- Listeners/event handlers attached to WebDriver never de-registered
- Static collections (lists/maps) used to store test data or driver references across tests, growing indefinitely without cleanup

**Fix — proper ThreadLocal lifecycle management**
- Initialize driver in `@BeforeMethod`/`@BeforeEach`
- Always call `driver.quit()` in `@AfterMethod`/`@AfterEach` — not just `close()`, which only closes the current window
- Explicitly call `ThreadLocalDriver.remove()` after quitting the driver, so the reference doesn't persist in the thread pool for reuse
- Wrap teardown in `try-finally` so cleanup always happens, even if the test fails/throws an exception

**Handling the NullPointerExceptions specifically**
- Root cause is usually: a thread is reused by the pool, and the new test accesses the ThreadLocal driver before it's initialized for that thread, or after it's been removed — resulting in null
- Fix: ensure driver initialization happens strictly in `@BeforeMethod` (not in constructor or `@BeforeClass`, which may run once per class, not once per thread/test)
- Add null-checks with clear custom exception messages during driver retrieval, so failures point directly to "driver not initialized for this thread" instead of a generic NPE — makes debugging in CI much faster
- Use try-catch around driver setup/teardown, log thread ID + test name on failure, so flaky/thread-related failures are traceable in parallel CI logs
