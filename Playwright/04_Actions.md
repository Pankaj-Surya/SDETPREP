# Playwright — Actions & Interactions — Notes with Real-Time Examples

Official references: https://playwright.dev/docs/input · https://playwright.dev/docs/api/class-locator · https://playwright.dev/docs/api/class-keyboard · https://playwright.dev/docs/api/class-mouse · https://playwright.dev/docs/actionability

Simple language, real test-automation scenarios, all in TypeScript.

---

## Introduction

Playwright interacts with the page the same way a real user would — text inputs, checkboxes, radio buttons, selects, mouse clicks, typing, keyboard shortcuts, file uploads, focus. ([Introduction](https://playwright.dev/docs/input#introduction))

Before almost every action, Playwright automatically: waits for the element to be attached to the DOM, waits for it to be visible and stable (no more animating), scrolls it into view, and waits until it can actually receive the action (nothing else covering it). This is why Playwright tests are far less flaky than raw Selenium clicks.

---

## 1. `click`, `dblclick`, `hover`, `focus`, `blur`

**Reference:** [Mouse click](https://playwright.dev/docs/input#mouse-click), [Focus element](https://playwright.dev/docs/input#focus-element)

**`click`** — a normal click, the most common action in any test.
```ts
await page.getByRole('button', { name: 'Add to Cart' }).click();
```

**Real-time example:** Clicking "Place Order" on a checkout page and confirming the order confirmation shows up.
```ts
await page.getByRole('button', { name: 'Place Order' }).click();
await expect(page.getByText('Order Confirmed')).toBeVisible();
```

**`dblclick`** — double click, common for things like renaming a file in a file manager UI, or expanding a table row.
```ts
await page.getByText('report.pdf').dblclick();
await expect(page.getByText('report.pdf - Preview')).toBeVisible();
```

**`hover`** — moving the mouse over an element without clicking, common for testing tooltips or dropdown menus that open on mouse-over.
```ts
await page.getByText('Help').hover();
await expect(page.getByRole('tooltip')).toHaveText('Need assistance? Click here');
```

**`focus`** — moving keyboard focus to an element programmatically (without clicking), useful for testing focus-triggered behavior like validation messages appearing.
```ts
await page.getByLabel('Email').focus();
```

**`blur`** — Playwright doesn't have a dedicated `.blur()` locator method; the common real pattern is to focus a *different* element (or press Tab) to trigger the original field's blur validation.
```ts
await page.getByLabel('Email').fill('not-an-email');
await page.getByLabel('Password').focus(); // moves focus away, triggering blur on Email
await expect(page.getByText('Enter a valid email')).toBeVisible();
```

Some clicks also support extra options that are worth knowing:
```ts
await page.getByText('Item').click({ button: 'right' });               // right click
await page.getByText('Item').click({ modifiers: ['Shift'] });          // shift+click
await page.getByText('Item').click({ position: { x: 0, y: 0 } });      // click a specific spot
await page.getByRole('button').click({ force: true });                 // bypass actionability checks
```

---

## 2. `fill`, `type`, `press`, `clear`

**Reference:** [Text input](https://playwright.dev/docs/input#text-input), [Type characters](https://playwright.dev/docs/input#type-characters), [Keys and shortcuts](https://playwright.dev/docs/input#keys-and-shortcuts)

**`fill`** — the recommended way to enter text; sets the value directly and fires an `input` event. Fast and reliable — use this by default.
```ts
await page.getByLabel('Full Name').fill('Asha Rao');
await page.getByLabel('Birth date').fill('2020-02-02'); // works for date inputs too
```

**Real-time example:** Filling out a signup form.
```ts
await page.getByLabel('Email').fill('asha@example.com');
await page.getByLabel('Password').fill('Secret123!');
await page.getByRole('button', { name: 'Sign Up' }).click();
```

**`type`** (`locator.pressSequentially()` in modern Playwright) — types character by character with real `keydown`/`keyup` events. Only needed when the page has custom keyboard logic — e.g., a live search box that filters results as you type, or a masked input.
```ts
// Fires keydown/keyup for every letter, so a "live filter as you type" search box actually reacts
await page.getByPlaceholder('Search products').pressSequentially('laptop', { delay: 100 });
await expect(page.getByText('Showing results for "laptop"')).toBeVisible();
```

**`press`** — a single keystroke (or shortcut) on a focused element.
```ts
await page.getByPlaceholder('Search').press('Enter');            // submit a search
await page.getByRole('textbox').press('Control+ArrowRight');     // jump a word
```

**Real-time example:** Submitting a comment box with Enter, instead of clicking a separate button.
```ts
await page.getByPlaceholder('Write a comment...').fill('Great article!');
await page.getByPlaceholder('Write a comment...').press('Enter');
await expect(page.getByText('Great article!')).toBeVisible();
```

**`clear`** — empties a text field, useful before re-entering different data in the same test.
```ts
await page.getByLabel('Promo Code').fill('SAVE10');
await page.getByLabel('Promo Code').clear();
await page.getByLabel('Promo Code').fill('SAVE20');
```

---

## 3. `check`, `uncheck`, `selectOption`

**Reference:** [Checkboxes and radio buttons](https://playwright.dev/docs/input#checkboxes-and-radio-buttons), [Select options](https://playwright.dev/docs/input#select-options)

**`check` / `uncheck`** — works on `input[type=checkbox]`, `input[type=radio]`, and `[role=checkbox]`. Unlike `.click()`, these are idempotent — calling `.check()` on an already-checked box does nothing (no accidental double-toggle).
```ts
await page.getByLabel('I agree to the terms above').check();
await page.getByLabel('Subscribe to newsletter').uncheck();
await page.getByLabel('XL').check(); // selecting a radio button
```

**Real-time example:** An e-commerce size selector (radio buttons) and a "Gift wrap this item" checkbox.
```ts
await page.getByLabel('Size: Large').check();
await page.getByLabel('Gift wrap this item').check();
await expect(page.getByText('+$2.00 for gift wrap')).toBeVisible();
```

**`selectOption`** — for native `<select>` dropdowns.
```ts
await page.getByLabel('Choose a color').selectOption('blue');             // by value
await page.getByLabel('Choose a color').selectOption({ label: 'Blue' });  // by visible label
await page.getByLabel('Choose multiple colors').selectOption(['red', 'green']); // multi-select
```

**Real-time example:** Choosing a country in a shipping form.
```ts
await page.getByLabel('Country').selectOption({ label: 'India' });
await expect(page.getByLabel('State')).toBeEnabled(); // state dropdown unlocks after country chosen
```

---

## 4. `dragAndDrop`

**Reference:** [Drag and Drop](https://playwright.dev/docs/input#drag-and-drop), [Dragging manually](https://playwright.dev/docs/input#dragging-manually)

**High-level way** — `locator.dragTo()` handles hover → mouse down → move → mouse up automatically.
```ts
await page.locator('#task-card-1').dragTo(page.locator('#in-progress-column'));
```

**Real-time example:** A Kanban/Trello-style board — dragging a task card from "To Do" into "In Progress".
```ts
await page.getByText('Fix login bug').dragTo(page.locator('[data-column="in-progress"]'));
await expect(page.locator('[data-column="in-progress"]').getByText('Fix login bug')).toBeVisible();
```

**Manual/low-level way** — needed when the app is picky about the exact sequence of mouse events (common with drag libraries that listen for `dragover`).
```ts
await page.locator('#task-card-1').hover();
await page.mouse.down();
await page.locator('[data-column="in-progress"]').hover();
await page.locator('[data-column="in-progress"]').hover(); // second hover, some apps need this to fire dragover
await page.mouse.up();
```

---

## 5. `tap` (mobile)

**In simple words:** Simulates a real touchscreen tap instead of a mouse click — only works when the browser context has `hasTouch: true` (automatically true for built-in mobile device profiles).

**Real-time example:** Testing a mobile web shop's "Add to Cart" button on an emulated iPhone.
```ts
import { test, expect, devices } from '@playwright/test';

test.use({ ...devices['iPhone 13'] }); // includes hasTouch: true automatically

test('tap adds item to cart on mobile', async ({ page }) => {
  await page.goto('https://shop.example.com/product/101');
  await page.getByRole('button', { name: 'Add to Cart' }).tap();
  await expect(page.getByText('1 item in cart')).toBeVisible();
});
```

> If you call `.tap()` on a context without `hasTouch: true`, Playwright throws an error telling you to enable it.

---

## 6. Keyboard (`page.keyboard` — Keyboard API)

**Reference:** https://playwright.dev/docs/api/class-keyboard

**In simple words:** Lower-level than `locator.press()` — lets you hold keys down, release them, or type raw text, without targeting a specific locator. Useful for global shortcuts or multi-key sequences.

**Real-time example 1 — a "select all + delete" shortcut test in a rich text editor:**
```ts
await page.getByRole('textbox').fill('Hello World!');
await page.keyboard.press('Control+A');
await page.keyboard.press('Backspace');
await expect(page.getByRole('textbox')).toHaveText('');
```

**Real-time example 2 — holding Shift while pressing arrow keys to select part of the text (fine-grained control with `down`/`up`):**
```ts
await page.keyboard.type('Hello World!');
await page.keyboard.press('Home');
await page.keyboard.down('Shift');
for (let i = 0; i < 5; i++) await page.keyboard.press('ArrowRight');
await page.keyboard.up('Shift');
await page.keyboard.press('Backspace'); // deletes "Hello"
```

**Real-time example 3 — navigating a custom dropdown menu with arrow keys (common accessibility test):**
```ts
await page.getByRole('button', { name: 'Sort by' }).click();
await page.keyboard.press('ArrowDown');
await page.keyboard.press('ArrowDown');
await page.keyboard.press('Enter');
await expect(page.getByText('Sorted by: Price (High to Low)')).toBeVisible();
```

---

## 7. Mouse (`page.mouse` — Mouse API)

**Reference:** https://playwright.dev/docs/api/class-mouse

**In simple words:** Raw, pixel-coordinate-based mouse control — for cases `locator.click()` can't express, like drawing, canvas interactions, or custom sliders.

**Real-time example 1 — testing a signature pad / drawing canvas:**
```ts
const canvasBox = await page.locator('#signature-canvas').boundingBox();
await page.mouse.move(canvasBox!.x + 10, canvasBox!.y + 10);
await page.mouse.down();
await page.mouse.move(canvasBox!.x + 100, canvasBox!.y + 80, { steps: 10 });
await page.mouse.up();
await expect(page.getByText('Signature captured')).toBeVisible();
```

**Real-time example 2 — dragging a custom range slider handle by exact pixels (when it's not a native `<input type=range>`):**
```ts
const handle = page.locator('.price-slider-handle');
const box = await handle.boundingBox();
await page.mouse.move(box!.x + box!.width / 2, box!.y + box!.height / 2);
await page.mouse.down();
await page.mouse.move(box!.x + 150, box!.y, { steps: 5 }); // drag right by 150px
await page.mouse.up();
```

**Real-time example 3 — scroll-wheel driven interaction (see also Scrolling below):**
```ts
await page.mouse.wheel(0, 500); // scroll down 500px
```

---

## 8. Scroll (`scrollIntoViewIfNeeded`, `wheel`)

**Reference:** [Scrolling](https://playwright.dev/docs/input#scrolling)

**In simple words:** Playwright auto-scrolls elements into view before acting on them — you rarely need to scroll manually. The exceptions: infinite-scroll lists, or precise screenshot positioning.

**Real-time example 1 — forcing an "infinite scroll" product list to load more items:**
```ts
await page.getByText('You reached the end').scrollIntoViewIfNeeded();
await expect(page.locator('.product-card')).toHaveCount(40); // more items loaded after scroll
```

**Real-time example 2 — scrolling inside a specific scrollable container (e.g., a chat window) using the mouse wheel:**
```ts
await page.getByTestId('chat-messages').hover();
await page.mouse.wheel(0, -1000); // scroll up to see older messages
await expect(page.getByText('Conversation started')).toBeVisible();
```

**Real-time example 3 — programmatic scroll via `evaluate` (when wheel isn't reliable, e.g., headless CI quirks):**
```ts
await page.getByTestId('scrolling-container').evaluate((el) => (el.scrollTop += 300));
```

---

## 9. File upload (`setInputFiles`)

**Reference:** [Upload files](https://playwright.dev/docs/input#upload-files)

**In simple words:** Attaches files directly to a hidden `<input type="file">`, skipping the OS file picker dialog entirely (which Playwright can't interact with anyway).

**Real-time example 1 — uploading a resume on a job application form:**
```ts
await page.getByLabel('Upload Resume').setInputFiles(path.join(__dirname, 'resume.pdf'));
await expect(page.getByText('resume.pdf uploaded')).toBeVisible();
```

**Real-time example 2 — uploading multiple images to a product listing:**
```ts
await page.getByLabel('Upload photos').setInputFiles([
  path.join(__dirname, 'photo1.jpg'),
  path.join(__dirname, 'photo2.jpg'),
]);
await expect(page.locator('.photo-thumbnail')).toHaveCount(2);
```

**Real-time example 3 — the file input is created dynamically only after a button click (no `<input>` to target upfront) — use the `filechooser` event:**
```ts
const fileChooserPromise = page.waitForEvent('filechooser');
await page.getByRole('button', { name: 'Attach File' }).click();
const fileChooser = await fileChooserPromise;
await fileChooser.setFiles(path.join(__dirname, 'invoice.pdf'));
```

**Bonus — uploading an in-memory file without needing a real file on disk (e.g., generated CSV):**
```ts
await page.getByLabel('Upload file').setInputFiles({
  name: 'data.csv',
  mimeType: 'text/csv',
  buffer: Buffer.from('id,name\n1,Asha\n2,Ravi'),
});
```

---

## Handling custom components (MUI, Bootstrap, Tailwind + Headless UI, etc.)

**The core problem:** Component libraries often don't use plain HTML `<select>`, `<input type=checkbox>`, etc. — they build custom-looking widgets out of `<div>`s and JavaScript. Native Playwright actions (`selectOption`, `check`) only work on real HTML form elements, so custom components need a different approach.

**The golden rule: locate by ARIA role, not by CSS class.** Component libraries like MUI implement proper ARIA roles (`role="combobox"`, `role="option"`, `role="dialog"`) specifically so assistive tech (and tools like Playwright) can interact with them predictably — use `getByRole()` / `getByLabel()` instead of digging into class names.

### Material UI (MUI) `Select` — not a native `<select>`

MUI's `Select` renders a clickable `div[role="button"]` (combobox) that opens a floating `listbox` with `li[role="option"]` items — `selectOption()` won't work on it.

```ts
// Open the dropdown, then pick the option from the listbox that appears
await page.getByLabel('Country').click(); // MUI Select acts like a button
await page.getByRole('option', { name: 'India' }).click();
```

### MUI `Autocomplete`

Type into the input, wait for the filtered options to render, then click one.

```ts
await page.getByLabel('Assign to').fill('Asha');
await page.getByRole('option', { name: 'Asha Rao' }).click();
```

### MUI `Checkbox` / `Switch`

These usually render a real (but visually hidden, `opacity: 0`) `<input type="checkbox">` under custom styling. `check()`/`uncheck()` normally still work because Playwright targets the real input via its associated label — but if actionability checks fail due to the overlay, force it:

```ts
await page.getByLabel('Enable notifications').check(); // usually just works
// if it's blocked by a styled overlay element:
await page.getByLabel('Enable notifications').check({ force: true });
```

### MUI `Slider`

Dragging a slider handle pixel-by-pixel is fragile across screen sizes. **Prefer keyboard**, which MUI sliders fully support:

```ts
const slider = page.getByRole('slider', { name: 'Price range' });
await slider.focus();
for (let i = 0; i < 10; i++) await page.keyboard.press('ArrowRight'); // increments by 10 steps
```

### MUI `Dialog` / Bootstrap `Modal` — rendered in a portal

Both libraries render modals **outside** their normal place in the DOM (appended near `<body>` via a React Portal, or moved by JS). This doesn't change how you locate them — `page.getByRole('dialog')` still finds it — just don't scope your locator to a parent container that visually "contains" the modal, since in the DOM it doesn't.

```ts
await page.getByRole('button', { name: 'Delete Account' }).click();
await expect(page.getByRole('dialog')).toBeVisible();
await page.getByRole('dialog').getByRole('button', { name: 'Confirm' }).click();
```

### Bootstrap dropdowns / toggles

Bootstrap dropdowns toggle an `aria-expanded` attribute and reveal a `.dropdown-menu` — again, role-based locators work well since Bootstrap markup is largely semantic HTML with Bootstrap's own ARIA attributes.

```ts
await page.getByRole('button', { name: 'Actions' }).click();
await expect(page.getByRole('button', { name: 'Actions' })).toHaveAttribute('aria-expanded', 'true');
await page.getByRole('menuitem', { name: 'Export CSV' }).click();
```

### Tailwind CSS

Tailwind is a **utility CSS framework, not a component/widget library** — it has no JS behavior of its own, so there's no special "Tailwind interaction pattern." The real thing to watch for: Tailwind projects are often paired with a **headless component library** (Radix UI, Headless UI, Ariakit) that supplies the actual dropdown/dialog/combobox *behavior* and ARIA roles. Since those are built specifically to be accessible, the same role-based approach applies:

```ts
// A Tailwind + Headless UI combobox — same pattern as MUI Autocomplete
await page.getByRole('combobox', { name: 'Choose a project' }).click();
await page.getByRole('option', { name: 'Website Redesign' }).click();
```

### General checklist for any custom component (MUI, Bootstrap, Tailwind/Radix, or a totally custom in-house design system)

1. Try `getByRole()` first — most well-built component libraries expose the right ARIA role.
2. If no meaningful role/label exists, use `getByTestId()` — ask developers to add a `data-testid` rather than reaching for a generated/hashed CSS class.
3. **Never** target CSS-in-JS generated class names (e.g., MUI's `.css-1a2b3c-MuiButton-root`) — they change between builds and aren't stable.
4. For anything that opens a floating panel (dropdown, autocomplete, tooltip, popover) — click to open, then assert the panel is visible **before** interacting with its contents, since it usually renders asynchronously.
5. For drag-based custom widgets (sliders, custom drag handles) — prefer keyboard interaction over pixel-dragging where the component supports it; it's far less flaky.

---

## Quick summary

| Action | One-line real-world use |
|---|---|
| `click` / `dblclick` / `hover` / `focus` | Basic interactions — buttons, tooltips, focus-triggered validation |
| `fill` / `type` / `press` / `clear` | Enter text; `type` only for special keyboard-driven UIs |
| `check` / `uncheck` / `selectOption` | Checkboxes, radios, native `<select>` dropdowns |
| `dragAndDrop` | Kanban boards, reordering lists |
| `tap` | Mobile-emulated taps (needs `hasTouch: true`) |
| Keyboard API | Shortcuts, held modifier keys, arrow-key navigation |
| Mouse API | Pixel-precise drag, canvas drawing, custom sliders |
| Scroll | Infinite lists, precise screenshot positioning |
| `setInputFiles` | File uploads, including dynamic inputs and in-memory files |
| Custom components (MUI/Bootstrap/Tailwind) | Use `getByRole()`/`getByLabel()`; never target generated CSS classes |
