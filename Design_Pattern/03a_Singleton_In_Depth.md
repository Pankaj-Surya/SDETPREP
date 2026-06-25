
## Singelton With Double Check

## The Thread-Safe Java Implementation

```java
public class Config {
    // 1. volatile prevents instruction reordering
    private static volatile Config instance = null; 

    private Config() {} // Private constructor

    public static Config getInstance() {
        // First check (no locking)
        if (instance == null) { 
            // Lock the class block, not the whole method
            synchronized (Config.class) { 
                // Second check (inside the lock)
                if (instance == null) { 
                    instance = new Config();
                }
            }
        }
        return instance;
    }
}

```

---

## Breaking Down the Pieces

### 1. Why the "Double Check"?

If we just put `synchronized` on the whole method (e.g., `public static synchronized Config getInstance()`), it would be safe, but **terribly slow**. Every single time your app needs the config, threads would have to line up and wait for the lock, even after the instance has already been created.

* **The First Check:** Prevents threads from acquiring an expensive lock if the instance *already* exists. 99.9% of the time, the app bypasses the lock entirely here.
* **The Synchronized Block:** Ensures only *one* thread can enter the creation phase at a time.
* **The Second Check:** Essential because if Thread A and Thread B both pass the first check, Thread A gets the lock and creates the instance. When Thread A leaves, Thread B enters the block. Without the *second* check, Thread B would blindly create a second instance.

### 2. Why the `volatile` Keyword?

This is the most subtle and complex part of the pattern. Without `volatile`, the double-checked locking pattern is actually **broken and dangerous** in Java due to how CPUs and compilers optimize code.

When a computer executes `instance = new Config();`, it performs three distinct operations under the hood:

1. **Allocate memory** for the `Config` object.
2. **Initialize** the `Config` object (run the constructor code).
3. **Assign** the `instance` variable to point to that allocated memory address.

However, the Java Virtual Machine (JVM) utilizes **Instruction Reordering** to optimize performance. It might swap steps 2 and 3:

1. Allocate memory.
2. Assign `instance` to point to the memory (The variable is **no longer null**!).
3. Initialize the object.

**The Danger:** If Thread A gets reordered this way, it points `instance` to empty memory before initializing it. At that exact microsecond, Thread B calls `getInstance()`. It hits the *first check*, sees `instance != null`, and immediately returns that uninitialized, empty object to your application—causing a massive crash or corrupted state.

> **What `volatile` does:** It enforces a "happens-before" relationship and creates a memory barrier. It tells the compiler, *"Do not reorder the instructions around this variable, and make changes instantly visible to all CPU caches."*

---

## A Modern Alternative (The Bill Pugh Holder)

While Double-Checked Locking is a classic interview favorite, it is notorious for being easy to mess up. Modern Java developers often prefer the **Initialization-on-demand holder idiom**. It achieves 100% thread safety and lazy initialization purely through Java's native class loading mechanisms, completely omitting the need for `volatile` or `synchronized`:

```java
public class Config {
    private Config() {}

    // Inner static class is only loaded when getInstance() is called
    private static class Holder {
        private static final Config INSTANCE = new Config();
    }

    public static Config getInstance() {
        return Holder.INSTANCE;
    }
}

```

### 

The short answer is **no, you do not need double-checked locking, `synchronized`, or `volatile` in Playwright TypeScript.** In fact, you *cannot* use them because Node.js (the runtime engine behind Playwright and TypeScript) uses a fundamentally different concurrency model than Java.

Here is why this pattern doesn't exist in TypeScript, and how we handle singletons in Playwright instead.

---

## Why TypeScript Doesn't Need It

### 1. JavaScript/TypeScript is Single-Threaded

Java uses a multi-threaded architecture where dozens of threads can execute the exact same block of code at the exact same time. This is why Java needs synchronization locks and memory barriers (`volatile`).

Node.js executes your TypeScript code on a **single main thread** using an **Event Loop**.

Because only one line of JavaScript code can ever execute at any single microsecond on that main thread, **race conditions during variable assignment are impossible**. Two pieces of code cannot hit your `if (instance === null)` block at the exact same physical millisecond.

### 2. Playwright's Multi-Worker Architecture

