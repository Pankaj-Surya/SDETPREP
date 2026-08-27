## CI/CD, Infrastructure & Cloud

## Q1. CI is failing 30% of the time. Walk me through diagnosing that end to end.

Your instincts are good — here's the structured, step-by-step version an interviewer wants to hear:

- **Check if it's consistent or random** — first pull test history/trends (previous run reports) to see: is it always the *same* tests failing (real bug or broken test), or *different random* tests each time (flakiness/environment issue)?
- **Confirm the right code is actually being tested** — verify correct branch, correct commit/build got deployed to the test environment, and it actually contains the intended changes (a surprisingly common cause of "false" failures)
- **Check environment health** — is the test environment itself stable? Any deployment still in progress, DB not fully migrated, dependent services down?
- **Check CI logs directly** — read the actual failure logs/stack trace in the CI tool, not just "test failed" — this usually immediately tells you if it's a real assertion failure, a timeout, a connection error, or a setup/config issue
- **Try to reproduce locally** — run the same failing test locally, both via automation and manually through the app — if it fails locally too, it's a real bug; if it passes locally but fails only in CI, it points to a CI-environment-specific issue
- **Check for parallel execution issues** — race conditions, shared test data conflicts (as covered in the earlier shared-staging question) — common cause of "flaky 30%" patterns
- **Check credentials/permissions/network** — expired test credentials, network restrictions in the CI runner, firewall/whitelist issues that don't exist locally
- **Check dependency/package versions** — confirm no dependency silently auto-updated to a new/breaking/deprecated version in the CI pipeline (this is a very common real-world cause — lock your dependency versions)
- **Check CI infrastructure itself** — is the CI runner/agent under-resourced (low memory/CPU) causing timeouts under load, especially with parallel jobs?

**Real-time example — banking**
- CI fails intermittently on "verify transaction history" test. Checking logs shows it fails only when running in parallel with the "create transaction" test — two tests hitting the same test account and racing each other. Root cause: shared test data, not a real app bug. Fix: give each test its own dynamically generated account.

**Real-time example — e-commerce**
- CI fails 30% of the time on checkout tests specifically after 6 PM IST. Turns out that's when the shared staging environment gets hit by another team's load testing, slowing responses past the timeout. Root cause: environment contention, not automation or app bug. Fix: either isolate environments or increase timeout with retry logic for known contention windows.

## Q2. We ship/deploy 3 times a day. Where does QA live in that cycle?

**Correction/reframe on your answer**
- With 3 deploys a day, full manual regression before every single deploy is unrealistic — you're right about that — but the answer isn't just "skip manual testing," it's about restructuring **where and how** QA fits in

**Where QA actually lives in a high-frequency deploy cycle**
- **Shift-left** — QA reviews requirements/acceptance criteria *before* code is written, catching ambiguity early, so fewer bugs are introduced in the first place
- **Automated smoke/sanity suite runs automatically** on every build in the CI pipeline — triggered the moment new code is merged, giving fast pass/fail feedback within minutes, without a human needing to manually trigger anything
- **Full regression runs on a schedule** (e.g., nightly) rather than before every single one of the 3 daily deploys — catches deeper issues without blocking deploy speed
- **Manual/exploratory testing happens in parallel**, focused specifically on the *new* feature/change being shipped that day — not re-testing the whole app every time
- **Feature flags** are commonly used alongside frequent deploys — new features are deployed but toggled off for most users until QA/product verifies them in production with a small user group, then gradually rolled out — reduces risk of shipping fast
- **Monitoring/alerting in production** becomes part of the QA safety net too — since you can't manually catch everything before 3 daily releases, production monitoring (error rates, key business metrics) acts as the last line of defense, and QA often helps define what to monitor

**Real-time example — banking**
- A bank shipping 3x/day: automated smoke suite (login, balance check, fund transfer happy path) runs on every deploy automatically and must pass before deploy proceeds; full regression (including edge cases, compliance checks) runs nightly; any new feature (like a new bill-payment option) is behind a feature flag, tested manually by QA with a small internal user group first before being enabled for all customers

**Real-time example — e-commerce**
- An e-commerce site shipping 3x/day: automated checkout smoke test runs on every deploy; QA manually explores just the specific new feature that day (e.g., a new discount code type); full regression suite runs overnight; a new "buy now pay later" option is feature-flagged and tested with 5% of traffic before full rollout

## Q3. What is CI/CD integration in automation?

