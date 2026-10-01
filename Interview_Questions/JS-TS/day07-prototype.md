# Day 7 — Prototypes & `this`

Everything you need to *understand it*, *explain it out loud*, and *answer follow-ups* in an interview.

---

## 1. The Prototype Chain

Every JavaScript object has an internal link to another object called its **prototype**. When you access a property on an object, JS first checks the object itself — if it's not found, it walks up the prototype chain, checking each linked object in turn, until it either finds the property or reaches `null` (the end of the chain).

```js
const animal = {
  eats: true,
  walk() {
    console.log("Animal walks");
  },
};

const rabbit = Object.create(animal); // rabbit's prototype is `animal`
rabbit.jumps = true;

console.log(rabbit.eats);   // true — not on rabbit itself, found via the prototype chain
rabbit.walk();               // "Animal walks" — method found on the prototype
console.log(rabbit.jumps);  // true — own property, found immediately

console.log(Object.getPrototypeOf(rabbit) === animal); // true
```

**Interview line:** "Every object has a hidden link to a prototype object. Property lookup isn't just 'does this object have it' — it's 'walk up the prototype chain until you find it or hit `null`.' This is how methods like `.map()` or `.toString()` are available on every array or object without being copied onto each instance individually — they live once on `Array.prototype` or `Object.prototype`, and every instance finds them via the chain."

### Functions, classes, and the prototype — how they connect

```js
function Animal(name) {
  this.name = name;
}

Animal.prototype.walk = function () {
  console.log(`${this.name} walks`);
};

const dog = new Animal("Rex");
dog.walk(); // "Rex walks" — found via dog's prototype chain, not an own property

console.log(dog.hasOwnProperty("walk")); // false — it's on the prototype, not the instance
console.log(Object.getPrototypeOf(dog) === Animal.prototype); // true
```

`class` syntax is mostly **syntactic sugar** over this exact same prototype mechanism:

```js
class AnimalClass {
  constructor(name) {
    this.name = name;
  }
  walk() {
    console.log(`${this.name} walks`);
  }
}

const cat = new AnimalClass("Whiskers");
console.log(Object.getPrototypeOf(cat) === AnimalClass.prototype); // true — same mechanism under the hood
```

**Interview line:** "`class` doesn't introduce a new inheritance model — it's syntactic sugar over prototypes. Methods defined in a class body still end up on `ClassName.prototype`, and `extends` just wires up the prototype chain between the child and parent automatically instead of you doing it manually."

---

## 2. `Object.create()` — Building the Prototype Chain Directly

```js
const vehiclePrototype = {
  startEngine() {
    console.log(`${this.type} engine starting...`);
  },
};

const car = Object.create(vehiclePrototype);
car.type = "Car";
car.startEngine(); // "Car engine starting..."

// Object.create(null) — an object with NO prototype at all, not even Object.prototype
const pureDict = Object.create(null);
pureDict.key = "value";
console.log(pureDict.toString); // undefined — no Object.prototype methods inherited!
```

**Interview line:** "`Object.create(proto)` creates a new object with `proto` explicitly set as its prototype — it's the most direct way to set up prototypal inheritance without a constructor function or `class` syntax at all. `Object.create(null)` is a handy trick for creating a truly 'bare' object with no inherited methods — useful as a safe dictionary/map where you don't want accidental collisions with things like a key literally named `toString`."

---

## 3. Prototypal Inheritance vs Classical Inheritance

| | Classical (Java, C++, etc.) | Prototypal (JavaScript) |
|---|---|---|
| What inherits | Classes inherit from classes (blueprints) | Objects inherit directly from other objects (live instances) |
| How it works | Instances are created from a class template, fixed at compile-time in most languages | Objects are linked to a prototype at creation, and the link can be changed dynamically at runtime |
| Relationship | "is-a" via a rigid, pre-defined class hierarchy | "delegates to" — an object asks its prototype for something it doesn't have |
| Flexibility | Class structure is fixed once defined | You can reassign an object's prototype at runtime, or mix and match behaviors dynamically |

**Interview line:** "In classical inheritance, a class is a blueprint, and objects are instances stamped out from that blueprint — the relationship is fixed at the class level. In JavaScript's prototypal model, objects inherit directly from *other objects*, not from an abstract class definition — a rabbit object literally points to an animal object and delegates to it for anything it doesn't have itself. `class` syntax in modern JS makes this look classical on the surface, but underneath, it's still objects linking to other objects via the prototype chain — there's no real 'class' at runtime, just functions and prototype objects."

---

## 4. `this` in Different Contexts — the Full Picture

`this` is **dynamic** for regular functions — its value depends entirely on *how the function was called*, not where it was defined (contrast with arrow functions, which are lexical — see Day 3).

