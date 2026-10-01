# Day 8 — Classes & OOP in JS

Written for someone with real experience — simple language, no fluff, focused on what actually comes up in a senior-level interview and in day-to-day automation code.

---

## 1. `class`, Constructor, and the Basics — Quick Refresher

You already know this, so just the key points worth saying out loud in an interview:

```js
class User {
  constructor(name, role) {
    this.name = name;
    this.role = role;
  }

  describe() {
    return `${this.name} is a ${this.role}`;
  }
}

const u = new User("Alice", "admin");
console.log(u.describe()); // "Alice is a admin"
```

The `constructor` runs once, when `new User(...)` is called. Any method you write inside the class body (`describe()` here) does **not** get copied onto every instance — it sits once on `User.prototype`, and every instance shares it through the prototype chain. That's worth mentioning proactively — it shows you're not just using the syntax, you know what it compiles down to.

```js
console.log(u.hasOwnProperty("describe")); // false — it's on the prototype, not on `u`
```

---

## 2. `extends` and `super` — Inheritance Done Right

```js
class BasePage {
  constructor(driver) {
    this.driver = driver;
  }

  async navigateTo(url) {
    await this.driver.goto(url);
  }
}

class LoginPage extends BasePage {
  constructor(driver) {
    super(driver); // must run first — sets up `this.driver` from the parent
    this.usernameField = "#username";
    this.passwordField = "#password";
  }

  async login(username, password) {
    await this.driver.fill(this.usernameField, username);
    await this.driver.fill(this.passwordField, password);
    await this.driver.click("#submit");
  }
}

const login = new LoginPage(driver);
await login.navigateTo("https://app.com/login");
await login.login("alice", "secret123");
```

Two things worth saying clearly in an interview:

**`super(...)` must be called before you touch `this` in a child constructor.** If you try `this.usernameField = "#username"` before calling `super(driver)`, JS throws a `ReferenceError`. The reason: `this` doesn't exist yet in a subclass until the parent constructor has finished setting it up. This is different from how `this` works in a plain base class constructor, and it's a real gotcha people hit.

**`super.methodName()` lets you call the parent's version of a method you've overridden**, instead of fully replacing it:

```js
class AdminPage extends BasePage {
  async navigateTo(url) {
    console.log("Logging admin navigation...");
    await super.navigateTo(url); // still does the parent's actual navigation
  }
}
```

---

## 3. Static Methods and Properties

Static members belong to the class itself, not to instances — think of them as utility/helper-level things that don't need an actual object to make sense.

```js
class TestDataFactory {
  static createUser(overrides = {}) {
    return { id: 1, name: "Test User", active: true, ...overrides };
  }

  static defaultTimeout = 5000;
}

const user = TestDataFactory.createUser({ name: "Bob" });
console.log(TestDataFactory.defaultTimeout); // 5000

// You do NOT create an instance to use these
const factory = new TestDataFactory();
console.log(factory.createUser); // undefined — static methods aren't on instances
```

**When to reach for static:** anything that's a pure helper tied conceptually to the class but doesn't need instance state — factories, constants, utility calculations. In automation code, this shows up constantly for mock data builders and shared config values.

---

## 4. Getters and Setters

These let you make a property *look* like a plain field from the outside, while actually running logic behind the scenes.

```js
class Order {
  constructor(items) {
    this._items = items; // convention: underscore = "treat as internal"
  }

  get total() {
    return this._items.reduce((sum, item) => sum + item.price, 0);
  }

  set items(newItems) {
    if (!Array.isArray(newItems) || newItems.length === 0) {
      throw new Error("Order must have at least one item");
    }
    this._items = newItems;
  }
}

const order = new Order([{ price: 100 }, { price: 50 }]);
console.log(order.total); // 150 — looks like a plain property, but it's computed live

order.items = [{ price: 200 }]; // goes through validation in the setter
console.log(order.total); // 200
```

**Interview line:** "Getters and setters let me control how a property is read or written without changing the calling code — `order.total` reads like a field, but it's actually a method running a calculation, and `order.items = ...` reads like a plain assignment but runs validation behind the scenes. It's a clean way to add guardrails without breaking the simple, field-like API the caller is used to."

---

## 5. Private Fields (`#field`) — the Modern Way to Hide State

Before `#fields`, people used closures (see Day 6) or just an underscore naming convention (`_balance`) that was private "by agreement," not actually enforced. `#fields` are truly private — not accessible or even visible from outside the class, at all.

```js
class BankAccount {
  #balance; // truly private — cannot be accessed as account.#balance from outside

  constructor(initialBalance) {
    this.#balance = initialBalance;
  }

  deposit(amount) {
    this.#balance += amount;
    return this.#balance;
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount(100);
account.deposit(50);
console.log(account.getBalance()); // 150
console.log(account.#balance); // ❌ SyntaxError — not just "undefined", it's a parse-time error
```

