## Playwright + TypeScript Setup Guide

---

### Step 1: Create Folder & Install TypeScript

* **What:** Creating a dedicated project folder and installing TypeScript locally as a development dependency alongside Node.js type definitions.
* **Why:** Installing TypeScript at the project level (rather than globally) ensures consistency across different machines, CI/CD pipelines, and team members. `@types/node` provides autocompletion and type checking for core Node.js features.
* **How:**
Run these commands in your terminal:
```bash
mkdir ProjectName
cd ProjectName
npm init -y
npm install --save-dev typescript @types/node

```



---

### Step 2: Initialize the Repository with TypeScript

* **What:** Generating a default `tsconfig.json` file in the root of your project directory.
* **Why:** The `tsconfig.json` file acts as the command center for the TypeScript compiler. Without it, the environment will not know how to parse, check, or compile your TypeScript code.
* **How:**
Run the TypeScript compiler initialization script:
```bash
npx tsc --init

```



---

### Step 3: Configure TypeScript Compiler Options

* **What:** Explicitly adding `"types": ["node"]` to the `compilerOptions` array inside your `tsconfig.json`.
* **Why:** This tells TypeScript to automatically include ambient Node.js types globally across your project. It prevents the compiler from throwing errors when you use global Node objects like `process.env`.
* **How:**
Open `tsconfig.json`, locate `compilerOptions`, and add or update the `types` array:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "types": ["node"],
    "skipLibCheck": true
  }
}

```



---

### Step 4: Configure Package Type to Module

* **What:** Adding `"type": "module"` into your root `package.json` file.
* **Why:** This opts the project into native Node.js ECMAScript Modules (ESM). It allows you to use modern `import / export` syntax seamlessly instead of legacy CommonJS `require()` syntax.
* **How:**
Open `package.json` and insert the property at the top level:
```json
{
  "name": "playwright_templates",
  "version": "1.0.0",
  "type": "module",
  "dependencies": {},
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0"
  }
}

```



---

### Step 5: Initialize and Install Playwright

* **What:** Executing the automated Playwright scaffolding script to pull down framework binaries, configuration files, and testing browsers.
* **Why:** This CLI tool handles the heavy lifting—it sets up basic folder frameworks, installs required browser runtimes (Chromium, Firefox, WebKit), and builds a sample test file.
* **How:**
Execute the initializer and input the configuration prompts:
```bash
npm init playwright@latest

```


**Prompts Checklist:**
```text
Need to install the following packages: create-playwright@1.17.139
Ok to proceed? (y) y

Initializing project in '.'
√ Do you want to use TypeScript or JavaScript? · TypeScript
√ Where to put your end-to-end tests? · tests
√ Add a GitHub Actions workflow? (Y/n) · true
√ Install Playwright browsers? (Y/n) · true

```



---

### Step 6: Resolve `exactOptionalPropertyTypes` Code Conflicts

* **What:** Modifying `playwright.config.ts` to substitute explicit `undefined` values with explicit types (numbers or strings) or removing them entirely.
* **Why:** If your `tsconfig.json` has strict safety checking turned on via `"exactOptionalPropertyTypes": true`, TypeScript considers an explicit assignment of `undefined` to an optional field an error. It requires the property to either match the type (e.g., a number) or be entirely absent.
* **How:**
Open `playwright.config.ts` and modify properties like `workers` or `retries`:
```typescript
// ❌ INCORRECT (Causes compiler errors under strict setups)
workers: process.env.CI ? 1 : undefined,

//  CORRECT (Option A: Provide a logical fallback value)
workers: process.env.CI ? 1 : 4,

//  CORRECT (Option B: Omit the property declaration completely to let Playwright handle defaults)
// (Simply delete or comment out the line)

```
