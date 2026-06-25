## Pattern 3 — Singleton

### What it is

Singleton ensures a class has exactly one instance for the lifetime of the program or test session. Any code that asks for that instance always gets the same one.

### Why it applies to test automation

Browser driver setup is expensive. If every test class created a new browser, your suite would be 10x slower. The Singleton pattern lets you create the browser once (per worker) and share it across all tests in that worker. The same applies to database connection pools, configuration loaders, and logger instances.

### When to use

Use it for expensive resources that are safe to share: browser driver pool, configuration object, connection pool, logger.

### When NOT to use

Do not use Singleton for the browser Page object or BrowserContext — those must be isolated per test to prevent state leaking between tests. Do not use it when running parallel workers — a true singleton shared across parallel workers will cause race conditions.

---

### TypeScript — Playwright

```typescript
// config/config.singleton.ts
// Configuration loaded once and shared across all tests in the process
// Safe to singleton because it is read-only

interface AppConfig {
  baseUrl: string;
  apiUrl: string;
  adminEmail: string;
  adminPassword: string;
  dbConnectionString: string;
  timeout: number;
}

export class Config {
  private static instance: Config | null = null;
  private readonly config: AppConfig;

  // Private constructor: nobody can do new Config() from outside
  private constructor() {
    // Load from environment — validate at startup, not mid-test
    this.config = {
      baseUrl:            this.require('BASE_URL'),
      apiUrl:             this.require('API_URL'),
      adminEmail:         this.require('ADMIN_EMAIL'),
      adminPassword:      this.require('ADMIN_PASSWORD'),
      dbConnectionString: this.require('DATABASE_URL'),
      timeout:            parseInt(process.env['TIMEOUT'] ?? '30000', 10),
    };
  }

  // Only way to get the instance
  static getInstance(): Config {
    if (Config.instance === null) {
      Config.instance = new Config();
    }
    return Config.instance;
  }

  // For testing the config singleton itself — reset between tests
  static resetForTesting(): void {
    Config.instance = null;
  }

  get baseUrl():            string { return this.config.baseUrl; }
  get apiUrl():             string { return this.config.apiUrl; }
  get adminEmail():         string { return this.config.adminEmail; }
  get adminPassword():      string { return this.config.adminPassword; }
  get dbConnectionString(): string { return this.config.dbConnectionString; }
  get timeout():            number { return this.config.timeout; }

  private require(key: string): string {
    const value = process.env[key];
    if (!value) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
    return value;
  }
}

// Usage anywhere in the framework:
// const config = Config.getInstance();
// config.baseUrl  → 'https://staging.example.com'
```

```typescript
// utils/logger.singleton.ts
// One logger instance per worker — writes structured logs

import fs from 'fs';
import path from 'path';

type LogLevel = 'DEBUG' | 'INFO' | 'WARN' | 'ERROR';

interface LogEntry {
  timestamp: string;
  level: LogLevel;
  message: string;
  data?: unknown;
}

export class TestLogger {
  private static instance: TestLogger | null = null;
  private readonly stream: fs.WriteStream;

  private constructor() {
    const logDir = path.join(process.cwd(), 'logs');
    if (!fs.existsSync(logDir)) {
      fs.mkdirSync(logDir, { recursive: true });
    }

    const logFile = path.join(logDir, `test-run-${Date.now()}.log`);
    this.stream   = fs.createWriteStream(logFile, { flags: 'a' });
  }

  static getInstance(): TestLogger {
    if (TestLogger.instance === null) {
      TestLogger.instance = new TestLogger();
    }
    return TestLogger.instance;
  }

  private write(level: LogLevel, message: string, data?: unknown): void {
    const entry: LogEntry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      data,
    };
    this.stream.write(JSON.stringify(entry) + '\n');
    if (level === 'ERROR') {
      console.error(`[${level}] ${message}`, data ?? '');
    }
  }

  info(message: string, data?: unknown):  void { this.write('INFO',  message, data); }
  warn(message: string, data?: unknown):  void { this.write('WARN',  message, data); }
  error(message: string, data?: unknown): void { this.write('ERROR', message, data); }
  debug(message: string, data?: unknown): void { this.write('DEBUG', message, data); }

  close(): void { this.stream.end(); }
}

// Usage in any file — always the same instance:
// TestLogger.getInstance().info('Test started', { testName: 'login' });
```

---

### Java — Selenium