**Interview line:** "The underscore convention (`_balance`) was always just a signal to other developers — nothing actually stopped you from touching it. `#fields` are enforced by the language itself; trying to access `account.#balance` from outside the class is a syntax error, not just bad practice. If I need real encapsulation today, I reach for `#fields` directly instead of closures, unless I specifically need the closure's flexibility, like with factory functions instead of classes."

---

## 6. How a JS Class Differs From a Java/C# Class Under the Hood — the Core Interview Question

Say this plainly, in this order:

**1. There's no real "class" at runtime — it's sugar over prototypes.**
In Java or C#, a class is a compile-time construct — the runtime has a genuine concept of a class, separate from objects. In JS, `class SomeClass { ... }` compiles down to a constructor function plus methods sitting on `SomeClass.prototype`. At runtime, `typeof SomeClass` is `"function"`, not some special "class" type.

```js
class Foo {}
console.log(typeof Foo); // "function" — still just a function under the hood
```

**2. Inheritance is dynamic and object-to-object, not fixed at compile time.**
In JS, `extends` just wires up the prototype chain — `ChildClass.prototype`'s prototype becomes `ParentClass.prototype`. That link can technically be changed at runtime with `Object.setPrototypeOf`, something that's just not a concept in Java/C#'s class model at all. You'd never do this in real code, but it shows the model is genuinely different underneath, not just stylistically different.

**3. No real access modifiers until recently, and even now they're limited.**
Java/C# have `private`, `protected`, `public` baked deeply into the type system, checked at compile time. JS only got real privacy with `#fields` (ES2022), and there's still no `protected` — no "visible to subclasses but not outside" concept at all.

**4. No method overloading.**
Java/C# let you define multiple versions of a method with different parameter types/counts, and the compiler picks the right one. JS has no such concept — a class can only have one method with a given name; you fake overloading with default parameters, rest parameters, or manually checking `arguments` inside a single method.

**5. No interfaces or abstract classes as language features.**
Java/C# have `interface` and `abstract class` as first-class, compiler-enforced constructs. JS has neither — you can *simulate* an abstract class by throwing an error in a base method that's supposed to be overridden, but nothing stops a subclass from skipping it, and there's no compile-time check at all:

```js
class BasePage {
  async open() {
    throw new Error("open() must be implemented by subclass");
  }
}
```

