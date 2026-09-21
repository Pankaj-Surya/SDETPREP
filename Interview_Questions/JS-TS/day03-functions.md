# Day 3 — Functions Deep Dive

Everything you need to *understand it*, *explain it out loud*, and *answer follow-ups* in an interview.

---

## 1. Function Declarations vs Function Expressions

```js
// Function declaration — hoisted with full body, callable before its line
function add(a, b) {
  return a + b;
}

// Function expression — NOT hoisted with a body; only the variable is hoisted
const subtract = function (a, b) {
  return a - b;
};

// Named function expression — name is only usable inside its own body (useful for recursion/stack traces)
const factorial = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1);
};
```

```js
console.log(add(2, 3));       // 5 — works, declarations are fully hoisted
console.log(subtract(5, 2));  // ❌ if called before the line: TypeError: subtract is not a function
                               //    (var would give "undefined is not a function", let/const give ReferenceError via TDZ)
```

**Interview line:** "Function declarations are hoisted completely — name and body — so you can call them anywhere in the scope. Function expressions are just a variable assignment; only the variable declaration is hoisted (or lands in the TDZ for `let`/`const`), not the function itself, so you can't call it before that line executes."

**Real-time example — why this matters for code organization:** in a utils file, a function declaration lets you organize "public API at top, helpers below" without worrying about order. A function expression forces you to define-before-use, which is usually the safer, more readable convention in modern codebases anyway.

---

## 2. Arrow Functions vs Regular Functions — the `this` difference

This is the single most important thing to understand about arrow functions, and the #1 interview topic in this section.

**Regular functions:** `this` is **dynamic** — determined by *how the function is called* (its call-site), not where it's defined.

**Arrow functions:** `this` is **lexical** — they don't have their own `this` at all. They capture `this` from the enclosing scope at the time they're *defined*, permanently.

### Real-time example: the classic broken `this` in a class/object method

```js
const cart = {
  items: ["Book", "Pen"],
  total: 0,

  // Regular function as a method — `this` is `cart` when called as cart.printItems()
  printItems: function () {
    this.items.forEach(function (item) {
      // ❌ Regular function passed to forEach — `this` here is NOT `cart`!
      // In non-strict mode, `this` is the global object; in strict mode / modules, it's `undefined`
      console.log(this.total, item); // TypeError or logs `undefined`
    });
  },

  printItemsFixed: function () {
    this.items.forEach((item) => {
      // ✅ Arrow function — `this` is inherited lexically from printItemsFixed,
      // where `this` correctly refers to `cart`
      console.log(this.total, item); // 0 Book, 0 Pen
    });
  },
};

cart.printItems();      // throws or logs wrong `this`
cart.printItemsFixed(); // works correctly
```

**Interview line:** "A regular function's `this` depends entirely on how it's invoked — as a method (`obj.fn()`), it's the object; as a plain callback, it defaults to the global object or `undefined` in strict mode. Arrow functions don't bind their own `this` — they close over `this` from their surrounding lexical scope at definition time, which is exactly what you want inside callbacks like `forEach`, `map`, or `setTimeout` where you want to keep referring to the outer object."

### Quick reference table

| | Regular function | Arrow function |
|---|---|---|
| Own `this` | Yes — depends on call-site | No — inherits from enclosing scope |
| Own `arguments` object | Yes | No — must use rest params |
| Can be used as a constructor (`new`) | Yes | No — throws `TypeError` |
| Has `prototype` property | Yes | No |
| Good for object methods that need `this` | ✅ Yes | ❌ No |
| Good for callbacks that need outer `this` | ❌ No (needs `.bind()`, `that = this`, etc.) | ✅ Yes |

## a. The this Keyword

* Regular Function: The value of this depends entirely on who called it. If you break the connection to the object, this changes.
* Arrow Function: It doesn't have its own this. It simply adopts the this from the surrounding code where it was written.
```
const user = {
  name: "Alice",
  // Regular Function
  regularLog: function() { console.log("Regular:", this.name); },
  // Arrow Function
  arrowLog: () => { console.log("Arrow:", this.name); }
};
// 1. Regular function looks at 'user' (the caller)
user.regularLog(); // Output: Regular: Alice
// 2. Arrow function ignores 'user' and looks at the global scope
user.arrowLog();   // Output: Arrow: undefined
```
------------------------------
## b. The arguments Object