You might wonder: *"But Playwright runs tests in parallel using multiple workers!"* Yes, it does. However, Playwright achieves parallelism by launching **entirely separate Node.js OS processes (workers)**, not shared threads.

* Worker 1 has its own completely isolated memory.
* Worker 2 has its own completely isolated memory.

They do not share a global heap memory. If Worker 1 creates a Singleton instance of your `Config` class, Worker 2 cannot see it or corrupt it; Worker 2 will naturally spin up its own isolated instance of the Singleton inside its own process.

---

## How to Write a Singleton in Playwright TypeScript

Because of Node.js's module caching system, writing a thread-safe global Singleton in TypeScript is actually incredibly simple. You don't even need a `getInstance()` method.

### The Idiomatic TypeScript Way (Module Singleton)

In Node.js, when you `import` a file, that file is executed **exactly once**, and the exported result is cached in memory. Any subsequent imports of that file just grab the cached instance.

```typescript
// config.ts
class Config {
    public baseUrl: string = 'https://api.example.com';
    
    constructor() {
        // This runs EXACTLY ONCE per Playwright worker process
        console.log('Config initialized!');
    }
}

// Export a direct INSTANCE of the class, not the class itself
export const config = new Config();

```

### How you use it in your Playwright Tests:

```typescript
// test1.spec.ts
import { test } from '@playwright/test';
import { config } from './config'; // Grabs the instance

test('Test 1', async ({ page }) => {
    await page.goto(config.baseUrl);
});

```

### Summary

The short answer is **no, you do not need double-checked locking, `synchronized`, or `volatile` in Playwright TypeScript.** In fact, you *cannot* use them because Node.js (the runtime engine behind Playwright and TypeScript) uses a fundamentally different concurrency model than Java.

Here is why this pattern doesn't exist in TypeScript, and how we handle singletons in Playwright instead.

---

## Why TypeScript Doesn't Need It

### 1. JavaScript/TypeScript is Single-Threaded

Java uses a multi-threaded architecture where dozens of threads can execute the exact same block of code at the exact same time. This is why Java needs synchronization locks and memory barriers (`volatile`).

Node.js executes your TypeScript code on a **single main thread** using an **Event Loop**.

Because only one line of JavaScript code can ever execute at any single microsecond on that main thread, **race conditions during variable assignment are impossible**. Two pieces of code cannot hit your `if (instance === null)` block at the exact same physical millisecond.

### 2. Playwright's Multi-Worker Architecture

You might wonder: *"But Playwright runs tests in parallel using multiple workers!"* Yes, it does. However, Playwright achieves parallelism by launching **entirely separate Node.js OS processes (workers)**, not shared threads.

* Worker 1 has its own completely isolated memory.
* Worker 2 has its own completely isolated memory.

They do not share a global heap memory. If Worker 1 creates a Singleton instance of your `Config` class, Worker 2 cannot see it or corrupt it; Worker 2 will naturally spin up its own isolated instance of the Singleton inside its own process.

---

## How to Write a Singleton in Playwright TypeScript

Because of Node.js's module caching system, writing a thread-safe global Singleton in TypeScript is actually incredibly simple. You don't even need a `getInstance()` method.

### The Idiomatic TypeScript Way (Module Singleton)

In Node.js, when you `import` a file, that file is executed **exactly once**, and the exported result is cached in memory. Any subsequent imports of that file just grab the cached instance.

```typescript
// config.ts
class Config {
    public baseUrl: string = 'https://api.example.com';
    
    constructor() {
        // This runs EXACTLY ONCE per Playwright worker process
        console.log('Config initialized!');
    }
}

// Export a direct INSTANCE of the class, not the class itself
export const config = new Config();

```

### How you use it in your Playwright Tests:

```typescript
// test1.spec.ts
import { test } from '@playwright/test';
import { config } from './config'; // Grabs the instance

test('Test 1', async ({ page }) => {
    await page.goto(config.baseUrl);
});

```

### Summary

* **Java:** Needs double-checked locking, `synchronized`, and `volatile` because multiple threads share the same memory and can execute code concurrently.
* **Playwright TypeScript:** Does not need any of it. Node.js is single-threaded per process, and Playwright isolates parallel workers into separate OS processes, making your code natively safe from thread race conditions.
