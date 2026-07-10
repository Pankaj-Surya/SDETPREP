# Playwright: Auto-Waiting vs. Auto-Retrying

The main difference is **what they are waiting for**: Auto-waiting waits for an element to be **actionable** (visible, stable, enabled), while Auto-retrying waits for a specific **test condition to become true** (like a URL change or specific text appearing).

---

### Playwright Auto-Waiting

Auto-waiting applies to **actions** (like `.click()`, `.fill()`, `.press()`). Before performing the action, Playwright runs a series of actionability checks on the element.

* **Trigger**: Happens automatically on almost all interaction commands.
* **Checks performed**: Ensures the element is attached to the DOM, visible, stable (not animating), enabled, and not obscured by other elements.
* **How it works**: Playwright pauses the script execution until all checks pass, then performs the action. If the checks do not pass within the timeout, the action fails.

```javascript
// Playwright automatically waits for this button to be visible and enabled before clicking
await page.locator('#submit-btn').click(); 
```

---

### Playwright Auto-Retrying

Auto-retrying applies to **Web-First Assertions** (commands starting with `expect()`). 

* **Trigger**: Happens automatically when you use assertions that check a specific state.
* **Checks performed**: Evaluates if the element or page matches the exact condition you specified.
* **How it works**: If the assertion fails initially, Playwright does not crash. It immediately re-locates the element and runs the assertion again. It continuously loops and polls the live DOM until the condition passes or the timeout (default 5 seconds) is reached.

```javascript
// Playwright will poll the DOM repeatedly until the text changes to 'Saved'
await expect(page.locator('.status')).hasText('Saved'); 
```

---

### Summary Comparison

| Feature | Auto-Waiting | Auto-Retrying |
| :--- | :--- | :--- |
| **Applied To** | Actions (`click()`, `fill()`, `check()`) | Assertions (`expect().toBeVisible()`, `expect().hasText()`) |
| **Goal** | Ensures an element can be interacted with safely | Ensures the application reached the expected state |
| **Mechanism** | Pauses, runs actionability checks, executes action | Polls the DOM and evaluates the condition repeatedly |
| **Customization** | Can pass `{ force: true }` to bypass checks | Can pass custom timeouts per assertion |