* Regular Function: Automatically creates a built-in arguments array-like object containing every value you passed in.
* Arrow Function: Does not have the arguments object. If you want to grab multiple inputs, you must use rest parameters (...args).

```
// Regular Functionfunction showRegularArgs() {
  console.log(arguments); // Built-in variable exists automatically
}
showRegularArgs("A", "B"); // Output: ['A', 'B']
// Arrow Functionconst showArrowArgs = () => {
  console.log(arguments); // ❌ Throws ReferenceError (or grabs global arguments)
};
// Fixed Arrow Function using Rest Parametersconst showArrowFixed = (...myArgs) => {
  console.log(myArgs); // ✅ Works perfectly
};
showArrowFixed("A", "B"); // Output: ['A', 'B']
```
------------------------------
## c. Using the new Keyword (Constructors)

* Regular Function: Can be used as a blueprint to manufacture new objects.
* Arrow Function: Cannot build objects. Trying to do so causes JavaScript to crash.

```
// Regular Functionfunction Person(name) {
  this.name = name;
}const bob = new Person("Bob"); // ✅ Works perfectly
// Arrow Functionconst Animal = (type) => {
  this.type = type;
};const dog = new Animal("Dog"); // ❌ Throws TypeError: Animal is not a constructor
```
------------------------------
## d. The prototype Property

* Regular Function: Automatically comes with a prototype object, which is used to share methods across instances when using new.
* Arrow Function: Has no prototype property at all because it can never be used with new.
```
function regularFn() {}
console.log(regularFn.prototype); // Output: { constructor: regularFn }
const arrowFn = () => {};
console.log(arrowFn.prototype);   // Output: undefined
```
------------------------------
## e. Good for Object Methods

* Regular Function (✅ Yes): Ideal for object methods because this correctly points to the object itself.
* Arrow Function (❌ No): Terrible for object methods because this skips the object and points to the outer global environment.
```
const counter = {
  count: 10,
  // ✅ Regular function works because 'this' is 'counter'
  nextRegular() {
    this.count++;
    console.log("Regular count:", this.count);
  },
  // ❌ Arrow function fails because 'this' looks outside 'counter'
  nextArrow: () => {
    this.count++; 
    console.log("Arrow count:", this.count);
  }
};

counter.nextRegular(); // Output: Regular count: 11
counter.nextArrow();   // Output: Arrow count: NaN (undefined + 1)
```
------------------------------
## f. Good for Callbacks (Timers, Event Listeners, Loops)

* Regular Function (❌ No): Inside timers like setTimeout, a regular function loses track of your object because it gets called globally. You used to have to save it manually with var self = this.
* Arrow Function (✅ Yes): Effortlessly passes through into the timer while remembering exactly what this meant in your main method.
```
const timerObj = {
  message: "Time is up!",
  
  startWithRegular() {
    setTimeout(function() {
      // ❌ Fails: 'this' resets to the global Window inside setTimeout
      console.log(this.message); 
    }, 1000);
  },

  startWithArrow() {
    setTimeout(() => {
      // ✅ Works: Arrow function inherits 'this' from startWithArrow()
      console.log(this.message); 
    }, 1000);
  }
};

timerObj.startWithRegular(); // Output: undefined (after 1 second)
timerObj.startWithArrow();   // Output: "Time is up!" (after 1 second)
```
---

## 3. Default Parameters

```js
function createUser(name, role = "guest") {
  return { name, role };
}

createUser("Alice");           // { name: "Alice", role: "guest" }
createUser("Bob", "admin");    // { name: "Bob", role: "admin" }
createUser("Eve", undefined);  // { name: "Eve", role: "guest" } — undefined triggers the default
createUser("Sam", null);       // { name: "Sam", role: null }    — null does NOT trigger the default!
```

**Interview gotcha to mention:** default parameters only kick in for `undefined`, not for other falsy values like `null`, `0`, or `""`. This trips people up when a value legitimately might be `null`.