```js
// 1. Global context (non-strict mode)
console.log(this); // global object (window in browsers, {} in Node modules)

// 2. As an object method — `this` is the object before the dot
const obj = {
  name: "Alice",
  greet() {
    console.log(this.name);
  },
};
obj.greet(); // "Alice" — `this` is `obj`

// 3. As a plain function call — `this` is undefined (strict mode) or global object (non-strict)
function standalone() {
  console.log(this);
}
standalone(); // undefined in strict mode / modules

// 4. Extracted from an object — LOSES its `this` binding!
const greetFn = obj.greet;
greetFn(); // ❌ TypeError or undefined — `this` is no longer `obj`

// 5. As a constructor with `new` — `this` is the newly created object
function Person(name) {
  this.name = name;
}
const p = new Person("Bob");
console.log(p.name); // "Bob"

// 6. Explicitly set with call/apply/bind — see section 5
```

**The #4 case is the one that trips people up most in real code** — passing a method as a callback (e.g., `setTimeout(obj.greet, 100)` or `array.map(obj.someMethod)`) silently detaches it from `obj`, and `this` breaks.

---

## 5. `call`, `apply`, and `bind` — Explicitly Controlling `this`

All three let you explicitly set what `this` refers to inside a function — the difference is *when* the function runs and *how* arguments are passed.

```js
function introduce(greeting, punctuation) {
  console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const person = { name: "Alice" };

// call — invokes immediately, arguments passed individually
introduce.call(person, "Hi", "!"); // "Hi, I'm Alice!"

// apply — invokes immediately, arguments passed as an array
introduce.apply(person, ["Hi", "!"]); // "Hi, I'm Alice!"

// bind — does NOT invoke immediately; returns a NEW function with `this` permanently locked in
const boundIntroduce = introduce.bind(person, "Hi");
boundIntroduce("!"); // "Hi, I'm Alice!" — can call it now, or store it, or pass it as a callback
```

**Interview line:** "`call` and `apply` both invoke the function right away with a given `this` — the only difference is argument style: `call` takes them individually, `apply` takes them as an array (mostly irrelevant since spread syntax, but still asked about). `bind` is different in kind, not just style — it doesn't call the function at all; it returns a brand-new function with `this` (and optionally some leading arguments) permanently baked in, which you can invoke later, pass as a callback, or store for repeated use."

### When would you use `bind`? (direct interview question)

**The canonical use case: fixing a detached method before passing it as a callback.**

```js
class Counter {
  constructor() {
    this.count = 0;
  }
  increment() {
    this.count++;
    console.log(this.count);
  }
}

const counter = new Counter();

// ❌ Broken — extracting the method as a callback loses `this`
document.getElementById("btn").addEventListener("click", counter.increment);
// clicking throws: Cannot read properties of undefined (reading 'count')

// ✅ Fixed with bind — `this` is permanently locked to `counter`
document.getElementById("btn").addEventListener("click", counter.increment.bind(counter));
```

**Interview line:** "I'd reach for `bind` any time I need to pass an object's method somewhere as a plain function reference — an event listener, a callback into another library, `setTimeout` — where it will be *called* without the object context still attached, and I want `this` to keep pointing back to the original object regardless of how it ends up being invoked."

---

## 6. Test Tie-In: `this` Inside Jest/Mocha/Playwright Hooks — `function(){}` vs Arrow Functions

This is a direct, practical consequence of everything above, and it's exactly where this topic shows up in day-to-day test automation work.

### Mocha: relies on binding a custom `this` to test callbacks

```js
describe("User API", function () {
  beforeEach(function () {
    this.timeout(5000);          // ✅ works — Mocha binds `this` to a test-context object
    this.testUser = { id: 1 };   // ✅ can attach shared data for this test run
  });

  it("fetches the user", function () {
    console.log(this.testUser); // ✅ { id: 1 } — same `this` context as beforeEach
  });
});
```

```js
// ❌ BROKEN — arrow functions ignore Mocha's `this` binding entirely
describe("User API", () => {
  beforeEach(() => {
    this.timeout(5000);   // ❌ TypeError — `this` here is NOT Mocha's context,
    this.testUser = {};   //    it's inherited from the surrounding module scope
  });

  it("fetches the user", () => {
    console.log(this.testUser); // ❌ undefined — never actually set
  });
});
```

**Why:** Mocha internally calls your hook/test function using `.call(testContextObject)` so that a regular `function(){}` receives Mocha's special `this`. But arrow functions *ignore* however they're called — they only ever look up `this` lexically from where they were written, which in this case is just the outer module scope, not Mocha's context. This is the exact mechanism from section 4 (case #4/#6 above) playing out directly inside a test framework.

### Playwright / Jest: less `this`-reliant, but the same principle applies to fixtures/context

