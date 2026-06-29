# Test Automation Lifecycle Framework (Template Method Pattern via Annotations)
This framework demonstrates how to implement the **Template Method Pattern** effectively using native testing framework annotations (`@BeforeMethod` / `@AfterMethod`). 

This architecture hides infrastructure complexity (browser setups, reporting logs, screenshot grabs) inside a base template class, leaving your test classes clean, readable, and focused entirely on verification actions.
---## 1. The Base Template ClassThe base class enforces the execution sequence structure. The testing runner triggers these hooks automatically before and after every single test method.
```java
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import java.lang.reflect.Method;

public abstract class BaseTestTemplate {

    /**
     * Step 1 of Template: Automatic initialization before EVERY test method.
     */
    @BeforeMethod
    public final void setupEnvironment(Method method) {
        System.out.println("[START LOG] --- Starting Test: " + method.getName() + " ---");
        initializeBrowser();
    }

    /**
     * Step 3 of Template: Automatic teardown after EVERY test method.
     */
    @AfterMethod
    public final void teardownEnvironment(Method method) {
        takeScreenshotOnFailure();
        closeBrowser();
        System.out.println("[END LOG] --- Finished Test: " + method.getName() + " ---\n");
    }

    // Shared internal framework infrastructure operations
    private void initializeBrowser() { 
        System.out.println("[CORE] Booting browser instance."); 
    }
    
    private void closeBrowser() { 
        System.out.println("[CORE] Closing browser instance safely."); 
    }
    
    private void takeScreenshotOnFailure() { 
        System.out.println("[CORE] Checking framework health / Saving screenshots."); 
    }
}
```
---## 2. The Clean Test Class (Multiple Test Steps)By inheriting the base template, you write standard, flat test actions without dealing with boilerplate wrapper blocks or nested execution loops.
```java
import org.testng.annotations.Test;

public class UserProfileTestSuite extends BaseTestTemplate {

    @Test
    public void testUpdateUsername() {
        // Step 2 of Template: Pure test logic execution
        System.out.println("[TEST] Step: Clearing username input field.");
        System.out.println("[TEST] Step: Typing 'New_User_123' and clicking Save.");
    }

    @Test
    public void testUpdatePassword() {
        // Step 2 of Template: Pure test logic execution
        System.out.println("[TEST] Step: Typing new password into security fields.");
        System.out.println("[TEST] Step: Verifying success validation message appears.");
    }
}
```
---## 3. Automatic Console Execution OutputWhen the suite is executed via a test runner, the base framework automatically manages the state boundaries surrounding your validation steps:
```text
[START LOG] --- Starting Test: testUpdateUsername ---
[CORE] Booting browser instance.
[TEST] Step: Clearing username input field.
[TEST] Step: Typing 'New_User_123' and clicking Save.
[CORE] Checking framework health / Saving screenshots.
[CORE] Closing browser instance safely.
[END LOG] --- Finished Test: testUpdateUsername ---

[START LOG] --- Starting Test: testUpdatePassword ---
[CORE] Booting browser instance.
[TEST] Step: Typing new password into security fields.
[TEST] Step: Verifying success validation message appears.
[CORE] Checking framework health / Saving screenshots.
[CORE] Closing browser instance safely.
[END LOG] --- Finished Test: testUpdatePassword ---
```
---## Key Benefits* **Zero Boilerplate**: Test scripts remain highly maintainable and free of custom script-wrapping code blocks.* **Guaranteed Cleanup**: Environment teardown logic runs unconditionally in the background, minimizing leaked browser instances.* **Separation of Concerns**: Engineers focusing on core business flows are completely isolated from low-level infrastructure configurations.

------------------------------
If you want to enhance this markdown file with more enterprise features, let me know if you would like to add:

* An ExtentReports / Allure Reporting engine integration snippet inside the hooks.
* Configuration steps to switch this example from TestNG over to JUnit 5 (@BeforeEach/@AfterEach) rules.


[1] [https://marketplace.visualstudio.com](https://marketplace.visualstudio.com/items?itemName=jmkrivocapich.drawfolderstructure)
