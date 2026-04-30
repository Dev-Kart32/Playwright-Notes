---
layout: default
title: My Site
---

# Playwright + TypeScript Enterprise Framework — Complete Tutorial
### Your codebase. Explained line by line. By Bob, for ktx.

---

## Table of Contents

- [Part 0 — Project Setup from Scratch](#part-0)
- [Part 1 — Project Overview](#part-1)
- [Part 2 — TypeScript Foundations](#part-2)
- [Part 3 — Playwright Core Concepts](#part-3)
- [Part 4 — Enterprise Patterns Deep Dive](#part-4)
- [Part 5 — Config File Complete Walkthrough](#part-5)
- [Part 6 — Test File Anatomy](#part-6)
- [Part 7 — Page Object Model Deep Dive](#part-7)
- [Part 8 — Data & Configuration Management](#part-8)
- [Part 9 — Running & Reporting](#part-9)
- [Part 10 — Beginner Mental Models](#part-10)

---

# PART 0 — Project Setup from Scratch {#part-0}

> Bob says: "This section walks you through standing up this exact project from a brand-new machine. Every step has a WHY — not just a what."

---

## Step 1 — Install Node.js

**What:** Node.js is a JavaScript/TypeScript runtime — it lets you execute code outside a browser.

**Why this step exists:** Playwright is a Node.js package. Without Node.js, you cannot run npm, install packages, or execute TypeScript. Everything in this project depends on it.

**On macOS (M1):**
```bash
# Option A: Official installer from nodejs.org — download the macOS ARM64 LTS version
# Option B: via Homebrew (recommended for dev machines)
brew install node

# Verify
node --version    # should be >= 16.x
npm --version     # comes bundled with Node
```

**On Windows (ThinkPad):**
Download the LTS `.msi` installer from [nodejs.org](https://nodejs.org). Run it, accept defaults. Node and npm are both installed.

**Why LTS?** Long-Term Support versions are stable and supported for ~3 years. Avoid "current" versions for automation work — library compatibility matters.

---

## Step 2 — Install VS Code

**What:** Your code editor.

**Why VS Code specifically:** TypeScript IntelliSense, Playwright's official VS Code extension for running/debugging tests directly in the editor, and ESLint plugin support are all first-class here. The Playwright extension even lets you click a play button next to each test.

Download from [code.visualstudio.com](https://code.visualstudio.com).

**Recommended Extensions to install:**
- `ms-playwright.playwright` — run/debug Playwright tests from the editor
- `dbaeumer.vscode-eslint` — shows lint errors inline
- `esbenp.prettier-vscode` — optional formatter

---

## Step 3 — Create the Project Folder

```bash
mkdir enterprise-playwright-framework
cd enterprise-playwright-framework
```

**Why name it this?** This is your project root. Everything you build lives here. The name is the `"name"` field in `package.json`.

---

## Step 4 — Initialize package.json

```bash
npm init -y
```

**What this does:** Creates `package.json`, which is your project's manifest. It records your project name, version, scripts, and — most importantly — all your dependencies.

**Why `-y`?** Skips interactive prompts and uses defaults. You can edit the file manually after.

**Why package.json matters in enterprise:** If you ever hand this project to a colleague or a CI server, they run `npm install` and get the exact same environment. Without `package.json`, there's no reproducibility.

---

## Step 5 — Initialize TypeScript

```bash
npm install --save-dev typescript @types/node
```

**What:** Installs TypeScript compiler and Node.js type definitions.

**Why `--save-dev`?** TypeScript is a development tool — it compiles your code but it's not needed at runtime. `--save-dev` puts it in `devDependencies` in `package.json`, which keeps production artifacts lean.

**Why `@types/node`?** TypeScript needs type definitions for Node.js built-in modules like `fs`, `path`, and `process`. This package provides those definitions. Without it, `import path from 'path'` would fail with a type error.

Now create the `tsconfig.json` file:

```json
{
  "compilerOptions": {
    "target": "es2018",
    "module": "commonjs",
    "strict": true,
    "resolveJsonModule": true,
    "esModuleInterop": true
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules"]
}
```

**Why each option:**
- `"target": "es2018"` — compile down to ES2018 JavaScript (supports async/await natively)
- `"module": "commonjs"` — use Node's module system (`require`/`exports`)
- `"strict": true` — enables all strict type checks; catches bugs before runtime
- `"resolveJsonModule": true` — lets you `import data from './data.json'` directly
- `"esModuleInterop": true` — allows `import winston from 'winston'` syntax (without this, you'd need `import * as winston`)

---

## Step 6 — Install Playwright

```bash
npm install --save-dev @playwright/test
npx playwright install
```

**First command:** Installs the Playwright test runner package.

**Second command:** Downloads the actual browser binaries (Chromium, Firefox, WebKit). Without this, Playwright knows how to drive browsers but has no browsers to drive. This is ~300 MB and goes into `~/.cache/ms-playwright` (not in your project folder).

**Why separate?** Playwright's philosophy is: bring your own browser. You can pin to exact browser versions for reproducibility in CI.

---

## Step 7 — Install All Project Dependencies

Looking at `package.json` from your codebase:

```bash
# Runtime dependencies
npm install dotenv crypto-js csvtojson csv-writer @faker-js/faker winston moment-timezone

# Dev dependencies
npm install --save-dev @types/crypto-js @typescript-eslint/eslint-plugin eslint eslint-config-standard-with-typescript eslint-plugin-import eslint-plugin-n eslint-plugin-promise
```

**Why each:**
| Package | Purpose |
|---|---|
| `dotenv` | Load `.env` files into `process.env` |
| `crypto-js` | AES encryption for credentials |
| `csvtojson` | Parse CSV files into JSON |
| `csv-writer` | Write JSON data out to CSV |
| `@faker-js/faker` | Generate realistic fake test data |
| `winston` | Structured logging with transports |
| `moment-timezone` | Timezone-aware date formatting for logs |
| `eslint` + plugins | Code quality linting |

---

## Step 8 — Create the Folder Structure

```bash
mkdir -p src/config src/fixtures src/logging src/pages src/testdata src/tests src/utils
```

**Why this structure?** Each folder has a single responsibility. This is the "separation of concerns" principle applied to folder layout. It makes the project navigable for anyone joining the team — they know exactly where to look for anything.

---

## Step 9 — Initialize Playwright Config

```bash
npx playwright test --init
# OR manually create playwright.config.ts
```

Your project's config is hand-written, not auto-generated. We'll walk through it fully in Part 5.

---

## Step 10 — Set Up ESLint

Create `.eslintrc.json` at the root. ESLint enforces coding standards so the whole team writes consistent code. Running `npm run lint` before committing catches style and type errors before they reach CI.

---

## Step 11 — Set Up .gitignore

Never commit these:
```
node_modules/
playwright-report/
test-results/
src/config/auth.json
src/config/.env
```

**Why:** `node_modules` is huge and regeneratable. Reports are build artifacts. `.env` contains secrets — committing this to GitHub is a security incident.

---

# PART 1 — Project Overview {#part-1}

## What Does This Project Do?

This is an **enterprise-grade test automation framework** built with Playwright and TypeScript targeting **Salesforce** as the Application Under Test (AUT).

Think of it like this: if Salesforce is a car, this framework is the professional test track, the diagnostic equipment, the fuel management system, and the safety protocols all built together into one organized pit.

It doesn't just "run some tests." It handles:
- Encrypted credentials so no passwords are ever in plain text
- Structured logging with timezone awareness (IST — your timezone, ktx)
- Multi-environment support (dev, QA, UAT)
- Self-healing locators so minor UI changes don't break tests
- Data-driven testing from JSON and CSV files
- Synthetic data generation with Faker
- API testing and mocking alongside UI tests
- Visual regression testing
- Auth state reuse so login doesn't repeat across every test
- Serial test execution for workflows that depend on each other
- CI/CD integration via GitHub Actions

---

## Folder Structure Annotated

```
enterprise-playwright-framework/
│
├── playwright.config.ts        # Master control — browser, timeouts, reporters, env
├── tsconfig.json               # TypeScript compiler rules
├── .eslintrc.json              # Code quality rules
├── package.json                # Dependencies and scripts manifest
├── .gitignore                  # What git should never track
│
├── .github/
│   └── workflows/
│       └── main.yml            # GitHub Actions CI/CD pipeline definition
│
└── src/                        # All source code lives here
    ├── config/
    │   ├── .env                # Default environment variables (credentials)
    │   ├── .env.qa             # QA environment overrides
    │   └── .env.uat            # UAT environment overrides
    │
    ├── fixtures/
    │   └── loginFixture.ts     # Custom Playwright fixtures — pre-wired login
    │
    ├── logging/
    │   ├── test_run.log        # Info-level log file (all test activity)
    │   └── test_error.log      # Error-level log file (only failures)
    │
    ├── pages/                  # Page Object Model classes
    │   ├── LoginPage.ts        # Salesforce login page interactions
    │   ├── HomePage.ts         # Post-login home/dashboard interactions
    │   ├── ContactPage.ts      # Contacts tab CRUD operations
    │   └── CasePage.ts         # Cases creation from contact detail page
    │
    ├── testdata/               # All external test data
    │   ├── contacts.json       # Simple array of contact names
    │   ├── datademo.json       # Generated demo data for data-driven tests
    │   ├── contactCaseFlow.json # Single-record data for serial workflow test
    │   ├── testData_en.json    # Faker-generated user records (JSON)
    │   ├── testData_en.csv     # Same data in CSV format
    │   └── data.csv            # Simple CSV for converter demo
    │
    ├── tests/                  # Test spec files — the actual test cases
    │   ├── loginTest.spec.ts   # Login scenarios + auth state saving
    │   ├── contactTest.spec.ts # Data-driven contact creation tests
    │   ├── SerialTest.spec.ts  # Serial workflow: login → contact → case
    │   ├── fixtureTest.spec.ts # Demonstrates custom fixture usage
    │   ├── apiTest.spec.ts     # API-only tests (no UI)
    │   ├── apiMockTest.spec.ts # Request interception and mocking
    │   ├── visTest.spec.ts     # Visual/screenshot regression tests
    │   └── oldTest.spec.ts     # Archived/commented utility tests
    │
    └── utils/                  # Reusable helper utilities
        ├── CryptojsUtil.ts     # AES encrypt/decrypt functions
        ├── EncryptEnvFile.ts   # Bulk-encrypt/decrypt the .env file
        ├── CsvtoJsonUtil.ts    # CSV file → JSON file converter
        ├── FakerDataUtil.ts    # Generate fake test data (JSON + CSV export)
        ├── fakersample.ts      # Quick demo of faker usage
        ├── LoggerUtil.ts       # Winston logger singleton
        └── SelfHealingUtill.ts # Try multiple locators, use first valid one
```

**Why this structure matters at scale:**

Imagine a team of 5 QA engineers on this project. With this layout:
- The engineer fixing a locator goes to `src/pages/`
- The engineer adding test data goes to `src/testdata/`
- The engineer writing a new test goes to `src/tests/`
- The engineer debugging logs checks `src/logging/`

Nobody steps on each other. This is **separation of concerns** applied to folder design — one of the most important principles in enterprise codebases.

---

# PART 2 — TypeScript Foundations {#part-2}

> Bob says: "TypeScript is JavaScript with a safety net. Every TypeScript feature you'll see in this project exists to catch a class of bug before it ever runs."

---

## 2.1 — Classes and Access Modifiers

**What:** A class is a blueprint for creating objects. Access modifiers (`private`, `public`, `protected`) control which parts of the code can see and use each piece of data.

**The problem TypeScript solves:** In plain JavaScript, any code anywhere can read or modify any variable. In a large test framework, this causes bugs where one test accidentally stomps on another test's state.

**From your codebase — LoginPage.ts:**

```typescript
export default class LoginPage {
  private readonly usernameInputSelector = "#username";
  private readonly passwordInputSelector = "#password";
  private readonly loginButtonSelector = "#Login";

  constructor(private page: Page) {}
```

**Line by line explanation:**

`private readonly usernameInputSelector = "#username"` — This is a class field (property). Three things are happening:
- `private`: Only code inside `LoginPage` class can read this. Tests can't accidentally use the raw selector string — they must call a method.
- `readonly`: Once set, it can never be reassigned. CSS selectors for stable elements shouldn't change during a test run.
- `= "#username"`: Initialized inline. No need for a constructor assignment.

**Why this way?** If `#username` appears in 10 test files as a raw string and Salesforce changes it to `.username-field`, you'd need to update 10 files. With POM, you update one line in `LoginPage.ts`. That's **single source of truth** — an industry-standard principle.

`constructor(private page: Page) {}` — This is TypeScript shorthand. `private page: Page` in the constructor parameter list does three things at once:
1. Declares a class property named `page`
2. Makes it `private`
3. Assigns the constructor argument to it

**The equivalent in plain TypeScript (longer form):**
```typescript
private page: Page;
constructor(page: Page) {
  this.page = page;
}
```

The shorthand is idiomatic TypeScript — you'll see it in every professional POM implementation. It's considered a best practice because it reduces boilerplate.

---

## 2.2 — Type Annotations

**What:** Explicitly telling TypeScript what type a variable, parameter, or return value is.

**The problem it solves:** In JavaScript, you can pass a number where a string is expected and only find out at runtime (when the test is running in CI at midnight). TypeScript catches this at compile time.

**From your codebase — FakerDataUtil.ts:**

```typescript
interface UserData {
  name: string;
  email: string;
  username: string;
  phone: string;
  age: number;
  address: string;
}

export const generateTestData = (numRecords: number): UserData[] => {
```

`(numRecords: number)` — the parameter must be a number. Pass a string and TypeScript refuses to compile.

`: UserData[]` — the return type is an array of `UserData` objects. This means the caller knows exactly what shape the data will be, and TypeScript will complain if any field is missing or the wrong type.

**Why the `interface`?** An interface defines the "contract" for an object's shape. It's documentation that TypeScript enforces. When you write `contact.email`, TypeScript knows `email` exists and is a string — no guessing.

---

## 2.3 — Type Inference

**What:** TypeScript figuring out the type without you explicitly writing it.

**From your codebase — SerialTest.spec.ts:**

```typescript
let page: Page;
```

Here the type is explicitly declared. But in many places:

```typescript
const authFile = "src/config/auth.json";
```

TypeScript infers `authFile` is of type `string` from the assigned value. You don't need to write `const authFile: string = "..."`. This is inference at work.

**The rule of thumb:** Declare types explicitly for function parameters and return types (public contracts). Let TypeScript infer types for local variables where the assignment makes the type obvious.

---

## 2.4 — async / await

**What:** A syntax for working with asynchronous operations (things that take time) without writing nested callbacks.

**The problem it solves:** Browser automation is inherently async — click a button, wait for a page to load, find an element, etc. Without async/await, you'd write deeply nested `.then().then().then()` chains that are hard to read and debug.

**From your codebase — LoginPage.ts:**

```typescript
async quickLogin(username: string, password: string) {
  await this.navigateToLoginPage();
  await this.fillUsername(username);
  await this.fillPassword(password);
  return await this.clickLoginButton();
}
```

`async` on the method declaration means: "this function returns a Promise." Playwright operations return Promises. If you call them without `await`, Playwright starts the action but your code moves on before it finishes — the username field might not exist yet.

`await` means: "pause here until this Promise resolves." It makes async code read like synchronous code, top to bottom.

**The analogy:** `await` is like saying "don't move on to filling the password until the username field exists and is ready." Without it, you'd be typing a password into a field that hasn't rendered yet.

---

## 2.5 — Modules and Imports

**What:** ES modules let you split code into files. `export` makes something available to other files. `import` brings it in.

**From your codebase — loginTest.spec.ts:**

```typescript
import { expect, test } from "@playwright/test";
import LoginPage from "../pages/LoginPage";
import logger from "../utils/LoggerUtil";
import { decrypt } from "../utils/CryptojsUtil";
```

**Three import patterns at work:**

`import { expect, test } from "@playwright/test"` — **Named imports**. The package exports multiple things; you pick the specific names you need. Curly braces mean named.

`import LoginPage from "../pages/LoginPage"` — **Default import**. When a file does `export default class LoginPage`, it has one main thing to export. You import it without curly braces and can name it anything (though you'd typically keep the same name).

`import { decrypt } from "../utils/CryptojsUtil"` — Named import from a local file. `CryptojsUtil.ts` exports both `encrypt` and `decrypt` as named exports. You only bring in `decrypt` here — import only what you need.

**Why this matters:** Modules enforce that code is organized, purposeful, and only exposes what needs to be public. `LoggerUtil.ts` does `export default logger` — the whole framework uses one logger instance, not ten different ones.

---

## 2.6 — The Non-Null Assertion (`!`)

**What:** Tells TypeScript "I know this value can't be null or undefined, trust me."

**From your codebase — loginTest.spec.ts:**

```typescript
await loginPage.fillUsername(decrypt(process.env.username!));
```

`process.env.username` is typed as `string | undefined` by TypeScript — environment variables might not be set. The `!` operator asserts it's definitely a string. Without it, TypeScript would give a compile error because you'd be passing `string | undefined` to a function that only accepts `string`.

**When to use it:** Only when you're certain the value is set — for example, because you load it from a `.env` file via dotenv at startup. Don't use `!` to silence TypeScript when you're not sure; that defeats the purpose.

---

## 2.7 — Generics

**What:** Type parameters that let you write reusable code that works with any type while still being type-safe.

**From your codebase — loginFixture.ts:**

```typescript
type UIPages = {
  homePage: HomePage;
};

export const test = base.extend<UIPages>({
```

`base.extend<UIPages>` — The `<UIPages>` is a generic type argument. It tells Playwright's `extend` function: "the new fixture properties you're adding look like this `UIPages` type." Playwright uses this to know that when a test asks for `{ homePage }` in its argument, it should provide an object of type `HomePage`.

**The analogy:** Generics are like molds. `base.extend<T>` is a mold that says "extend with this shape `T`." You pass in `UIPages` as the shape, and TypeScript ensures everything is consistent.

---

## 2.8 — Optional Chaining (`?.`)

**What:** Safely access a property or call a method on something that might be null/undefined, without throwing a runtime error.

**From your codebase — SelfHealingUtill.ts:**

```typescript
await usernameInputLocator?.fill(username);
const enteredValue = await usernameInputLocator?.inputValue();
```

`usernameInputLocator` is typed as `Locator | null`. Calling `.fill()` on `null` would crash. The `?.` says: "only call `.fill()` if `usernameInputLocator` is not null or undefined. Otherwise, skip this and return `undefined`."

**Why this way?** The self-healing utility returns `null` if no valid locator is found. Using `?.` lets the code proceed gracefully rather than crashing with a `Cannot read properties of null` error.

---

## 2.9 — Type Aliases

**From your codebase — loginFixture.ts:**

```typescript
type UIPages = {
  homePage: HomePage;
};
```

`type` creates an alias for a type definition. Here it names the shape of the fixture object. It's similar to an `interface` but more flexible — `type` can represent unions, intersections, and primitives, not just object shapes. For object shapes in this context, `type` and `interface` are largely interchangeable.

---

# PART 3 — Playwright Core Concepts {#part-3}

> Bob says: "Playwright's API is designed to be close to how a human uses a browser. Once you internalize the Browser → Context → Page hierarchy, everything else makes sense."

---

## 3.1 — The Browser / BrowserContext / Page Hierarchy

**The mental model:**

```
Browser (the application, e.g., Chrome)
  └── BrowserContext (an isolated session — like an incognito window)
        └── Page (a single tab)
```

**Why three layers?**

A **BrowserContext** is a fully isolated session. Two BrowserContexts don't share cookies, localStorage, or cache — even if they're in the same Browser. This is critical for tests that need to run in parallel without interfering with each other.

A **Page** is one tab within a context. Most tests work with a single page.

**From your codebase — SerialTest.spec.ts:**

```typescript
test.beforeAll(async ({ browser }) => {
  page = await browser.newPage();
```

Here `browser` is Playwright's `Browser` object. Calling `browser.newPage()` creates a new BrowserContext and a Page inside it in one step. For the serial test suite, one page is shared across all tests — that's intentional (explained in Part 4).

**From loginTest.spec.ts (auth state):**

```typescript
test.skip("Login with auth file", async ({ browser }) => {
  const context = await browser.newContext({ storageState: authFile });
  const page = await context.newPage();
```

Here, a new **BrowserContext** is created with pre-loaded auth state (cookies + localStorage from a previous login). This is explicit separation — you're saying "create an isolated session that starts already logged in."

---

## 3.2 — The `page` Fixture

**What:** When a test function has `{ page }` in its argument, Playwright automatically creates a fresh BrowserContext and Page for that test. No setup needed.

**From your codebase — loginTest.spec.ts:**

```typescript
test("simple login test", async ({ page }) => {
  const loginPage = new LoginPage(page);
```

Playwright injects the `page` object. It's fresh — no cookies from previous tests, completely clean. After the test, Playwright closes it automatically.

**Why this way?** This is the **fixture pattern** — pre-configured objects injected into your tests. It removes boilerplate and ensures every test starts from a known-good state. Industry best practice.

---

## 3.3 — Locator Strategies

**What:** How Playwright finds elements on the page.

Playwright has several locator strategies, and your codebase uses all the important ones:

**CSS Selector (most common):**
```typescript
// LoginPage.ts
await this.page.locator("#username").fill(username);
```
`#username` is a CSS ID selector. Fast and precise when IDs exist.

**Role-based (semantic, preferred by Playwright team):**
```typescript
// ContactPage.ts
await this.page.getByRole('button', { name: 'New' }).click();
await this.page.getByRole('link', { name: 'Contacts' }).click();
```
`getByRole` finds elements by their ARIA role. This is how screen readers find elements. It's more resilient than CSS selectors because it's based on semantic meaning, not CSS class names (which can change).

**Label-based:**
```typescript
// CasePage.ts
await this.page.getByLabel(this.caseOriginDropdownLocator).click();
```
Finds an input associated with a `<label>`. Salesforce Lightning uses accessible labels extensively.

**Placeholder-based:**
```typescript
// ContactPage.ts
await this.page.getByPlaceholder(this.firstNameTextFieldLocator).fill(fname);
```
Finds an input by its `placeholder` attribute. Useful in Salesforce forms where labels and IDs can be dynamic.

**Alt text:**
```typescript
// visTest.spec.ts
const logo = await page.getByAltText('Salesforce');
```
Finds images by their `alt` attribute.

**Why role-based is the industry preference:** CSS selectors break when developers refactor class names or IDs. Role-based selectors are tied to accessibility semantics, which are much more stable. When you see `getByRole('button', { name: 'Save' })`, you're saying "find the button that a screen reader would call 'Save'" — that meaning doesn't change when CSS changes.

---

## 3.4 — Auto-Waiting

**What:** Playwright automatically waits for elements to be actionable before interacting with them. You don't write `wait()` calls manually.

**Why this is revolutionary:** Older frameworks (like Selenium) require explicit waits: `driver.implicitlyWait(10)` or `WebDriverWait(driver, 10).until(...)`. Forgetting waits causes flaky tests.

Playwright's auto-waiting checks:
1. Is the element attached to the DOM?
2. Is it visible?
3. Is it stable (not animating)?
4. Is it enabled (not disabled)?
5. Is it editable (for inputs)?

Only when all conditions pass does it perform the action.

**From your codebase:** Every `await page.locator(...).click()` and `await page.locator(...).fill()` has auto-waiting built in. You don't add explicit waits unless you need to wait for a specific state.

**The one explicit wait in your codebase:**

```typescript
// HomePage.ts
await expect(this.page.getByTitle(this.serviceTitleLocator)).toBeVisible({
  timeout: 15000,
});
```

This sets a custom timeout of 15 seconds specifically for the Service title assertion — because Salesforce login can be slow. The global timeout in `playwright.config.ts` is 45 seconds, but this shows how to override per-assertion.

---

## 3.5 — Assertions with `expect`

**What:** `expect` is Playwright's assertion library. It combines assertions with auto-waiting.

**From your codebase:**

```typescript
await expect(this.page.locator(this.contactFullNameLabelLocator))
  .toContainText(`${fname} ${lname}`);
```

`expect(locator).toContainText(...)` doesn't just check the current DOM state. It polls the DOM repeatedly until the text appears or the timeout expires. This prevents false failures when the UI is still updating.

**Other assertions used in your codebase:**

```typescript
expect(response).toHaveProperty("page");       // Object has property
expect(response.page).toEqual(2);               // Strict equality
expect(typeof response.page).toBe("number");    // Type check
expect(Array.isArray(response.data)).toBe(true);// Array check
await expect(page).toHaveScreenshot();          // Visual snapshot
```

---

## 3.6 — Hooks: beforeAll, afterAll, beforeEach

**What:** Code that runs at specific points in the test lifecycle.

**From your codebase — SerialTest.spec.ts:**

```typescript
test.beforeAll(async ({ browser }) => {
  page = await browser.newPage();
  const loginPage = new LoginPage(page);
  const homePage = await loginPage.quickLogin(...);
  await homePage.expectServiceTitleToBeVisible();
});

test.afterAll(async () => {
  await page.close();
});
```

`beforeAll` — runs once before all tests in the describe block. Used here to log in once and share the session.

`afterAll` — runs once after all tests finish. Used to clean up (close the page).

**Why this matters:** If you logged in inside each individual test, and there are 5 serial tests, you'd log in 5 times. That's 5x the time, 5x the chance of a timeout. Login once, share the session — that's efficient test design.

---

## 3.7 — test.describe and test.describe.configure

**From your codebase — SerialTest.spec.ts:**

```typescript
test.describe.configure({ mode: "serial" });
```

`test.describe` groups related tests. `configure({ mode: "serial" })` tells Playwright: run tests in this file one at a time, in order, not in parallel. This is critical when tests have dependencies (test 2 needs the data created by test 1).

**Default behavior:** Playwright runs tests in parallel. This is fast but requires tests to be independent. For workflow tests (create contact → create case → verify case), serial mode is the right call.

---

## 3.8 — test.skip

**From your codebase — contactTest.spec.ts:**

```typescript
test.skip(`Advance DD test for ${contact.firstName} `, async ({ page }) => {
```

`test.skip` marks a test as skipped — it shows in the report as "skipped" rather than passing or failing. Used here because the data-driven tests require a live Salesforce connection that may not always be available (e.g., in a demo environment).

**Industry practice:** Use `test.skip` for tests under construction or environment-dependent tests that can't always run. Use `test.only` to run a single test in isolation during debugging (but never commit `test.only` — the config has `forbidOnly: !!process.env.CI` to catch this).

---

# PART 4 — Enterprise Patterns Deep Dive {#part-4}

> Bob says: "This is where the framework goes from 'tests that work' to 'tests that work at scale, in CI, for a team of engineers, for 2 years.' These patterns are what separates enterprise automation from student projects."

---

## 4.1 — Authentication State Reuse

**The problem at scale:** Logging in takes 3–8 seconds per test. If you have 50 tests, that's 4 minutes of just logging in. At 500 tests, it's 40 minutes. For a CI pipeline that needs to give fast feedback, this is unacceptable.

**The solution: save session cookies to a file, load them instead of logging in.**

**From your codebase — loginTest.spec.ts:**

```typescript
const authFile = "src/config/auth.json";

test("simple login test", async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.navigateToLoginPage();
  await loginPage.fillUsername(decrypt(process.env.username!));
  await loginPage.fillPassword(decrypt(process.env.password!));
  const homePage = await loginPage.clickLoginButton();
  await homePage.expectServiceTitleToBeVisible();
  
  // This line is the key — save the entire session state
  await page.context().storageState({ path: authFile });
});

test.skip("Login with auth file", async ({ browser }) => {
  const context = await browser.newContext({ storageState: authFile });
  const page = await context.newPage();
  await page.goto("https://...force.com/lightning/page/home");
  await expect(page.getByRole("link", { name: "Accounts" })).toBeVisible();
});
```

`page.context().storageState({ path: authFile })` — serializes the BrowserContext's state (cookies, localStorage, sessionStorage) to a JSON file. This is the same data your browser stores to keep you logged in.

`browser.newContext({ storageState: authFile })` — creates a new context pre-loaded with those cookies. The app sees valid session cookies and skips the login flow entirely.

**Why `auth.json` is in `.gitignore`:** It contains session tokens that are as sensitive as passwords. Never commit it.

**Industry note:** This is a Playwright-recommended pattern. In large frameworks, you run a global setup step that logs in once and saves `auth.json`, then all tests load from it. Zero redundant logins.

---

## 4.2 — Password Encryption with CryptoJS

**The problem:** You can't put plain-text passwords in `.env` files that might be shared, copied to CI, or accidentally logged. Even encrypted `.env` files are better than plaintext.

**The solution: AES encryption with a SALT that lives only in the runtime environment.**

**From your codebase — CryptojsUtil.ts:**

```typescript
export function encrypt(text: string) {
  const SALT = process.env.SALT || "omg";
  const cipherText = CryptoJSUtil.AES.encrypt(text, SALT).toString();
  return cipherText;
}

export function decrypt(cipherText: string) {
  const SALT = process.env.SALT || "defaultSALT";
  const bytes = CryptoJSUtil.AES.decrypt(cipherText, SALT);
  const originalText = bytes.toString(CryptoJSUtil.enc.Utf8);
  return originalText;
}
```

**The workflow:**
1. Run `encryptEnvFile()` once — it reads `.env`, encrypts every value with AES using `SALT`, and writes the ciphertexts back
2. The `.env` file now contains encrypted values like `U2FsdGVkX197mBdFhci0yNUxOuds...` instead of `mypassword123`
3. At test time: `decrypt(process.env.userid!)` decrypts on the fly using `SALT`
4. `SALT` is stored separately — as a CI environment variable or a separate secret, never in `.env`

**From EncryptEnvFile.ts:**

```typescript
export function encryptEnvFile() {
  const SALT = process.env.SALT || "defaultSALT";
  const envFileContent = fs.readFileSync(envFilePath, "utf8");
  const envLines = envFileContent.split("\n");

  const encryptedLines = envLines.map((line) => {
    const [key, value] = line.split("=");
    if (value) {
      const encryptedValue = CryptoJSUtilFile.AES.encrypt(value, SALT).toString();
      return `${key}=${encryptedValue}`;
    }
    return line;
  });

  fs.writeFileSync(envFilePath, encryptedLines.join("\n"), "utf8");
}
```

`envFileContent.split("\n")` — splits the file into lines.
`line.split("=")` — splits each line into key and value.
`AES.encrypt(value, SALT)` — encrypts the value.
The `map` preserves the key, replaces the value with encrypted text.

**Why this approach:** The security model is "the `.env` file alone is useless." An attacker who gets the `.env` file still can't decrypt anything without `SALT`. `SALT` lives only in CI secrets or a developer's machine environment.

**The trade-off:** AES with a password (not a proper key derivation) isn't cryptographically perfect, but it's significantly better than plaintext and appropriate for test credentials (not production banking systems). It's a pragmatic enterprise choice.

---

## 4.3 — Self-Healing Locators

**The problem:** Salesforce Lightning updates frequently. A locator like `#username` might change to `.username-input` in the next release. Every change breaks tests.

**The solution:** Maintain an array of possible locators for the same element. Try each one in order. Use the first one that works.

**From your codebase — SelfHealingUtill.ts:**

```typescript
export default async function findValidElement(
  page: Page,
  locators: string[]
): Promise<Locator | null> {
  let validElement: Locator | null = null;
  const TIMEOUT_MS = 5000;

  for (const locator of locators) {
    try {
      const element = page.locator(locator);
      await element.waitFor({ state: "attached", timeout: TIMEOUT_MS });
      validElement = element;
      logger.info(`Found valid element with locator: ${locator}`);
      break;
    } catch (error) {
      logger.error(`Invalid locator: ${locator}`);
    }
  }

  return validElement;
}
```

**Used in LoginPage.ts:**

```typescript
private readonly usernameInputSelectors = [
  "#username",
  'input[name="username"]',
  ".username",
  '//*[@id="username"]'
];

async fillUsername_selfheal(username: string) {
  let usernameInputLocator = await findValidElement(this.page, this.usernameInputSelectors);
  await usernameInputLocator?.fill(username);
}
```

**Walk-through:**
1. Try `#username` — wait 5 seconds for it to be attached to DOM
2. If it works → use it, break out of loop
3. If it times out → log the failure, try the next one
4. If all fail → return `null`, calling code handles gracefully with `?.`

**Why `state: "attached"` not `"visible"`?** Attached means the element exists in the DOM. Visible means it's also rendered and not hidden. Using `attached` is more permissive — some Salesforce elements are in the DOM but not yet visible during transitions.

**The industry context:** "Self-healing" is a term used in AI-assisted test tools like Healenium and Testim. This project implements a simpler manual version of the same concept. For a more complete implementation, you'd log which fallback locator was used and alert the team to update the primary selector.

---

## 4.4 — Custom Fixtures

**The problem:** Every test that needs to start logged-in repeats the same setup code: navigate, fill username, fill password, click login. That's 4–5 lines of boilerplate in every test.

**The solution:** Playwright fixtures let you define setup code once and inject a pre-configured object into any test that requests it.

**From your codebase — loginFixture.ts:**

```typescript
type UIPages = {
  homePage: HomePage;
};

export const test = base.extend<UIPages>({
  homePage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigateToLoginPage();
    await loginPage.fillUsername(decrypt(process.env.userid!));
    await loginPage.fillPassword(decrypt(process.env.password!));
    const homePage = await loginPage.clickLoginButton();
    await use(homePage); // <-- hand control to the test
    // teardown code would go here (after use())
  },
});
```

**The `use` function is critical:** It signals "the setup is done, give the test its turn." Everything before `use(homePage)` is setup. Everything after `use()` (if you add it) is teardown. This is the fixture lifecycle.

**From fixtureTest.spec.ts — how it's consumed:**

```typescript
import { test } from "../fixtures/loginFixture";

test("Fixture test", async ({ homePage }) => {
  await homePage.expectServiceTitleToBeVisible();
});
```

The test imports `test` from the fixture file (not from `@playwright/test`). It just destructures `{ homePage }` — and gets a fully logged-in `HomePage` object, ready to use. No boilerplate.

**Why this is an industry best practice:** DRY (Don't Repeat Yourself). Login logic exists in one place. Change it once, every test benefits. This scales to 200 tests with zero change to the tests themselves.

---

## 4.5 — Serial Tests with Shared State

**The problem:** Some test scenarios are workflows. Step 1 creates data that Step 2 depends on. If Step 2 runs before Step 1, it fails — not because it's broken, but because the data doesn't exist yet.

**The solution:** `test.describe.configure({ mode: "serial" })` forces sequential execution and shared state.

**From your codebase — SerialTest.spec.ts:**

```typescript
test.describe.configure({ mode: "serial" });
let page: Page;

test.beforeAll(async ({ browser }) => {
  page = await browser.newPage();
  const loginPage = new LoginPage(page);
  const homePage = await loginPage.quickLogin(...);
});

test("Create Contact and Open", async () => {
  const contactpage = new ContactPage(page);
  await contactpage.createNewContact(testdata.contactFName, testdata.contactLName);
  await contactpage.findExistingContactByLastName(testdata.contactLName);
});

test("Create Case Test", async () => {
  const casePage = new CasePage(page);
  await casePage.createNewCaseFromContactDetailPage(...);
});

test.afterAll(async () => {
  await page.close();
});
```

Key observations:
- `page` is declared outside tests at file scope — it persists across all tests
- `beforeAll` populates `page` once
- Each test receives `page` via closure (not via Playwright injection — notice the tests take no arguments)
- The contact created in test 1 exists in Salesforce when test 2 runs (same session, same page)

**The tradeoff:** Serial tests are slower (no parallelism) and have a "poison" effect — if test 1 fails, tests 2 and 3 also fail (they depend on test 1's output). Use serial only when the workflow genuinely requires it.

**Playwright docs reference this exact pattern** at `https://playwright.dev/docs/test-retries#reuse-single-page-between-tests` — noted right in your SerialTest file.

---

## 4.6 — API Testing and Mocking

**The problem:** UI tests are slow. Every test that goes through the full UI takes 5–30 seconds. For certain verifications, it's faster and more reliable to call the API directly. For other tests, you need to control what the API returns to test edge cases.

**Three modes in your codebase:**

### Mode 1: Pure API Testing (apiTest.spec.ts)

```typescript
test("API test with existing context", async ({ page }) => {
  const context = page.request;
  const response = await (await context.get("/api/users?page=2")).json();
  expect(response.page).toEqual(2);
```

`page.request` is Playwright's built-in HTTP client. It shares the browser context (including cookies/auth headers). Calls go through the same session as the UI.

```typescript
test("API test with new context", async ({ playwright }) => {
  const apirequest = playwright.request;
  const newcontext = await apirequest.newContext({
    baseURL: "https://cat-fact.herokuapp.com",
  });
  const apiResponse = await newcontext.get("/facts/");
```

`playwright.request.newContext` creates a completely independent HTTP client with its own baseURL — no browser context, no cookies. Pure API testing.

### Mode 2: Request Monitoring (apiMockTest.spec.ts)

```typescript
page.on("request", (request) => {
  logger.info(`Requested Url is ${request.url()}`);
  logger.info(`Requested method is ${request.method()}`);
});

page.on("response", (response) => {
  logger.info(`Response status is ${response.status()}`);
});
```

Event listeners on `page` capture every HTTP request/response. Useful for debugging network traffic, verifying which APIs are called, and monitoring performance.

### Mode 3: Request Interception and Mocking

```typescript
// Intercept and modify headers:
await page.route("**/*", (route) => {
  const headers = route.request().headers();
  headers["X-Custom-Header"] = "integration-check";
  route.continue({ headers });
});

// Full mock — return fake data:
page.route(
  "https://demo.playwright.dev/api-mocking/api/v1/fruits",
  async (route) => {
    const json = [
      { name: "Mandarin", id: 3 },
      { name: "Tangerine", id: 1 },
    ];
    await route.fulfill({ json });
  }
);
```

`page.route(pattern, handler)` intercepts matching requests. You can:
- `route.continue()` — let it pass through, optionally modified
- `route.fulfill()` — return a fake response
- `route.abort()` — simulate a network error

**Why mocking is enterprise-essential:** You can test the UI's error handling for a "503 Service Unavailable" response without taking your API down. You can test a UI component with a specific data shape that's hard to create in a real environment. Tests run faster because no real API calls are made.

---

## 4.7 — Visual Regression Testing

**The problem:** A CSS change might break the visual layout without breaking any functional assertions. `expect(button).toBeVisible()` passes even if the button moved off-screen due to a CSS bug.

**The solution:** Capture a screenshot on first run, compare subsequent runs against it pixel by pixel.

**From your codebase — visTest.spec.ts:**

```typescript
test('Screenshot compare test', async ({ page }) => {
  await page.goto('/');
  const loginPage = new LoginPage(page);
  await loginPage.fillUsername("demo");
  await expect(page).toHaveScreenshot();
});
```

First run: Playwright takes a screenshot and saves it to `visTest.spec.ts-snapshots/Screenshot-compare-test-1-Google-Chrome-win32.png` (you can see this file in your project).

Subsequent runs: Playwright takes a new screenshot, compares it pixel-by-pixel to the saved baseline. If they differ beyond a threshold, the test fails.

**Additional visual assertions in your project:**

```typescript
const boundingBox = await logo?.boundingBox();
expect(boundingBox.width).toBe(160.890625);
expect(boundingBox.height).toBe(112.984375);
```

`boundingBox()` returns the element's exact position and size on the page. This verifies layout precision — the logo is exactly 160.89px wide.

```typescript
const logoStyle = await logo.evaluate((element) => {
  const style = window.getComputedStyle(element);
  return { color: style.color };
});
expect(logoStyle.color).toBe("rgb(22, 50, 92)");
```

`evaluate()` runs JavaScript inside the browser context. `getComputedStyle` returns the final rendered CSS. This verifies the exact rendered color value.

---

## 4.8 — Structured Logging with Winston

**The problem:** `console.log` is not enough in enterprise. You need: timestamped logs, log levels (info vs error), log rotation, multiple output targets (console + files), and timezone-aware timestamps.

**From your codebase — LoggerUtil.ts:**

```typescript
const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp({ format: () => moment().tz(timeZone).format() }),
    customFormat
  ),
  transports: [
    new winston.transports.Console({ level: "debug" }),
    new winston.transports.File({
      filename: path.join(loggingDir, "test_run.log"),
      maxFiles: 5,
      maxsize: 300 * 1024,
      level: "info",
    }),
    new winston.transports.File({
      filename: path.join(loggingDir, "test_error.log"),
      maxFiles: 5,
      maxsize: 10 * 1024,
      level: "error",
    }),
  ],
});

export default logger;
```

**Three transports (outputs):**
1. **Console** — `debug` level and above. Everything during local development.
2. **test_run.log** — `info` level and above. All meaningful test activity. Max 300KB per file, keep 5 files (rotation).
3. **test_error.log** — `error` level only. Just failures. Small file (10KB max) easy to share with developers.

**Why `export default logger`?** The singleton pattern. One logger instance is created once and exported. Every file that imports it shares the same logger — same transports, same configuration. If you created a new logger in each file, you'd have inconsistent configuration and possibly conflicting file handles.

**Why IST timezone?** You're in Chennai, ktx. Logs with UTC timestamps are harder to correlate with your daily schedule. `Asia/Kolkata` means you see `2024-01-15T14:30:00+05:30` — immediately meaningful.

---

## 4.9 — Data-Driven Testing with Faker

**From your codebase — FakerDataUtil.ts:**

```typescript
const generateUserData = (): UserData => {
  return {
    name: faker.person.firstName(),
    email: faker.internet.email(),
    username: faker.internet.userName(),
    phone: faker.phone.number(),
    age: faker.number.int({ min: 18, max: 99 }),
    address: faker.location.country(),
  };
};

export const generateTestData = (numRecords: number): UserData[] => {
  const testData: UserData[] = faker.helpers.multiple(generateUserData, {
    count: numRecords
  });
  return testData;
};
```

`faker.helpers.multiple(fn, { count: n })` — calls `generateUserData` `n` times and returns an array. Cleaner than a manual for-loop.

**From contactTest.spec.ts:**

```typescript
import cdata from "../testdata/datademo.json";

for (const contact of cdata) {
  test.skip(`Advance DD test for ${contact.firstName}`, async ({ page }) => {
    // ... creates contact in Salesforce with this data
  });
}
```

This is data-driven testing at its most direct: loop over JSON data, generate one test per record. Each test gets a unique name including the contact's first name — making failures easy to identify in reports.

**Why Faker over hardcoded data?** Hardcoded test data causes collisions in Salesforce — if you always create "John Doe", you get duplicate records and tests interfere with each other. Faker generates unique, realistic data every run.

---

# PART 5 — Config File Complete Walkthrough {#part-5}

`playwright.config.ts` is the constitution of your framework. Everything flows from it.

```typescript
if (!process.env.NODE_ENV) {
  require("dotenv").config({ path: `${__dirname}//src//config//.env` });
} else {
  require("dotenv").config({
    path: `${__dirname}//src//config//.env.${process.env.NODE_ENV}`,
  });
}
```

**What this does:** Before Playwright loads, this runs. It checks if `NODE_ENV` is set. If not, load the default `.env`. If it is set (e.g., `NODE_ENV=qa`), load `.env.qa`. This is environment switching.

**Why at the top of the config?** The config itself might read env vars (like `baseURL: process.env.BASE_URL`). Environment must load before config is evaluated.

**Why double backslashes (`//`)?** Path handling quirk. On Windows, `\\` is the separator. On macOS, `/` works. Using `//` is a defensive choice that works on both (extra slashes are treated as single in most contexts).

---

```typescript
export default defineConfig({
  timeout: 45000,
```

**`timeout: 45000`** — global test timeout of 45 seconds. If any single test runs longer than 45 seconds, it's automatically failed. Salesforce can be slow, so 45s is a realistic upper bound.

**What breaks if changed?** Too low (e.g., 5000ms): Every test fails due to Salesforce load times. Too high (e.g., 120000ms): A genuinely broken test (infinite loop, hung network call) wastes 2 minutes before CI fails it.

---

```typescript
  testDir: "./src/tests",
```

**Where Playwright looks for test files.** Only files matching `**/*.spec.ts` inside this directory are treated as tests.

**What breaks if removed?** Playwright defaults to the project root and finds no `*.spec.ts` files, running zero tests.

---

```typescript
  fullyParallel: true,
```

**What:** Run test files in parallel (multiple workers). This is the default Playwright behavior for maximum speed.

**What breaks if removed?** Tests run sequentially. A 100-test suite that takes 5 minutes in parallel might take 40 minutes serially.

---

```typescript
  forbidOnly: !!process.env.CI,
```

**What:** `test.only` is a debugging tool to run just one test. If a developer forgets to remove it before pushing, only that one test runs in CI — the rest are silently skipped, giving false confidence.

`!!process.env.CI` evaluates to `true` when running in CI (GitHub Actions sets `CI=true` automatically). When `forbidOnly` is true, any `test.only` in the codebase fails the entire CI run with a clear error.

**This is industry standard practice.** Never commit `test.only`.

---

```typescript
  retries: 0,
```

**What:** How many times to retry a failed test. Set to 0 — no retries.

**Why 0 in this project?** Retries can mask underlying flakiness. A test that passes on retry-2 doesn't mean it passed — it means it's flaky. The project takes a zero-tolerance approach to flakiness.

**Alternative (commented out):** `retries: process.env.CI ? 2 : 0` — retry twice only in CI (where flakiness from network conditions is more common) but fail fast locally.

---

```typescript
  workers: process.env.CI ? 1 : undefined,
```

**What:** Number of parallel workers (browsers). `undefined` uses Playwright's default (half your CPU cores). `1` runs everything in a single process.

**Why 1 in CI?** CI runners (GitHub Actions free tier) have limited CPU. Running multiple browsers simultaneously can cause resource contention, timeouts, and flaky tests. 1 worker trades parallelism for reliability in CI. Locally, you keep full parallelism for speed.

---

```typescript
  reporter: "html",
```

**What:** The `html` reporter generates an interactive HTML report in `playwright-report/`. You can view it with `npx playwright show-report`.

**What breaks if removed?** You'd still see test results in the console, but no visual report.

---

```typescript
  use: {
    baseURL: "https://login.salesforce.com",
    trace: "on-first-retry",
    screenshot: "on",
    video: "on"
  },
```

**`baseURL`:** When your tests call `page.goto("/")`, Playwright prepends this URL, giving you `https://login.salesforce.com/`. Without this, you'd have to write the full URL in every `goto` call.

**`trace: "on-first-retry"`:** Traces record a full snapshot of every action — DOM state, network requests, console logs, screenshots. They're massive files. `on-first-retry` means: capture a trace only when a test fails AND is being retried. This avoids storing huge traces for every passing test.

**`screenshot: "on"`:** Take a screenshot after every test (pass or fail). This is very useful for debugging what the UI looked like when a test passed — sometimes a passing test was passing for the wrong reason.

**`video: "on"`:** Record a video of every test. Combined with traces, this is your complete audit trail. On CI, this is invaluable when a test fails and you can't reproduce it locally.

**What breaks if `screenshot` and `video` are removed?** You lose the ability to debug failures you can't reproduce. In Salesforce testing where the UI is dynamic and state-dependent, this can be the only way to understand what happened.

---

```typescript
  projects: [
    {
      name: "Google Chrome",
      use: { ...devices["Desktop Chrome"], channel: "chrome" },
    },
  ],
```

**What:** A project in Playwright is a browser/device configuration to run tests against. Your codebase runs only on Chrome (others are commented out).

`...devices["Desktop Chrome"]` — spreads Playwright's pre-defined Desktop Chrome settings (viewport: 1280x720, user agent, etc.).

`channel: "chrome"` — use the installed Google Chrome binary rather than Playwright's bundled Chromium. This is important for Salesforce: it uses Chrome-specific features and your tests should run against the same browser your users use.

**Multi-project uncommenting:** To run on Firefox too, uncomment the Firefox project. Playwright will run the entire test suite twice — once per project — in parallel.

---

# PART 6 — Test File Anatomy {#part-6}

> Picking `SerialTest.spec.ts` as the most representative — it uses the most patterns in one file.

```typescript
import { test, Page } from "@playwright/test";
import LoginPage from "../pages/LoginPage";
import logger from "../utils/LoggerUtil";
import { decrypt } from "../utils/CryptojsUtil";
import ContactPage from "../pages/ContactPage";
import testdata from "../testdata/contactCaseFlow.json";
import CasePage from "../pages/CasePage";
```

**Line-by-line:**

`import { test, Page }` — Named imports. `test` is the test runner function. `Page` is the TypeScript type for Playwright's Page object (needed for `let page: Page`).

`import LoginPage from "../pages/LoginPage"` — Default import of the POM class.

`import logger from "../utils/LoggerUtil"` — The singleton logger.

`import { decrypt }` — Only the decrypt function is needed; encrypt is not.

`import testdata from "../testdata/contactCaseFlow.json"` — Direct JSON import. TypeScript knows the shape because `resolveJsonModule: true` is in `tsconfig.json`. `testdata.contactFName` is fully typed.

---

```typescript
test.describe.configure({ mode: "serial" });
```

Applies serial mode to this entire file. All tests run in order, not in parallel.

---

```typescript
let page: Page;
```

Module-level variable. Declared with `let` (not `const`) because it's assigned in `beforeAll`. `Page` is the TypeScript type — if you try to assign anything other than a Playwright Page to it, TypeScript errors.

---

```typescript
test.beforeAll(async ({ browser }) => {
  page = await browser.newPage();
  const loginPage = new LoginPage(page);
  const homePage = await loginPage.quickLogin(
    decrypt(process.env.userid!),
    decrypt(process.env.password!)
  );
  await homePage.expectServiceTitleToBeVisible();
  logger.info("login is completed");
});
```

`async ({ browser })` — `beforeAll` hooks receive Playwright fixtures as arguments. `{ browser }` destructures the `browser` fixture from Playwright's built-in fixture set.

`browser.newPage()` — creates a BrowserContext and Page together.

`new LoginPage(page)` — instantiates the POM class, passing in `page`. The POM class doesn't create the page — it receives it.

`loginPage.quickLogin(...)` — convenience method that navigates, fills username, fills password, clicks login, and returns the `HomePage` POM object.

`decrypt(process.env.userid!)` — decrypts the userid env var. The `!` asserts it's not undefined.

`await homePage.expectServiceTitleToBeVisible()` — this is the assertion that confirms login succeeded. If this fails, the `beforeAll` fails, all tests are skipped.

`logger.info(...)` — structured log entry recording completion. This shows up in `test_run.log`.

---

```typescript
test("Create Contact and Open", async () => {
  const contactpage = new ContactPage(page);
  await contactpage.createNewContact(testdata.contactFName, testdata.contactLName);
  await contactpage.expectContactLabelContainsFirstNameAndLastName(
    testdata.contactFName, testdata.contactLName
  );
  await contactpage.findExistingContactByLastName(testdata.contactLName);
});
```

Notice: `async ()` — no `{ page }` fixture argument. This test uses the module-level `page` variable via closure. This is intentional — using the fixture would create a new page, breaking the shared session.

`new ContactPage(page)` — instantiates the ContactPage POM with the shared page.

`testdata.contactFName` — typed access to the JSON. TypeScript knows this field is a string.

---

```typescript
test.afterAll(async () => {
  await page.close();
});
```

Cleanup. `page.close()` closes the tab and the BrowserContext. Without this, Playwright would warn about open pages after the test suite finishes.

---

# PART 7 — Page Object Model Deep Dive {#part-7}

> Picking `ContactPage.ts` — it has the most varied locator strategies and clearly shows the full POM pattern.

## 7.1 — Why POM Exists

**The problem without POM:**

Imagine your test file doing this:
```typescript
await page.getByRole('link', { name: 'Contacts' }).click();
await page.getByRole('button', { name: 'New' }).click();
await page.getByPlaceholder('First Name').fill("Shiva");
await page.getByPlaceholder('Last Name').fill("Rudra");
await page.getByRole('button', { name: 'Save', exact: true }).click();
```

Now you have 8 test files. Each repeats this block. Salesforce changes "First Name" placeholder to "Given Name". You search-replace across 8 files, miss one, and get a 1-AM alert that 3 tests are failing in prod.

**With POM:**

```typescript
await contactsPage.createNewContact("Shiva", "Rudra");
```

One change in `ContactPage.ts` fixes every test that creates a contact.

---

## 7.2 — ContactPage Implementation Walkthrough

```typescript
export default class ContactPage {
  private readonly contactsLink = "Contacts";
  private readonly newButtonLocator = "New";
  private readonly firstNameTextFieldLocator = "First Name";
  private readonly lastNameTextFieldLocator = "Last Name";
  private readonly saveButtonLocator = "Save";
  private readonly searchBoxLocator = "Search this list...";
  private readonly contactFullNameLabelLocator = "sfa-output-name-with-hierarchy-icon-wrapper";
```

**All locator strings are private readonly class properties.** This is the most important design decision in a POM class.

- **Private:** Tests can't reference these directly — they must go through methods.
- **Readonly:** Locators don't change during a test run. `readonly` prevents accidental reassignment.
- **Named constants instead of inline strings:** `"sfa-output-name-with-hierarchy-icon-wrapper"` is a Salesforce-specific CSS class. If you put this string inline in multiple places, a change requires finding every occurrence. Here, you change `contactFullNameLabelLocator` once.

```typescript
constructor(private page: Page) {}
```

The POM doesn't create a page. It receives one. Why? Because the caller manages the page lifecycle, not the POM. This is dependency injection — a core enterprise pattern.

---

```typescript
async createNewContact(fname: string, lname: string) {
  await this.page.getByRole('link', { name: this.contactsLink }).click();
  await this.page.getByRole('button', { name: this.newButtonLocator }).click();
  logger.info("New button is clicked");
  await this.page.getByPlaceholder(this.firstNameTextFieldLocator).click();
  await this.page.getByPlaceholder(this.firstNameTextFieldLocator).fill(fname);
  logger.info(`First name is filled as ${fname}`)
  await this.page.getByPlaceholder(this.firstNameTextFieldLocator).press('Tab');
  await this.page.getByPlaceholder(this.lastNameTextFieldLocator).fill(lname);
  await this.page.getByRole('button', { name: this.saveButtonLocator, exact: true })
    .click()
    .catch((error) => {
      logger.error(`Error clicking Save button: ${error}`);
      throw error;
    })
    .then(() => logger.info("Save Button is clicked"));
}
```

**Method signature:** `async createNewContact(fname: string, lname: string)` — typed parameters prevent passing wrong types. Return type is inferred as `Promise<void>`.

**`.press('Tab')`** after filling first name — Salesforce requires a Tab keypress after the first name field to move focus and trigger field validation. This is a Salesforce-specific behavior that a POM encapsulates perfectly: the test doesn't need to know about this quirk.

**`.catch().then()` pattern on the Save button click:**
```typescript
.click()
.catch((error) => {
  logger.error(`Error clicking Save button: ${error}`);
  throw error; // re-throw so test fails
})
.then(() => logger.info("Save Button is clicked"));
```

This is Promise chaining. `.catch` handles the error case (logs it, then re-throws so the test still fails). `.then` handles the success case (logs it). The re-throw is critical — without it, the error is swallowed and the test would continue as if the click succeeded.

**Why `exact: true` on Save?** Salesforce has multiple buttons containing the word "Save" (e.g., "Save & New"). `exact: true` ensures you match only the button whose full text is exactly "Save".

---

```typescript
async expectContactLabelContainsFirstNameAndLastName(fname: string, lname: string) {
  await expect(this.page.locator(this.contactFullNameLabelLocator))
    .toContainText(`${fname} ${lname}`);
  logger.info(`New contact created and ${fname} ${lname} is visible`);
  await this.page.getByRole('link', { name: this.contactsLink }).click();
}
```

**Assertion in POM:** The assertion is inside the POM, not the test. This is a deliberate design choice — the "expect" methods (by convention named `expectXxx`) encapsulate the assertion logic. Tests read like behavior: "expect contact label contains first name and last name."

**Template literal:** `` `${fname} ${lname}` `` — TypeScript/JS string interpolation. Equivalent to `fname + " " + lname` but more readable.

---

```typescript
async findExistingContactByLastName(lname: string) {
  await this.page.getByRole("link", { name: this.contactsLink }).click();
  await this.page.getByPlaceholder(this.searchBoxLocator).click();
  await this.page.getByPlaceholder(this.searchBoxLocator).fill(lname);
  await this.page.getByPlaceholder(this.searchBoxLocator).press("Enter");
  await this.page.getByRole("link", { name: lname }).click();
}
```

Search → type last name → press Enter → click the result link. Pure encapsulation. The test just calls `findExistingContactByLastName("Rudra")` — it doesn't know how Salesforce search works internally.

---

## 7.3 — How Tests Consume POM

The pattern is always:

```typescript
// 1. Instantiate POM with the shared page
const contactpage = new ContactPage(page);

// 2. Call methods — read them like English sentences
await contactpage.createNewContact(testdata.contactFName, testdata.contactLName);
await contactpage.expectContactLabelContainsFirstNameAndLastName(...);
await contactpage.findExistingContactByLastName(testdata.contactLName);
```

Test files become readable specifications of behavior, not implementation details.

---

# PART 8 — Data & Configuration Management {#part-8}

## 8.1 — Environment Files

**Three environments, three files:**

```
src/config/.env         # Default (dev/local)
src/config/.env.qa      # QA environment
src/config/.env.uat     # UAT environment
```

**Switching environments:**

```bash
# Default (loads .env)
npx playwright test

# QA environment (loads .env.qa)
NODE_ENV=qa npx playwright test

# UAT environment (loads .env.uat)
NODE_ENV=uat npx playwright test
```

**Structure of .env files:**
```
userid=U2FsdGVkX1...    # AES-encrypted username
password=U2FsdGVkX2...  # AES-encrypted password
SALT=your-secret-salt   # Encryption key (or set as real env var)
```

**The config loading logic in `playwright.config.ts`:**

```typescript
if (!process.env.NODE_ENV) {
  require("dotenv").config({ path: `${__dirname}//src//config//.env` });
} else {
  require("dotenv").config({
    path: `${__dirname}//src//config//.env.${process.env.NODE_ENV}`,
  });
}
```

Simple, readable, effective. `__dirname` is Node.js's "absolute path to the folder this file is in" — ensures paths work regardless of where the command is run from.

---

## 8.2 — Test Data Approaches

Your project uses four data strategies:

### 1. Static JSON
```typescript
// contactCaseFlow.json
{
  "contactFName": "demo-011-fn",
  "contactLName": "demo-011-ln",
  "caseOrigin": "Phone"
}

// In SerialTest.spec.ts
import testdata from "../testdata/contactCaseFlow.json";
await contactpage.createNewContact(testdata.contactFName, testdata.contactLName);
```

**Use when:** Single-run workflow tests with fixed, specific data. The data is version-controlled — you always know what state you're testing.

### 2. JSON Array (Data-Driven)
```typescript
// datademo.json — array of contacts
[{ "firstName": "Henry", "lastName": "Ford" }, ...]

// In contactTest.spec.ts
for (const contact of cdata) {
  test.skip(`Advance DD test for ${contact.firstName}`, async ({ page }) => {
    // one test per contact
  });
}
```

**Use when:** You need to test the same workflow with multiple data variations. Each variation becomes its own test case with its own name in the report.

### 3. CSV (via CsvtoJsonUtil)
```typescript
convertCsvFileToJsonFile("data.csv", "datademo.json");
```

**Use when:** Test data is maintained in Excel/Sheets by non-developers (business analysts, PMs). They export to CSV, you convert to JSON. Bridges the gap between business data and automation.

### 4. Faker (Synthetic Data Generation)
```typescript
const testData = generateTestData(20);
exportToJson(testData, 'testData_en.json');
exportToCsv(testData, 'testData_en.csv');
```

**Use when:** You need realistic but unique data for each run, especially to avoid Salesforce duplicate record warnings. Faker generates 20 unique users with valid-looking names, emails, phone numbers.

---

## 8.3 — Secrets Management

| Layer | What's stored | How secured |
|---|---|---|
| `.env` files | Encrypted credentials | AES encryption; file in `.gitignore` |
| `SALT` | Encryption key | System env var or CI secret; never in files |
| `auth.json` | Session cookies | Generated at runtime; in `.gitignore` |
| CI secrets | `SALT`, `NODE_ENV`, any overrides | GitHub Actions encrypted secrets |

**The flow:**
1. Developer sets `SALT` as a system env var on their machine
2. Runs `encryptEnvFile()` to encrypt credentials in `.env`
3. `.env` is safe to store in the repo (encrypted values)
4. CI machine has `SALT` as a GitHub Actions secret
5. At test time, `dotenv` loads `.env`, `decrypt()` recovers plaintext, tests run

---

# PART 9 — Running & Reporting {#part-9}

## 9.1 — Running Tests

```bash
# Run all tests
npx playwright test

# Run a specific file
npx playwright test src/tests/loginTest.spec.ts

# Run tests matching a keyword in the name
npx playwright test --grep "login"

# Run in headed mode (see the browser)
npx playwright test --headed

# Run with a specific number of workers
npx playwright test --workers=2

# Run a specific project (browser)
npx playwright test --project="Google Chrome"

# Run in debug mode (Playwright Inspector opens)
npx playwright test --debug

# Show the HTML report after a run
npx playwright show-report

# Run for a specific environment
NODE_ENV=qa npx playwright test
```

## 9.2 — Reading the HTML Report

After `npx playwright test` runs, open `playwright-report/index.html` in your browser (or `npx playwright show-report`).

You'll see:
- **Pass/Fail/Skip counts** at the top
- Each test listed with its status
- Click a failed test → see the error message, screenshots, and video
- Click "Trace" (if trace was captured) → full timeline of every action

The trace viewer shows:
- DOM snapshots at each action
- Network requests
- Console logs
- Time taken per action

This is your single most powerful debugging tool for CI failures.

## 9.3 — GitHub Actions CI/CD

**From `.github/workflows/main.yml`:**

```yaml
# The workflow triggers on push to main or on pull requests
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci             # Install dependencies (uses package-lock.json exactly)
      - run: npx playwright install --with-deps  # Install browsers + OS deps
      - run: npx playwright test
    env:
      SALT: ${{ secrets.SALT }}  # Injected from GitHub secrets
      NODE_ENV: qa               # Or from a matrix/variable
```

**Key points:**
- `npm ci` (not `npm install`) — deterministic install from `package-lock.json`. CI environments must be reproducible.
- `npx playwright install --with-deps` — installs OS-level dependencies (libglib, libnss, etc.) that Chrome needs on Ubuntu
- `CI=true` is set automatically by GitHub Actions — this activates `forbidOnly` and sets `workers: 1` in your config
- `SALT` from GitHub secrets — never appears in logs, never in code

---

# PART 10 — Beginner Mental Models {#part-10}

## 10.1 — Five Mental Models That Make Everything Click

### 1. "Tests are user stories with assertions"
Every test describes a scenario a user would do: "User navigates to login, enters credentials, lands on home page." Playwright's API is designed to feel like describing user actions, not DOM manipulation. When you struggle with "how do I do X in Playwright," ask "how would a user do X?" — that usually points to the right API.

### 2. "POM is a vocabulary, tests are sentences"
Your POM classes define the vocabulary of what's possible on each page. Your test files write sentences using that vocabulary. `await contactpage.createNewContact("Henry", "Ford")` is a sentence. `await page.getByRole('link', {name: 'Contacts'}).click()` is raw vocabulary spilling into the test. Sentences are better.

### 3. "Every `await` is a checkpoint"
`await` doesn't just wait — it's Playwright checking: "did the action work?" If the element isn't there when you `await`, Playwright retries until the timeout. Think of each `await` as Playwright saying "I won't move to the next line until this is confirmed done."

### 4. "The fixture system is a DI container for tests"
Dependency injection (DI) is the pattern where objects receive their dependencies instead of creating them. In your fixture: the test doesn't create `homePage` — the fixture creates it and injects it. This decouples tests from setup logic. Sound familiar from C# DI? Same idea, different syntax.

### 5. "Config is the environment contract"
`playwright.config.ts` is the contract between your framework and the environments it runs in. Every property answers: "what's the correct behavior in this context?" Changing a config property changes what the framework guarantees. Treat it like a legal document — intentional, documented, reviewed.

---

## 10.2 — Three Most Common Beginner Mistakes (And How This Codebase Avoids Them)

### Mistake 1: Hardcoding selectors inline in tests

**What beginners do:**
```typescript
// In test file
await page.locator("#username").fill("john@example.com");
```

**What this codebase does:** All selectors are in POM classes as `private readonly` properties. Tests never see raw selectors.

**Why it matters:** Salesforce changes DOM structure regularly. One refactor from your project team can break 20 tests. POM means one fix heals everything.

### Mistake 2: No separation between test data and test logic

**What beginners do:**
```typescript
test("create contact", async ({ page }) => {
  await contactPage.createNewContact("John", "Doe"); // hardcoded data
});
```

**What this codebase does:** Data lives in `src/testdata/` as JSON files, loaded via imports. Fixture-based data via Faker generates unique data per run.

**Why it matters:** When a test fails, you need to know "was it a bug in the test, or was it the data?" Separation makes this clear. Also, hardcoded names cause duplicate record conflicts.

### Mistake 3: Shared mutable state across independent tests

**What beginners do:**
```typescript
let loggedIn = false;
test("test 1", async ({ page }) => { loggedIn = true; });
test("test 2", async ({ page }) => { if (loggedIn) { ... } }); // depends on test 1
```

**What this codebase does:** Independent tests each get a fresh `page` via Playwright fixtures. No shared state. When shared state IS needed (serial tests), it's explicit — `test.describe.configure({ mode: "serial" })` and module-level `let page: Page` with `beforeAll`.

**Why it matters:** Parallel execution randomizes test order. Tests with hidden dependencies become intermittently failing — the hardest class of bug to diagnose.

---

## 10.3 — Recommended Learning Order for Going Deeper

1. **TypeScript Fundamentals** — Complete the official TypeScript Handbook (typescriptlang.org/docs/handbook). Focus on: interfaces, generics, access modifiers, async/await. Budget: 1 week.

2. **Playwright Docs — Core Concepts** — Read: Getting Started, Writing Tests, Locators, Assertions, and Fixtures. Don't just read — build alongside. Budget: 1 week.

3. **Rebuild This Project From Scratch** — Seriously. Open a blank folder. Re-implement `LoginPage.ts`, `loginTest.spec.ts`, and `playwright.config.ts` without looking. This forces you to understand rather than recognize. Budget: 1–2 days.

4. **Add Features** — Pick one requirement from `requirements.md` that isn't fully implemented and implement it. Good candidates: implement the retry mechanism in config, add a second environment, add a GitHub Actions badge to the README.

5. **Playwright Advanced Topics** — Once the basics are solid, explore: Auth state reuse in global setup, Component testing, Playwright API as a testing strategy (skip the UI layer for setup/teardown).

6. **Real Playwright Projects on GitHub** — Search GitHub for `playwright-salesforce` or `playwright-enterprise`. Reading other people's frameworks teaches idioms and patterns that docs don't cover.

---

> Bob's final note to ktx: "You've got a genuinely solid framework here. It solves real enterprise problems — encryption, self-healing, serial workflows, data-driven testing, API mocking — not just 'here's how to click a button.' The next level is making every test independently runnable (fixing the serial dependency issue), adding proper global setup/teardown for auth state, and wiring up allure reporter for richer CI reports. But the foundation is there, and now you understand why every decision was made the way it was."

---

*Tutorial generated by Bob — Playwright + TypeScript Expert Mentor. For ktx, Automation QA Engineer.*
