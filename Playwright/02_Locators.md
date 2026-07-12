# Playwright Locators — Simple Notes

#### Docs:  https://playwright.dev/docs/locators#introduction

## 1. What is a Locator?

A **locator** is a way to find element(s) on a web page. It is the central piece of Playwright's **auto-waiting** and **retry** ability.

Think of it like an address that tells Playwright *where* an element is. The key point: the element is found *fresh* every time you use it, so if the page changes/re-renders, Playwright still finds the correct, up-to-date element.

**Why it matters:** This makes tests stable and reliable, even when the page updates dynamically.

```js
const locator = page.getByRole('button', { name: 'Sign in' });
await locator.hover();   // element found here
await locator.click();   // element found AGAIN here (fresh)
```

---

## 2. The Recommended Locators (Quick List)

Playwright gives you built-in locators. The order below is roughly the **priority order** — use the top ones first.

| Locator | Finds element by... |
|---|---|
| `getByRole()` | Role + accessible name (button, checkbox, heading, link…) |
| `getByText()` | Text content |
| `getByLabel()` | A form field's label text |
| `getByPlaceholder()` | Input's placeholder text |
| `getByAltText()` | Image's `alt` text |
| `getByTitle()` | Element's `title` attribute |
| `getByTestId()` | `data-testid` attribute |

**Golden rule:** Prefer locators close to *how a real user sees the page* (role, text, label). Avoid CSS/XPath when possible.

---

## 3. Each Locator — What, Why, When, How

### getByRole() — the #1 choice

- **What:** Finds elements by their role (button, checkbox, heading, link, list, table, etc.) as users and screen readers perceive them.
- **Why:** Closest to how real users and assistive tech experience the page → most reliable.
- **When:** For almost all interactive elements. Make it your default.

**HTML:**
```html
<h3>Sign up</h3>
<label>
  <input type="checkbox" /> Subscribe
</label>
<button>Submit</button>
```

**Code:**
```js
await expect(page.getByRole('heading', { name: 'Sign up' })).toBeVisible();
await page.getByRole('checkbox', { name: 'Subscribe' }).check();
await page.getByRole('button', { name: /submit/i }).click(); // regex, case-insensitive
```

Simple single-button example:
```html
<button>Sign in</button>
```
```js
await page.getByRole('button', { name: 'Sign in' }).click();
```

**Use case:** Clicking a "Sign in" button on a login page — works even if styling or DOM structure changes.

---

### getByLabel() — for form fields

- **What:** Finds a form control using its associated label text.
- **Why / When:** Best choice for filling out forms (inputs, selects).

**HTML:**
```html
<label>Password <input type="password" /></label>
```

**Code:**
```js
await page.getByLabel('Password').fill('secret');
```

**Use case:** Filling username and password on a login form.

---

### getByPlaceholder() — inputs without labels

- **What:** Finds an input using its placeholder text.
- **When:** The form field has **no label**, but has a placeholder hint.

**HTML:**
```html
<input type="email" placeholder="name@example.com" />
```

**Code:**
```js
await page.getByPlaceholder('name@example.com').fill('playwright@microsoft.com');
```

**Use case:** A search box or email field that only shows greyed-out hint text.

---

### getByText() — non-interactive elements

- **What:** Finds an element by the text it contains (substring, exact, or regex).
- **When:** For **non-interactive** elements like `div`, `span`, `p`. (For buttons/links use `getByRole`.)

**HTML:**
```html
<span>Welcome, John</span>
```

**Code:**
```js
await expect(page.getByText('Welcome, John')).toBeVisible();                   // substring
await expect(page.getByText('Welcome, John', { exact: true })).toBeVisible();  // exact
await expect(page.getByText(/welcome, [A-Za-z]+$/i)).toBeVisible();            // regex
```

**Note:** Whitespace is auto-normalized (extra spaces/line breaks become single spaces).

**Use case:** Checking a "Welcome, John" message appears after login.

---

### getByAltText() — images

- **What:** Finds an element (usually an image) by its `alt` text.
- **When:** Elements that support alt text, like `img` and `area`.

**HTML:**
```html
<img alt="playwright logo" src="/img/playwright-logo.svg" width="100" />
```

**Code:**
```js
await page.getByAltText('playwright logo').click();
```

**Use case:** Clicking a logo image or verifying an image is present.

---

### getByTitle() — elements with a title attribute

- **What:** Finds an element by its `title` attribute.
- **When:** The element has a `title` attribute.