### Real-time example: defaults that depend on earlier parameters

```js
function createRectangle(width, height = width) {
  return { width, height };
}

createRectangle(5);      // { width: 5, height: 5 } — square by default
createRectangle(5, 10);  // { width: 5, height: 10 }
```

---

## 4. Rest Parameters (and how they replace `arguments`)

```js
// Old way — the `arguments` object (array-like, not a real array, doesn't exist in arrow functions)
function sumOld() {
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}

// Modern way — rest parameters (a real array, works everywhere including arrow functions)
function sumNew(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

sumNew(1, 2, 3, 4); // 10
```

```js
// Rest params can follow named params, capturing "everything else"
function logEvent(eventName, ...details) {
  console.log(eventName, details);
}

logEvent("click", "button", "submit", { x: 10, y: 20 });
// "click" ["button", "submit", { x: 10, y: 20 }]
```

**Interview line:** "Rest parameters give you a real array with all the array methods available immediately, unlike the old `arguments` object, which is array-*like* but not a real array and doesn't exist inside arrow functions at all — another reason arrow functions aren't a drop-in replacement for regular functions everywhere."

---

## 5. IIFEs (Immediately Invoked Function Expressions)

```js
(function () {
  const privateVar = "I'm not accessible outside";
  console.log("IIFE ran immediately");
})();

// Arrow function IIFE
(() => {
  console.log("Arrow IIFE ran immediately");
})();
```

**Why they exist:** before ES6 modules and `let`/`const` block scoping, IIFEs were the standard way to create a private scope — avoiding polluting the global namespace with helper variables, and enabling the classic "module pattern."

```js
const counterModule = (function () {
  let count = 0; // private — not accessible from outside
  return {
    increment: () => ++count,
    getCount: () => count,
  };
})();

counterModule.increment();
counterModule.increment();
console.log(counterModule.getCount()); // 2
console.log(counterModule.count);      // undefined — truly private
```

**Interview line:** "IIFEs solved the problem of variable leakage into the global scope before we had proper modules and block scoping. You still see them today for things like one-off setup/config code, or the module pattern for encapsulating private state — though ES modules have replaced most of that use case."

---

## 6. Test Tie-In: Writing Reusable Test Utility/Helper Functions Correctly

This is where the `this`-binding difference between arrow and regular functions becomes a *real, everyday bug* in test suites — not just theory.

### The bug: an arrow function breaks `this` inside a test framework hook

Many test frameworks (Jest, Mocha, Jasmine) bind a special `this` context inside `function () {}` test callbacks — used for things like increasing timeouts, sharing context between hooks, or accessing framework-provided helpers.

```js
// ❌ BROKEN — arrow function passed to Mocha's `it`
it("should complete within timeout", () => {
  this.timeout(5000); // ❌ TypeError: this.timeout is not a function
  // `this` here is NOT bound by Mocha — it's inherited from the
  // surrounding (module-level) scope, which has no `.timeout()` method
});

// ✅ CORRECT — regular function preserves Mocha's custom `this` binding
it("should complete within timeout", function () {
  this.timeout(5000); // ✅ works — Mocha binds `this` to the test context
});
```

**Interview line:** "Arrow functions ignore whatever `this` a framework tries to bind to them, because they don't have their own `this` — they just look up the enclosing scope. So if a test runner relies on binding a custom `this` (like Mocha does for hooks and timeouts), you must use a regular `function` there, not an arrow function, even though arrow functions are the 'modern default' everywhere else."

### Writing a correct, reusable test helper function

```js
// A generic reusable helper for asserting API responses across many test files
function expectSuccessResponse(response, expectedData) {
  expect(response.status).toBe(200);
  expect(response.body).toEqual(expectedData); // deep equality — see Day 2!
}

// Usage across multiple test files:
it("returns the correct user", async () => {
  const response = await getUser(1);
  expectSuccessResponse(response, { id: 1, name: "Alice" });
});
```

**Why a regular function (or a plain arrow function, here) works fine in this case:** this helper doesn't rely on any framework-injected `this` — it's a pure utility taking explicit arguments, which is the pattern you actually want for 90% of test helpers. The `this` problem only bites you when the *test callback itself* (the one passed directly to `it`/`describe`/`beforeEach`) needs framework-bound context.

