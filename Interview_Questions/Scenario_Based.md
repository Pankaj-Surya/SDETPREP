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