```java
// src/main/java/driver/DriverManager.java
// Thread-safe Singleton for WebDriver
// Uses ThreadLocal to give each parallel thread its own driver
// (ThreadLocal Singleton — the correct approach for parallel Selenium)

package driver;

import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;

public class DriverManager {

  // ThreadLocal: each thread (parallel test) gets its own WebDriver
  // This looks like singleton but is actually per-thread
  private static final ThreadLocal<WebDriver> driverThreadLocal = new ThreadLocal<>();

  // Private constructor: blocks new DriverManager()
  private DriverManager() {}

  public static WebDriver getDriver() {
    if (driverThreadLocal.get() == null) {
      driverThreadLocal.set(createDriver());
    }
    return driverThreadLocal.get();
  }

  public static void quitDriver() {
    WebDriver driver = driverThreadLocal.get();
    if (driver != null) {
      driver.quit();
      driverThreadLocal.remove(); // Important: prevent memory leak
    }
  }

  private static WebDriver createDriver() {
    String browser = System.getProperty("browser", "chrome").toLowerCase();

    return switch (browser) {
      case "firefox" -> {
        WebDriverManager.firefoxdriver().setup();
        yield new FirefoxDriver();
      }
      default -> {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        if (Boolean.parseBoolean(System.getProperty("headless", "false"))) {
          options.addArguments("--headless=new");
        }
        options.addArguments("--no-sandbox", "--disable-dev-shm-usage");
        yield new ChromeDriver(options);
      }
    };
  }
}
```

```java
// src/test/java/base/BaseTest.java
// TestNG base test using the ThreadLocal DriverManager

package base;

import driver.DriverManager;
import org.openqa.selenium.WebDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;

public abstract class BaseTest {

  protected WebDriver driver;

  @BeforeMethod
  public void setUp() {
    // Gets driver — creates it if this thread does not have one yet
    driver = DriverManager.getDriver();
    driver.manage().window().maximize();
  }

  @AfterMethod
  public void tearDown() {
    // Quit and remove from ThreadLocal
    DriverManager.quitDriver();
  }
}
```

---

Let’s break down exactly what is happening in this line of code. This is a classic snippet used in Java-based Selenium frameworks (like TestNG or Cucumber) to make tests run in parallel safely.

Here is the simple, junior-engineer-friendly breakdown of why each keyword is there and what happens if you delete it.

---

## 1. The Breakdown of Every Keyword

### `private`

* **What it means:** This variable can only be accessed *inside* this specific class.
* **Why use it:** You don't want a test file accidentally modifying or overriding your driver instance directly. Tests should interact with the driver through specific methods (like `getDriver()`).
* **If you remove it:** It defaults to "package-private." Any other class in the same folder can see and modify your `driverThreadLocal`, which breaks security and encapsulation.

### `static`

* **What it means:** The variable belongs to the **Class itself**, not to a specific object instance of the class.
* **Why use it:** In test automation, you want *one global driver container* that all your test methods can refer to. Without `static`, every time you create an instance of your Page Object or Base Test class, a brand-new, empty variable would be created.
* **If you remove it:** The variable becomes an instance variable. If Test A and Test B both try to fetch the driver, they won't be looking at the same container, and you'll get a `NullPointerException`.

### `final`

* **What it means:** Once this variable is initialized, it **cannot be reassigned** to a new object.
* **Why use it:** It is a safety guardrail. You want to make sure no one accidentally types `driverThreadLocal = new ThreadLocal<>()` somewhere else in the code, which would wipe out all active browser sessions.
* **If you remove it:** The code still runs, but you lose that safety net. Someone could accidentally reassign the variable and crash your framework.

### `ThreadLocal<WebDriver>`

* **What it means:** This is the most important part. It is a special Java container that assigns a **private pocket of memory** to whatever thread is currently running.
* **Why use it:** If you run 3 tests in parallel, Java spawns 3 **Threads**. If you just used a normal `WebDriver driver`, all three threads would fight over the exact same browser window. By using `ThreadLocal`, Thread 1 gets its own isolated Chrome browser, Thread 2 gets its own, and they never cross paths or interfere.

---

## 2. What if we don't write `ThreadLocal`? (The Ultimate Multi-Thread Danger)

Let’s visualize what happens if you change that line to a standard static driver:

```java
// Danger Zone: No ThreadLocal!
private static WebDriver driver; 

```

### The Scenario: Running 2 Tests in Parallel (Thread 1 and Thread 2)

1. **Thread 1** starts running `Test_Login`. It opens a Chrome browser and inputs the username.
2. At the exact same millisecond, **Thread 2** starts running `Test_Search`. It executes `driver.get("https://google.com")`.
3. Because `driver` is `static` and shared, **Thread 2 hijacks Thread 1's browser!** 4. Suddenly, Thread 1's login page vanishes, replaced by Google. Thread 1 tries to click the login button, can't find it, and throws a `NoSuchElementException`. Your tests completely fall apart.

### The Fix with `ThreadLocal`:

When you use `driverThreadLocal = new ThreadLocal<>()`:

* When Thread 1 calls `driverThreadLocal.get()`, Java looks inside Thread 1's private pocket and hands it **Browser A**.
* When Thread 2 calls `driverThreadLocal.get()`, Java looks inside Thread 2's private pocket and hands it **Browser B**.

They can run at the exact same time on the same machine, completely isolated and beautifully safe!
