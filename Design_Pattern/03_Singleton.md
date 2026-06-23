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