**Plain English**
- It means your automated test suite is wired directly into the build/deployment pipeline, so tests run **automatically** whenever code is pushed/merged/deployed — instead of someone manually triggering test runs
- CI (Continuous Integration) = automatically build and test code every time it's pushed
- CD (Continuous Deployment/Delivery) = automatically deploy that code to an environment (staging/production) once it passes those tests

**How it works in practice**
- Developer pushes code → CI pipeline automatically triggers → code builds → automated test suite runs (smoke/sanity/regression as configured) → if tests pass, code can automatically deploy to the next environment (staging → UAT → production) → if tests fail, the pipeline stops and the team gets notified immediately

**Real-time example — banking**
- A developer fixes a bug in the fund-transfer service and pushes code → CI pipeline automatically builds it, runs the automated regression suite for transfers and related modules → all tests pass → code auto-deploys to UAT → team gets a Slack notification with the result, no manual test-triggering needed

**Real-time example — e-commerce**
- Developer adds a new discount logic → pushes code → pipeline runs the cart/checkout automated suite automatically → if a test fails (discount not applying correctly), the pipeline blocks deployment and alerts the team immediately, before it ever reaches real customers

## Q4. How do you integrate tests with Jenkins / GitHub Actions?

**Your answer is correct — here's the polished/complete version**

**Jenkins**
- Create a **declarative pipeline** (a `Jenkinsfile` checked into the repo) defining stages: checkout code → build → run tests → publish reports → deploy
- Example stages: `Checkout` → `Build` → `Run Automation Tests` → `Publish Test Report` (e.g., using Allure/JUnit plugin) → `Deploy` (only if tests passed)
- Jenkins can be configured to trigger automatically via a webhook whenever code is pushed to the repo (GitHub/Bitbucket/Azure DevOps)

**GitHub Actions / Azure DevOps**
- Create a YAML workflow file (`.github/workflows/tests.yml` for GitHub Actions) defining triggers (e.g., "on push to main" or "on pull request") and steps (checkout code, install dependencies, run tests, upload report as an artifact)
- Azure DevOps uses a similar YAML pipeline (`azure-pipelines.yml`) with stages/jobs/steps

**Real-time example — banking**
```yaml
# GitHub Actions example - simplified
on: [push]
jobs:
  run-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install
      - run: npx playwright test --grep "fund-transfer"
      - uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: playwright-report/
```
- Every push to the banking app's repo automatically triggers this — runs the fund-transfer automated suite, uploads the report as a downloadable artifact for the team to review

**Real-time example — e-commerce**
- A Jenkins declarative pipeline for the e-commerce checkout module: `Checkout code` → `npm install` → `Run smoke suite` → if pass, `Deploy to staging` → `Run full regression on staging` → if pass, `Notify team on Slack, ready for prod deploy`

## Q5. What is your experience with Docker in test automation?

**What Docker is — plain English**
- Docker packages your application (or your test environment) into a lightweight, portable "container" that includes everything needed to run — code, dependencies, config — so it behaves exactly the same on any machine (your laptop, CI server, teammate's laptop), avoiding the classic "works on my machine" problem

**How it's used in test automation**
- Run browsers in containers for consistent, isolated test execution — e.g., Selenium Grid with Docker containers for Chrome/Firefox nodes, so tests run in a clean, identical browser environment every time, regardless of what's installed on the host machine
- Spin up the **entire test environment on demand** — application + database + dependent services — all as Docker containers, run tests against it, then tear it down; useful for isolated, repeatable testing without a shared, potentially "dirty" staging environment
- Use `docker-compose` to define and start multiple containers together (app + DB + mock services) with one command, which is great for local development and CI setup
- CI pipelines commonly run test automation itself inside a Docker container, so the exact same Node.js/Java/browser versions are used every time in CI, matching what was tested locally

**Real-time example — banking**
- Instead of all QA engineers hitting one shared, sometimes-unstable staging environment (leading to the race-condition/shared-data issues covered in Q5 of the API section), the team uses Docker Compose to spin up an isolated instance of the banking app + test database for each CI run — tests run against a clean, dedicated environment every single time, then it's torn down, so no data conflicts between parallel CI runs

**Real-time example — e-commerce**
- Running Selenium Grid with Dockerized Chrome/Firefox nodes in CI — instead of installing and maintaining specific browser versions on every CI agent, containers spin up fresh browser instances for each test run, ensuring consistent results regardless of which physical CI machine picks up the job
