# Cypress + Cucumber (E-Commerce Web Application)

E2E tests using [Cypress](https://www.cypress.io/) for browser automation and [Cucumber](https://cucumber.io/) (via [@badeball/cypress-cucumber-preprocessor](https://github.com/badeball/cypress-cucumber-preprocessor)) for BDD (Gherkin) scenarios.

**Application under test:** [Magento 2 Demo](https://magento2-demo.magebit.com/)

## Setup

```bash
npm install
```

## Run tests

```bash
# Headless (default)
npm test

# With browser visible
npm run test:headed

# Interactive mode
npm run test:open

# Run by tags
npm run test:smoke       # @smoke
npm run test:admin       # @admin
npm run test:cart        # @cart
npm run test:registration # @registration
```

## Project layout

```
cypress-cucumber/
├── cypress/
│   ├── e2e/
│   │   └── features/           # Gherkin feature files
│   │       ├── user-registration-login.feature
│   │       ├── admin-login.feature
│   │       └── product-cart.feature
│   ├── step-definitions/       # Step definitions (separate from pages)
│   │   ├── hooks.ts
│   │   ├── user-registration-login.steps.ts
│   │   ├── admin-login.steps.ts
│   │   └── product-cart.steps.ts
│   ├── support/
│   │   ├── commands.ts         # Custom Cypress commands
│   │   ├── context.ts          # Shared scenario context
│   │   └── e2e.ts
│   └── components/             # Reusable UI components
├── pages/                      # Page Object Model
│   ├── BasePage.ts
│   ├── HomePage.ts
│   ├── LoginPage.ts
│   ├── CreateAccountPage.ts
│   ├── AccountDashboardPage.ts
│   ├── ProductPage.ts
│   ├── CartPage.ts
│   ├── AdminLoginPage.ts
│   └── AdminOrdersPage.ts
│       ├── HeaderComponent.ts
│       └── FormComponent.ts
├── configs/
│   ├── TestDataLoader.ts        # Test data loader
│   └── TestDataSchema.ts        # Test data type definitions
├── data/
│   ├── test-data.json          # External test data (JSON)
│   └── entities/                # Persisted entities (e.g. registered-user.json)
├── utils/
│   ├── AssertUtils.ts          # Soft assertions
│   ├── EntityStore.cjs         # Persist/load user JSON
│   ├── IdUtils.ts              # Dynamic email generation
│   └── Logger.ts               # Logging utility
├── .cypress-cucumber-preprocessorrc.json
├── cypress.config.cjs          # CommonJS config (avoids ts-node parsing issues in CI)
├── package.json
├── tsconfig.json
└── README.md
```

## Architecture

- **Page Object Model**: Pages encapsulate locators and actions; no step logic in pages
- **Step definitions**: Orchestrate only—load data, call page methods, assertions
- **Reusable components**: HeaderComponent, FormComponent for shared UI patterns
- **External test data**: `data/test-data.json` for static test data
- **Dynamic data**: `IdUtils.generateUniqueEmail()` for unique registration emails

## Environment

- **BASE_URL** – App URL (default: `https://magento2-demo.magebit.com/`)
- **ADMIN_BASE_URL** – Admin portal URL (default: `{BASE_URL}/admin/`)
- **ENTITY_FILE_PATH** – Path for saved registered user JSON (default: `./data/entities/registered-user.json`)
- **DEBUG** – Enable verbose logging when set

## CI (GitHub Actions)

The `.github/workflows/e2e.yml` workflow runs tests on push/PR to main/master. Artifacts (videos, screenshots) are uploaded on failure.

## Notes

- Registration flow saves email/password to the entity file after success; the login step reads from it via `cy.task`
- Step definitions use `cy.then()` for async assertions to comply with Cypress's command chain
- Custom commands: `cy.login()`, `cy.assertVisible()`, `cy.fillForm()`
