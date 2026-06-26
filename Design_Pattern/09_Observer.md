# Observer Pattern in Test Automation

The **Observer pattern** establishes a one-to-many dependency between objects. When one object (the **Subject**) changes its state, all its dependents (**Observers**) are notified and updated automatically. 

In test automation, this pattern is highly effective for building decoupled **event-driven logging, reporting, real-time metrics tracking, or taking automatic screenshots/videos** upon test failures without polluting your core test scripts or Page Objects with logging logic.

---

### How the Observer Pattern Works
Instead of explicitly calling a reporting tool inside every test step or catch block, components listen for automation execution events.

* **The Subject (Test Runner / Custom Event Broadcaster):** Monitors execution state changes (e.g., `onTestStart`, `onStepPass`, `onTestFailure`) and maintains a list of observers.
* **The Observers (Listeners):** Subscribed modules (like an Allure Reporter, an Slack Notifier, or a Local Screenshot Utility) that execute specific actions when notified.
* **The Test Case:** Runs its normal workflow completely unaware of who is watching or logging the results.

---

### Benefits for Test Automation
* **Strict Separation of Concerns:** Your test scripts focus purely on assertions and business logic, while logging, recording, and reporting logic live entirely in dedicated listener classes.
* **Dynamic Plug-and-Play Architecture:** You can add, remove, or swap reporting tools (e.g., switching from ExtentReports to Allure or adding a Microsoft Teams alert) instantly by attaching or detaching observers.
* **Cleaner Maintenance:** Changes to a reporting API or a notification webhook only require modifications in one specific Observer class rather than across hundreds of tests.

---

### Implementation Code

#### 1. Playwright / TypeScript Implementation
Playwright provides native support for this pattern via custom **Test Reporters**. 

**Define the Observers (Custom Reporters):**
```typescript
import { Reporter, TestCase, TestResult, TestStep } from '@playwright/test/reporter';

// Observer 1: Real-time Console Logger
export class ConsoleLogObserver implements Reporter {
    onStepEnd(test: TestCase, result: TestResult, step: TestStep) {
        if (step.category === 'test.step') {
            console.log(`[STEP FINISHED]: ${step.title} -> Status: ${step.error ? '❌ FAILED' : '✅ PASSED'}`);
        }
    }
}

// Observer 2: Slack Alert Dispatcher (Notifies only on unexpected failures)
export class SlackNotificationObserver implements Reporter {
    async onTestEnd(test: TestCase, result: TestResult) {
        if (result.status === 'failed' || result.status === 'timedOut') {
            console.log(`[SLACK NOTIFIER]: Sending alert! Test "${test.title}" failed in ${result.duration}ms.`);
            // Actual axios/fetch webhook logic goes here
        }
    }
}
```

**Register the Observers inside your Config (`playwright.config.ts`):**
```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
    testDir: './tests',
    // Registering multiple observers to listen to the test runner stream
    reporter: [
        ['list'], // Playwright's default built-in observer
        ['./reporters/ConsoleLogObserver.ts'],
        ['./reporters/SlackNotificationObserver.ts']
    ],
    use: {
        screenshot: 'only-on-failure',
    },
});
```

---

#### 2. Selenium / Java Implementation
In Java automation frameworks, TestNG or JUnit Listeners elegantly implement the Observer design pattern.

**Define the Observers using TestNG Listeners:**
```java
package listeners;

import org.testng.ITestContext;
import org.testng.ITestListener;
import org.testng.ITestResult;

// The Subject (TestNG) notifies this Observer automatically on state changes
public class ReportingObserver implements ITestListener {

    @Override
    public void onTestStart(ITestResult result) {
        System.out.println("[REPORTING]: Starting Test Case: " + result.getName());
    }

    @Override
    public void onTestSuccess(ITestResult result) {
        System.out.println("[REPORTING]: Test Case Passed 🎉: " + result.getName());
    }

    @Override
    public void onTestFailure(ITestResult result) {
        System.err.println("[REPORTING]: Test Case Failed ❌: " + result.getName());
        System.err.println("[LOG EXTRACTION]: Capturing system logs...");
        // Integration with ExtentReports or Allure reporting occurs here
    }

    @Override
    public void onFinish(ITestContext context) {
        System.out.println("[REPORTING]: All tests executed. Generating final dashboard...");
    }
}
```

**Attach the Observer to your Test Runner XML (`testng.xml`):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org">
<suite name="Regression Suite">
    
    <!-- Registering the reporting observer globally -->
    <listeners>
        <listener class-name="listeners.ReportingObserver" />
    </listeners>

    <test name="UI Application Tests">
        <classes>
            <class name="tests.LoginTest" />
            <class name="tests.DashboardTest" />
        </classes>
    </test>
</suite>
```
