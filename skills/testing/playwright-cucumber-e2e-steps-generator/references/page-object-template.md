# Page Object Template

## Base Structure

All Page Objects extend `BasePage` from `src/page_objects/base_page.ts`.

```typescript
import { Page } from 'playwright';
import { BasePage } from './base_page';

export class ExamplePage extends BasePage {
    // Selectors as private readonly constants
    private readonly selectors = {
        usernameInput: '[data-testid="username"]',
        passwordInput: '[data-testid="password"]',
        submitButton: 'button[type="submit"]',
        errorMessage: '.error-message',
    };

    constructor(page: Page) {
        super(page);
    }

    // Navigation method
    async navigate(): Promise<void> {
        await this.goto('/example-path');
        await this.waitForPageLoad();
    }

    // Action methods - verb-based naming
    async enterUsername(username: string): Promise<void> {
        await this.fill(this.selectors.usernameInput, username);
    }

    async enterPassword(password: string): Promise<void> {
        await this.fill(this.selectors.passwordInput, password);
    }

    async clickSubmit(): Promise<void> {
        await this.click(this.selectors.submitButton);
    }

    // Compound actions
    async login(username: string, password: string): Promise<void> {
        await this.enterUsername(username);
        await this.enterPassword(password);
        await this.clickSubmit();
    }

    // Getter methods for assertions
    async getErrorMessage(): Promise<string> {
        return this.getText(this.selectors.errorMessage);
    }

    async isErrorVisible(): Promise<boolean> {
        return this.isVisible(this.selectors.errorMessage);
    }

    // Assertion methods
    async expectLoginSuccess(): Promise<void> {
        await this.expectUrl(/\/dashboard/);
    }

    async expectErrorMessage(message: string): Promise<void> {
        await this.expectText(this.selectors.errorMessage, message);
    }
}
```

## Selector Strategies (Priority Order)

1. **data-testid** (preferred): `[data-testid="submit-btn"]`
2. **data-* attributes**: `[data-action="save"]`
3. **aria-label**: `[aria-label="Close dialog"]`
4. **role + text**: `role=button[name="Submit"]`
5. **CSS selectors**: `.btn-primary`, `#login-form`
6. **XPath** (last resort): `//button[contains(text(), "Submit")]`

## Complex Page Example

```typescript
import { Page } from 'playwright';
import { BasePage } from './base_page';

export class DashboardPage extends BasePage {
    private readonly selectors = {
        // Header section
        header: {
            logo: '[data-testid="header-logo"]',
            userMenu: '[data-testid="user-menu"]',
            logoutButton: '[data-testid="logout-btn"]',
        },
        // Sidebar section
        sidebar: {
            menuItem: (name: string) => `[data-testid="menu-${name}"]`,
            expandButton: '[data-testid="sidebar-expand"]',
        },
        // Main content
        content: {
            title: 'h1.page-title',
            loadingSpinner: '.loading-spinner',
            dataTable: '[data-testid="data-table"]',
            tableRow: (index: number) => `[data-testid="table-row-${index}"]`,
        },
        // Modals
        modal: {
            container: '[data-testid="modal"]',
            closeButton: '[data-testid="modal-close"]',
            confirmButton: '[data-testid="modal-confirm"]',
        },
    };

    constructor(page: Page) {
        super(page);
    }

    // Wait for page to be fully loaded
    async waitForDashboardLoad(): Promise<void> {
        await this.waitForSelector(this.selectors.header.logo);
        await this.page.waitForSelector(this.selectors.content.loadingSpinner, { 
            state: 'hidden',
            timeout: 30000 
        });
    }

    // Dynamic selector usage
    async clickMenuItem(menuName: string): Promise<void> {
        await this.click(this.selectors.sidebar.menuItem(menuName));
    }

    async getTableRowData(rowIndex: number): Promise<string> {
        return this.getText(this.selectors.content.tableRow(rowIndex));
    }

    // Modal interactions
    async confirmModal(): Promise<void> {
        await this.waitForSelector(this.selectors.modal.container);
        await this.click(this.selectors.modal.confirmButton);
        await this.page.waitForSelector(this.selectors.modal.container, { 
            state: 'hidden' 
        });
    }

    // User menu actions
    async logout(): Promise<void> {
        await this.click(this.selectors.header.userMenu);
        await this.click(this.selectors.header.logoutButton);
    }
}
```

## Export Pattern

In `src/page_objects/pages.ts`:

```typescript
export { BasePage } from './base_page';
export { LoginPage } from './login_page';
export { DashboardPage } from './dashboard_page';
export { CustomersPage } from './customers_page';
// Add new pages here
```