```js
// Playwright test — page/context are passed as explicit arguments, NOT via `this`,
// so arrow functions are perfectly safe here (Playwright doesn't rely on `this` binding at all)
test("login works", async ({ page }) => {
  await page.goto("https://example.com/login");
  // arrow function is fine — Playwright never tried to bind a custom `this` in the first place
});
```

```js
// Jest — similarly doesn't bind a custom `this` to test callbacks,
// so arrow functions are the conventional/idiomatic choice
describe("Cart", () => {
  let cart;

  beforeEach(() => {
    cart = createCart(); // shared state via closure over an outer `let`, not via `this`
  });

  it("starts empty", () => {
    expect(cart.items).toHaveLength(0);
  });
});
```

**Interview line:** "The rule isn't 'always avoid arrow functions in tests' — it's 'know whether the framework relies on binding its own `this` to your callback.' Mocha does, for things like `this.timeout()` and sharing test-scoped data across hooks, so you must use `function(){}` there. Jest and Playwright generally don't rely on `this` at all — Jest shares state through closures over outer variables, and Playwright passes fixtures like `page` as explicit function arguments — so arrow functions are perfectly idiomatic and arguably preferred in those frameworks. Knowing *why* the distinction exists, rather than just memorizing 'use regular functions in tests,' is what shows real understanding here."

---

## 7. Interview Q&A Script

**Q: How does prototypal inheritance differ from classical inheritance?**
> "In classical inheritance, classes are abstract blueprints, and objects are instances created from them, with the hierarchy fixed at the class level. In JavaScript's prototypal model, objects inherit directly from other live objects — there's no abstract class at runtime, just objects with a link to a prototype object they delegate to for anything they don't have themselves. `class` syntax is sugar over this same mechanism — it doesn't change the underlying model, just the syntax for setting it up."

**Q: What does `bind` do and when would you use it?**
> "`bind` returns a new function with `this` (and optionally some arguments) permanently locked to whatever you pass in, without invoking the original function immediately — unlike `call`/`apply`, which invoke right away. I use it whenever I need to pass an object's method somewhere as a plain callback — an event listener, `setTimeout`, or into a library — where it will be called without the object still attached, and I need `this` to keep referring to the original object no matter how it's eventually invoked."

**Q: What's the difference between `call` and `apply`?**
> "Both invoke the function immediately with an explicit `this`. `call` takes the function's arguments individually; `apply` takes them as a single array. Functionally interchangeable given spread syntax today, but `apply` used to be the only option when you had arguments already in array form."

**Q: Why does `this` break when you pass an object's method as a callback?**
> "Because `this` in a regular function is determined by the call-site, not by where the method is defined. When you extract `obj.method` and pass just the function reference somewhere else — a callback, an event listener — it gets called later as a plain function call, with no object before the dot, so `this` is `undefined` (strict mode) or the global object. `bind` fixes this by permanently attaching the correct `this` regardless of how it's later called."

**Q: Why would Mocha require a regular function in `beforeEach`, but Jest doesn't care?**
> "Mocha internally invokes your hook using something like `.call(testContext)`, intentionally binding a custom `this` object so you can call things like `this.timeout()` or share data across hooks in the same test. Regular functions respect that binding; arrow functions ignore it entirely and just inherit `this` from the surrounding scope, so `this.timeout` doesn't exist and throws. Jest doesn't rely on `this` binding at all — it shares state through closures — so arrow functions work fine and are the idiomatic choice there."

---

## 8. One-Page Cheat Sheet

- **Prototype chain:** every object links to a prototype; property lookup walks up the chain until found or `null`. `class` is sugar over this — methods still live on `ClassName.prototype`.
- **`Object.create(proto)`** sets up a prototype link directly, no constructor needed. `Object.create(null)` makes a prototype-less object — no inherited methods at all, useful as a safe dictionary.
- **Prototypal vs classical:** objects inherit from *objects* (delegation, runtime-flexible) vs classes inheriting from classes (fixed blueprint hierarchy).
- **`this` is dynamic** in regular functions — set by the call-site: object method (`obj.fn()` → the object), plain call (→ `undefined`/global), `new` (→ the new instance), or explicitly via `call`/`apply`/`bind`. Extracting a method as a bare reference loses its `this`.
- **`call`/`apply`** invoke immediately with an explicit `this` (individual args vs array args). **`bind`** returns a new function with `this` permanently locked, without invoking it — the fix for passing methods as detached callbacks.
- **Testing rule:** use regular `function(){}` in Mocha's `it`/`beforeEach`/etc. whenever you need `this.timeout()` or shared `this`-based test context, since Mocha explicitly binds its own `this` and arrow functions would ignore it. Jest and Playwright don't rely on `this` binding (Jest uses closures, Playwright passes fixtures as arguments), so arrow functions are safe and idiomatic there.