### A helper factory using default + rest params together

```js
function createMockUser(overrides = {}, ...tags) {
  return {
    id: 1,
    name: "Test User",
    active: true,
    ...overrides,
    tags,
  };
}

createMockUser();                                  // default mock user, tags: []
createMockUser({ name: "Bob" });                   // override just the name
createMockUser({ active: false }, "vip", "beta");  // override + tags: ["vip", "beta"]
```

**Interview line:** "Good test helpers combine default parameters (so callers only override what they care about) with rest parameters (so the helper stays flexible for variable-length input) — this keeps test setup DRY across dozens of test files instead of duplicating full mock objects everywhere."

---

## 7. Interview Q&A Script

**Q: Why doesn't `this` work the same in arrow functions?**
> "Arrow functions don't have their own `this` binding at all — they don't create a new execution context for `this` the way regular functions do. Instead, they lexically inherit `this` from whatever scope they were *defined* in, permanently. Regular functions get `this` dynamically based on how they're *called* — as a method, a plain function, with `.call`/`.apply`/`.bind`, or with `new`. That's why arrow functions are great inside callbacks where you want to preserve the outer `this`, but bad as object methods or constructors."

**Q: When would an arrow function break a test helper?**
> "Whenever the test framework relies on binding its own `this` to the callback — like Mocha does inside `it()` or `beforeEach()` for things like `this.timeout()` or sharing test context. Since arrow functions ignore that binding and just inherit from the enclosing scope, calling `this.timeout(...)` inside an arrow-function test callback throws, because `this` isn't what the framework intended. The fix is to use a regular `function () {}` for the callback passed directly to the test runner."

**Q: What's the difference between `arguments` and rest parameters?**
> "`arguments` is an array-like object automatically available inside regular functions, but it's not a real array (no `.map`, `.reduce`, etc., without converting it first) and it doesn't exist inside arrow functions. Rest parameters (`...args`) give you a real array, work in both regular and arrow functions, and can be combined with named parameters to capture 'everything else' explicitly."

**Q: Can arrow functions be used as constructors?**
> "No — arrow functions don't have a `prototype` property and throw a `TypeError` if you try to call them with `new`. That's a direct consequence of not having their own `this`: constructors need to bind `this` to the newly created object, which arrow functions structurally can't do."

**Q: Why would you still use an IIFE today?**
> "Mostly for creating a private, self-contained scope for one-off logic — config initialization, or the classic module pattern for encapsulating private state before ES modules existed. With `let`/`const` block scoping and real ES modules now standard, IIFEs are far less necessary, but they still show up in some libraries and legacy code, and it's useful to recognize the pattern."

---

## 8. One-Page Cheat Sheet

- **Declarations** are fully hoisted (name + body) → callable before their line. **Expressions** are not → only the variable binding is hoisted (or TDZ'd for `let`/`const`).
- **Regular function `this`:** dynamic, set by the call-site (`obj.fn()`, `fn()`, `.call/.apply/.bind`, `new`).
- **Arrow function `this`:** lexical, permanently inherited from where it was *defined* — no own `this`, no own `arguments`, can't be used with `new`.
- **Rule of thumb:** use arrow functions for callbacks where you want to keep the outer `this` (`forEach`, `map`, `setTimeout`); use regular functions for object methods, constructors, and any test-runner callback that relies on a framework-bound `this` (e.g., Mocha's `this.timeout()`).
- **Default params** trigger only on `undefined`, not `null`/`0`/`""`. They can reference earlier parameters.
- **Rest params (`...args`)** are a real array, replace `arguments`, and work in arrow functions (unlike `arguments`).
- **IIFEs** create an instant private scope — the historical way to avoid global pollution and build the module pattern before ES modules/block scope existed.
- **Test helper rule:** pure utility functions (assertions, mock factories) can be arrow or regular — doesn't matter. But the callback passed *directly* to `it`/`describe`/`beforeEach` must be a regular `function` if the framework needs to bind its own `this` to it.