That's a convention, not a language guarantee — worth saying this difference out loud, because it shows you understand JS doesn't give you compile-time safety the way a typed, class-based language does. (TypeScript adds interfaces and real `private`/`protected` back in, but that's compile-time only — it still compiles away to plain JS classes with none of that enforcement left at runtime.)

**One-line summary to give if asked to be concise:** "A JS class is just a readable wrapper around functions and prototypes — dynamic, flexible, and enforced mostly by convention rather than the type system. A Java/C# class is a real compile-time construct with strict access control, method overloading, and interfaces baked in. JS trades strict guarantees for runtime flexibility."

---

## 7. Test Tie-In: Page Object Model (POM) with a Base Class and Inheritance

This is exactly where `class`, `extends`, `super`, and static methods show up together in real automation code — not academic, this is how you'd actually structure a Playwright or Selenium framework with more than a handful of pages.

### The base class — shared behavior every page needs

```js
class BasePage {
  constructor(page) {
    this.page = page; // Playwright page object, or a Selenium driver wrapper
  }

  async navigateTo(url) {
    await this.page.goto(url);
  }

  async waitForElement(selector, timeout = 5000) {
    await this.page.waitForSelector(selector, { timeout });
  }

  async click(selector) {
    await this.waitForElement(selector);
    await this.page.click(selector);
  }

  async fill(selector, value) {
    await this.waitForElement(selector);
    await this.page.fill(selector, value);
  }

  async getText(selector) {
    await this.waitForElement(selector);
    return this.page.textContent(selector);
  }
}
```

### A specific page, built on top of the base

```js
class LoginPage extends BasePage {
  constructor(page) {
    super(page);
    this.selectors = {
      username: "#username",
      password: "#password",
      submitBtn: "#login-submit",
      errorMessage: "#error-message",
    };
  }

  async login(username, password) {
    await this.fill(this.selectors.username, username);   // inherited from BasePage
    await this.fill(this.selectors.password, password);   // inherited from BasePage
    await this.click(this.selectors.submitBtn);            // inherited from BasePage
  }

  async getErrorMessage() {
    return this.getText(this.selectors.errorMessage); // inherited from BasePage
  }
}
```

### Another page, reusing the exact same base

```js
class DashboardPage extends BasePage {
  constructor(page) {
    super(page);
    this.selectors = {
      welcomeMessage: "#welcome",
      logoutBtn: "#logout",
    };
  }

  async getWelcomeText() {
    return this.getText(this.selectors.welcomeMessage);
  }

  async logout() {
    await this.click(this.selectors.logoutBtn);
  }
}
```

### Using them in a test

```js
test("login redirects to dashboard", async ({ page }) => {
  const loginPage = new LoginPage(page);
  const dashboardPage = new DashboardPage(page);

  await loginPage.navigateTo("https://app.com/login"); // inherited from BasePage
  await loginPage.login("alice@test.com", "correctpassword");

  const welcome = await dashboardPage.getWelcomeText();
  expect(welcome).toBe("Welcome, Alice");
});
```

**Why this design is actually good, and worth explaining proactively in an interview:**

- `waitForElement`, `click`, `fill`, `getText` are written **once**, in `BasePage`, and every page class gets them for free through `extends`. If you need to change the waiting strategy for the whole framework — say, swap in a retry mechanism — you change it in one place, and every page automatically picks it up.
- Each page class only holds what's specific to it: its selectors, and any page-specific actions (`login`, `getWelcomeText`). That's the real value of inheritance here — shared mechanics live once, page-specific details live close to where they're used.
- Using a static factory for commonly-needed combinations is a nice addition on top of this pattern:

```js
class PageFactory {
  static createLoginPage(page) {
    return new LoginPage(page);
  }
  static createDashboardPage(page) {
    return new DashboardPage(page);
  }
}
```

**A senior-level point worth raising unprompted:** deep inheritance chains (`BasePage` → `AuthenticatedBasePage` → `AdminBasePage` → `AdminUserListPage`) get hard to maintain fast — you end up hunting through four files to understand one method's actual behavior. In practice, a flatter hierarchy (one `BasePage`, then each concrete page extends it directly) plus composition for cross-cutting stuff (like a `Logger` or `RetryHelper` passed in or imported, not inherited) tends to age much better than a deep class tree. This is a good thing to say if an interviewer asks "would you add more inheritance layers here?" — showing you know inheritance isn't automatically the right tool just because it's available.

---

## 8. Interview Q&A Script

**Q: How is a JS class different from a Java/C# class under the hood?**
> "A JS class isn't a real runtime construct — it compiles down to a constructor function with methods sitting on its prototype, so `typeof SomeClass` is actually `'function'`. Inheritance via `extends` just links prototypes together, and that link can even be changed at runtime, which isn't a concept in Java or C#'s model at all. JS also doesn't have real method overloading, doesn't have interfaces or abstract classes as language features, and only got genuine private fields recently with `#fields` — before that, privacy was just a naming convention. Overall, a JS class trades the strict compile-time guarantees you get in Java or C# for a lot more runtime flexibility."

**Q: What's the difference between `#privateField` and just naming a property `_privateField`?**
> "The underscore is just a convention — nothing stops any code from reading or writing `obj._privateField` directly. `#field` is enforced by the language itself; trying to access it from outside the class is a syntax error, not just bad practice that linting might catch. I use `#fields` now whenever I actually need real encapsulation."

**Q: Why does `super()` have to be called before using `this` in a subclass constructor?**
> "In a subclass, `this` isn't actually created until the parent constructor runs — JS needs the parent to finish setting up the object first. If you try to touch `this` before calling `super()`, you get a `ReferenceError`, because there's technically no `this` yet to touch."

**Q: When would you use a getter/setter instead of a plain property?**
> "When I need to run logic on read or write without changing how the calling code looks — like computing a value on the fly (`order.total` as a getter instead of a stored, possibly stale field), or validating input before it's stored (a setter that rejects invalid data). It keeps the external API looking like a simple property while adding real behavior behind it."

**Q: In a Page Object Model, what goes in the base class vs the specific page class?**
> "The base class holds genuinely shared mechanics — waiting strategies, click/fill helpers, navigation — things every page needs regardless of what it actually shows. The specific page class holds only what's unique to that page: its selectors and its specific user actions, like `login()` on a login page. That split means if I need to change how waiting works across the whole suite, I change it once in the base class instead of in every single page file."

---

## 9. One-Page Cheat Sheet

- **`class`** is sugar over constructor functions + prototypes — methods live once on `ClassName.prototype`, shared by every instance, not copied per instance.
- **`extends`/`super`** wire up the prototype chain. `super(...)` must run before touching `this` in a subclass constructor. `super.method()` calls the parent's version of an overridden method.
- **Static members** (`static method() {}`, `static prop = ...`) belong to the class itself, not instances — good for factories, constants, shared helpers.
- **Getters/setters** let a property look plain from the outside while running real logic (computed values, validation) behind the scenes.
- **`#privateField`** is real, language-enforced privacy (ES2022) — not just a naming convention like `_field`. Accessing it from outside throws a syntax error.
- **JS class vs Java/C# class, in one line:** no real runtime "class" concept (just functions + prototypes), dynamic/reassignable inheritance links, no method overloading, no interfaces/abstract classes as language features, and only recent, partial access control via `#fields` (still no `protected`).
- **POM pattern:** put shared mechanics (waits, clicks, fills, navigation) in a `BasePage`; put page-specific selectors and actions in subclasses that `extends BasePage`. Keep the hierarchy flat — prefer composition over deep multi-level inheritance chains as the framework grows.