**HTML:**
```html
<span title='Issues count'>25 issues</span>
```

**Code:**
```js
await expect(page.getByTitle('Issues count')).toHaveText('25 issues');
```

**Use case:** Checking a tooltip-style element like an issues counter.

---

### getByTestId() — most resilient, but not user-facing

- **What:** Finds an element by its `data-testid` attribute.
- **Why:** Most *resilient* — the test still passes even if text or role changes.
- **When:** When you follow a test-id methodology, OR when you can't use role/text.

**HTML:**
```html
<button data-testid="directions">Itinéraire</button>
```

**Code:**
```js
await page.getByTestId('directions').click();
```

**Customizing the attribute** (config file):
```js
export default defineConfig({
  use: { testIdAttribute: 'data-pw' }  // now uses data-pw instead of data-testid
});
```
Then your HTML uses the custom attribute:
```html
<button data-pw="directions">Itinéraire</button>
```
```js
await page.getByTestId('directions').click();
```

**Use case:** A button whose label is in a foreign language or changes often — tag it with a stable test id.

---

### getByCSS / getByXPath — last resort only

- **What:** Uses raw CSS or XPath selectors via `page.locator()`.
- **Why avoid:** These break easily when the DOM structure changes → unstable tests.
- **When:** Only if you *absolutely* must.

**HTML:**
```html
<button>Sign in</button>
```

**Code:**
```js
await page.locator('css=button').click();
await page.locator('xpath=//button').click();
await page.locator('button').click();     // auto-detected as CSS
await page.locator('//button').click();   // auto-detected as XPath
```

**Bad practice example (avoid this):**
```html
<div id="tsf">
  <div>
    <div class="A8SBwf">
      <div class="RNNXgb">
        <div><div class="a4bIc"><input /></div></div>
      </div>
    </div>
  </div>
</div>
```
```js
// ❌ Fragile — breaks the moment the DOM changes
await page.locator('#tsf > div:nth-child(2) > div.A8SBwf > div.RNNXgb > div > div.a4bIc > input').click();
```

---

## 4. Chaining & Narrowing Down

You can chain locators to search inside a specific part of the page.

**HTML:**
```html
<ul>
  <li>
    <h3>Product 1</h3>
    <button>Add to cart</button>
  </li>
  <li>
    <h3>Product 2</h3>
    <button>Add to cart</button>
  </li>
</ul>
```

**Code:**
```js
const product = page.getByRole('listitem').filter({ hasText: 'Product 2' });
await product.getByRole('button', { name: 'Add to cart' }).click();
```

Chain two locators (find "Save" inside a dialog):

**HTML:**
```html
<div data-testid="settings-dialog">
  <button>Cancel</button>
  <button>Save</button>
</div>
```
```js
const saveButton = page.getByRole('button', { name: 'Save' });
const dialog = page.getByTestId('settings-dialog');
await dialog.locator(saveButton).click();
```

---

## 5. Filtering Locators

Useful when many similar elements exist (e.g. product cards in a list).

**HTML (used for all filter examples below):**
```html
<ul>
  <li>
    <h3>Product 1</h3>
    <button>Add to cart</button>
  </li>
  <li>
    <h3>Product 2</h3>
    <button>Add to cart</button>
  </li>
</ul>
```

**Filter by text:**
```js
await page.getByRole('listitem')
  .filter({ hasText: 'Product 2' })
  .getByRole('button', { name: 'Add to cart' })
  .click();
```

**Filter by NOT having text:**
```js
await expect(page.getByRole('listitem').filter({ hasNotText: 'Out of stock' })).toHaveCount(5);
```

**Filter by a child/descendant:**
```js
await page.getByRole('listitem')
  .filter({ has: page.getByRole('heading', { name: 'Product 2' }) })
  .getByRole('button', { name: 'Add to cart' })
  .click();
```

**Filter by NOT having a child:**
```js
await expect(page.getByRole('listitem').filter({ hasNot: page.getByText('Product 2') })).toHaveCount(1);
```

> The `has`/`hasNot` locator must be **relative** to the outer locator (matched from inside it, not from the whole document).

---

## 6. Locator Operators

**`and()` — match BOTH conditions:**

**HTML:**
```html
<button title="Subscribe">Subscribe</button>
```
```js
const button = page.getByRole('button').and(page.getByTitle('Subscribe'));
```

**`or()` — match ONE of several (when unsure which appears):**

