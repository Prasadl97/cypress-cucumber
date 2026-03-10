# Playwright + Cucumber (E-Commerce Web Application)

E2E tests using [Playwright](https://playwright.dev/) for browser automation and [Cucumber](https://cucumber.io/) for BDD (Gherkin) scenarios.

**Application under test:** [Magento 2 Demo](https://magento2-demo.magebit.com/)

## Setup

```bash
cd playwright-cucumber
npm install
npx playwright install
```

## Run tests

```bash
# Headless (default: Chromium)
npm test

# With browser visible
npm run test:headed

# With browser visible and slowed down (e.g. 500ms per action)
npm run test:ui

# Cross-browser: run on Firefox or WebKit
BROWSER=firefox npm test
BROWSER=webkit npm test

# Or use convenience scripts (requires cross-env)
npm run test:firefox
npm run test:webkit
npm run test:all-browsers   # runs full suite on Chromium, then Firefox, then WebKit
```

## Project layout

```
playwright-cucumber/
├── src/
│   ├── features/           # Gherkin feature files
│   │   ├── user-registration-login.feature
│   │   ├── admin-login.feature
│   │   └── product-cart.feature
│   ├── step-definitions/   # Step definitions
│   │   ├── user-registration-login.steps.ts
│   │   ├── admin-login.steps.ts
│   │   └── product-cart.steps.ts
│   ├── pages/              # Page objects (PascalCase)
│   │   ├── BasePage.ts
│   │   ├── LoginPage.ts
│   │   ├── HomePage.ts
│   │   ├── AccountDashboardPage.ts
│   │   ├── ProductPage.ts
│   │   ├── CartPage.ts
│   │   ├── CreateAccountPage.ts
│   │   ├── AdminLoginPage.ts
│   │   └── AdminOrdersPage.ts
│   ├── hooks/
│   │   └── hooks.ts        # Before/After (init & close browser)
│   ├── support/
│   │   └── world.ts        # Custom World with browser, context, page
│   └── utils/
│       ├── id-utils.ts     # Dynamic email generation
│       └── entity-store.ts # Persisted user JSON
├── configs/
│   └── testData.ts        # loadTestData from JSON
├── data/
│   ├── test-data.json      # External test data
│   └── entities/           # Persisted test data (e.g. registered-user.json)
├── reports/                # Test reports output
├── cucumber.js             # Cucumber config (paths, formatter, world params)
├── cucumber.mjs            # ESM Cucumber config (same as cucumber.js)
├── playwright.config.ts    # Playwright config
├── package.json
├── tsconfig.json
└── README.md
```

## Writing scenarios

1. Add or edit `.feature` files under `src/features/`.
2. Implement steps in `src/step-definitions/` using `this.page` and page objects from `src/pages/`.
3. Use `this.parameters.baseURL` and `this.parameters.entityFilePath` for the app and persisted user file.

## Environment

- **BASE_URL** – Override app URL (default: `https://magento2-demo.magebit.com/`).
- **ADMIN_BASE_URL** – Override admin portal URL (default: `{BASE_URL}/admin/`). Use when admin is on a different host or path.
- **BROWSER** – Browser for test run: `chromium` (default), `firefox`, or `webkit`. Enables cross-browser execution.
- **ENTITY_FILE_PATH** – Override path for saved registered user JSON (default: `./data/entities/registered-user.json`).
- **DATA_FILE_PATH** – Override path for test data JSON (default: `data/test-data.json`).

Config is read from `process.env` in `cucumber.js` / `cucumber.mjs`; override via env or world parameters.

## Run by tag

```bash
node --import tsx node_modules/@cucumber/cucumber/bin/cucumber-js --tags "@smoke"
```

## Notes

- Tests run in **Chromium** by default; set `BROWSER=firefox` or `BROWSER=webkit` for cross-browser execution. Install all browsers with `npx playwright install`.
- Registration flow saves email/password to the entity file after success; the login step reads from it.
- `playwright.config.ts` is for optional Playwright-native runs; Cucumber uses the custom World and `playwright` inside step definitions.

## Technical requirements

| Requirement | Implementation |
|-------------|----------------|
| **Page Object Model** | `src/pages/`: `BasePage` plus `LoginPage`, `HomePage`, `AccountDashboardPage`, etc. Selectors and actions encapsulated; World exposes page instances. |
| **Async/await correctness** | All hooks, steps, and page methods are `async` and use `await`; no `.then()` or sync blocking. |
| **Environment configuration** | `BASE_URL`, `ADMIN_BASE_URL`, `ENTITY_FILE_PATH` (and `DATA_FILE_PATH`) read from `process.env` in `cucumber.js` with fallbacks. |
| **External test data** | `data/test-data.json` for default registration; `configs/testData.ts` loads it. Entity store for persisted user (`data/entities/registered-user.json`). |
| **Hooks** | `src/hooks/hooks.ts`: `Before` (init browser/context/page), `After` (destroy). |
| **No hardcoded waits** | Only Playwright auto-waiting (`waitFor({ state: 'visible' })`, `fill`, `click`); no `waitForTimeout` or `setTimeout`. |
| **Clean reusable code** | World exposes page getters; shared base URL and test data loader; no duplicated page construction in steps. |