**HTML:**
```html
<!-- Sometimes this appears -->
<button>New</button>

<!-- ...other times this appears instead -->
<div>Confirm security settings <button>Dismiss</button></div>
```
```js
const newEmail = page.getByRole('button', { name: 'New' });
const dialog = page.getByText('Confirm security settings');

await expect(newEmail.or(dialog).first()).toBeVisible();
if (await dialog.isVisible())
  await page.getByRole('button', { name: 'Dismiss' }).click();
await newEmail.click();
```
**Use case:** Sometimes a security popup appears instead of the button — handle both.

**Match only visible elements:**

**HTML:**
```html
<button style="display: none">Invisible</button>
<button>Visible</button>
```
```js
await page.locator('button').filter({ visible: true }).click(); // clicks the visible one
```

---

## 7. Working with Lists

**HTML (used for the examples below):**
```html
<ul>
  <li>apple</li>
  <li>banana</li>
  <li>orange</li>
</ul>
```

**Count items:**
```js
await expect(page.getByRole('listitem')).toHaveCount(3);
```

**Assert all text in order:**
```js
await expect(page.getByRole('listitem')).toHaveText(['apple', 'banana', 'orange']);
```

**Get a specific item:**
```js
await page.getByText('orange').click();                                  // by text
await page.getByRole('listitem').filter({ hasText: 'orange' }).click();  // by filter
await page.getByRole('listitem').nth(1);                                 // by position (banana)
```

**Get by test id** (needs test ids in HTML):
```html
<ul>
  <li data-testid='apple'>apple</li>
  <li data-testid='banana'>banana</li>
  <li data-testid='orange'>orange</li>
</ul>
```
```js
await page.getByTestId('orange').click();
```

**Chaining filters** (row with "Mary" AND a "Say goodbye" button):

**HTML:**
```html
<ul>
  <li><div>John</div><div><button>Say hello</button></div></li>
  <li><div>Mary</div><div><button>Say hello</button></div></li>
  <li><div>John</div><div><button>Say goodbye</button></div></li>
  <li><div>Mary</div><div><button>Say goodbye</button></div></li>
</ul>
```
```js
const rowLocator = page.getByRole('listitem');
await rowLocator
  .filter({ hasText: 'Mary' })
  .filter({ has: page.getByRole('button', { name: 'Say goodbye' }) })
  .screenshot({ path: 'screenshot.png' });
```

**Loop through all items:**
```js
for (const row of await page.getByRole('listitem').all())
  console.log(await row.textContent());
```

---

## 8. Strictness (Important!)

Locators are **strict**: if an action would match **more than one element**, Playwright throws an error. This protects you from accidentally acting on the wrong element.

**HTML:**
```html
<button>Sign in</button>
<button>Sign up</button>
```
```js
await page.getByRole('button').click();  // ❌ ERROR — two buttons match
await page.getByRole('button').count();  //  OK — count works with many
```

**Escape hatch (use sparingly):** `first()`, `last()`, `nth()`.
```js
const banana = page.getByRole('listitem').nth(1);
```
> Not recommended — if the page changes, you may target the wrong element. Prefer a unique locator instead.

---

## 9. Shadow DOM

All locators work inside Shadow DOM **by default** — no extra work needed.

Exceptions: **XPath** does not pierce shadow roots, and **closed-mode** shadow roots are not supported.

**HTML:**
```html
<x-details role=button aria-expanded=true aria-controls=inner-details>
  <div>Title</div>
  #shadow-root
    <div id=inner-details>Details</div>
</x-details>
```
```js
await page.getByText('Details').click();                              // works inside shadow root
await page.locator('x-details', { hasText: 'Details' }).click();     // click the host element
await expect(page.locator('x-details')).toContainText('Details');
```

---

## Key Takeaways (Cheat Sheet)

1. **Prefer `getByRole()`** for most elements — it mirrors how users see the page.
2. **Use `getByLabel()`** for form fields, `getByPlaceholder()` if no label.
3. **Use `getByText()`** for non-interactive text (div/span/p).
4. **Use `getByTestId()`** when you need maximum stability or nothing else works.
5. **Avoid CSS/XPath** — they break when the DOM changes.
6. **Filter and chain** to pinpoint one element among many.
7. **Locators are strict** — matching multiple elements throws an error; fix it with a better locator, not `nth()`.

---

Every locator type now pairs the exact **HTML** with the matching Playwright **code**, just like your example. Want me to save this as a downloadable Markdown (`.md`) file, or extend it with the related **Actions** page (click, fill, check, etc.) to complete your study set?
