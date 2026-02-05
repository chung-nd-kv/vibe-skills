# Common Steps Reference

Reusable step patterns for common testing scenarios.

## Authentication Steps

```typescript
// src/step_definitions/web/auth_steps.ts
import { Given, When, Then } from '@cucumber/cucumber';
import { CustomWorld } from '@support/world';
import { LoginPage } from '@page_objects/login_page';
import { getAccountCredentials } from '@support/fixture/account';

let loginPage: LoginPage;

Given('I am logged in as {string} user', async function (this: CustomWorld, userType: string) {
    const accounts = getAccountCredentials();
    const account = accounts['KiotVietTest'];
    const { username, password } = account.users[userType];
    
    loginPage = new LoginPage(this.page);
    await loginPage.navigate();
    await loginPage.login(username, password);
});

Given('I am logged in to {string} as {string}', async function (
    this: CustomWorld, 
    tenant: string,
    userType: string
) {
    const accounts = getAccountCredentials();
    const account = accounts[tenant];
    const { username, password } = account.users[userType];
    
    loginPage = new LoginPage(this.page);
    await loginPage.navigateToTenant(account.merchantCode);
    await loginPage.login(username, password);
});

When('I logout', async function (this: CustomWorld) {
    await this.page.click('[data-testid="user-menu"]');
    await this.page.click('[data-testid="logout-btn"]');
});
```

## Navigation Steps

```typescript
// src/step_definitions/web/common_steps.ts
import { Given, When, Then } from '@cucumber/cucumber';
import { CustomWorld } from '@support/world';
import config from '@config/index';

Given('I am on the {string} page', async function (this: CustomWorld, pageName: string) {
    const pageUrls: Record<string, string> = {
        'dashboard': '/dashboard',
        'customers': '/customers',
        'products': '/products',
        'orders': '/orders',
        'settings': '/settings',
    };
    
    const path = pageUrls[pageName.toLowerCase()];
    if (!path) {
        throw new Error(`Unknown page: ${pageName}`);
    }
    
    await this.page.goto(`${config.web.baseUrl}${path}`);
    await this.page.waitForLoadState('networkidle');
});

When('I navigate to {string}', async function (this: CustomWorld, path: string) {
    await this.page.goto(`${config.web.baseUrl}${path}`);
});

When('I click the {string} menu item', async function (this: CustomWorld, menuName: string) {
    await this.page.click(`[data-testid="menu-${menuName.toLowerCase()}"]`);
});

When('I go back', async function (this: CustomWorld) {
    await this.page.goBack();
});

When('I refresh the page', async function (this: CustomWorld) {
    await this.page.reload();
    await this.page.waitForLoadState('networkidle');
});
```

## Form Interaction Steps

```typescript
When('I fill in {string} with {string}', async function (
    this: CustomWorld, 
    fieldName: string, 
    value: string
) {
    await this.page.fill(`[data-testid="${fieldName}"]`, value);
});

When('I clear the {string} field', async function (this: CustomWorld, fieldName: string) {
    await this.page.fill(`[data-testid="${fieldName}"]`, '');
});

When('I select {string} from {string} dropdown', async function (
    this: CustomWorld, 
    value: string, 
    dropdownName: string
) {
    await this.page.click(`[data-testid="${dropdownName}"]`);
    await this.page.click(`[data-testid="option-${value}"]`);
});

When('I check the {string} checkbox', async function (this: CustomWorld, checkboxName: string) {
    await this.page.check(`[data-testid="${checkboxName}"]`);
});

When('I uncheck the {string} checkbox', async function (this: CustomWorld, checkboxName: string) {
    await this.page.uncheck(`[data-testid="${checkboxName}"]`);
});

When('I upload file {string} to {string}', async function (
    this: CustomWorld, 
    filename: string, 
    inputName: string
) {
    const fileInput = this.page.locator(`[data-testid="${inputName}"]`);
    await fileInput.setInputFiles(`test-data/${filename}`);
});
```

## Button/Click Steps

```typescript
When('I click the {string} button', async function (this: CustomWorld, buttonName: string) {
    await this.page.click(`button:has-text("${buttonName}")`);
});

When('I click on {string}', async function (this: CustomWorld, text: string) {
    await this.page.click(`text=${text}`);
});

When('I double click on {string}', async function (this: CustomWorld, text: string) {
    await this.page.dblclick(`text=${text}`);
});

When('I right click on {string}', async function (this: CustomWorld, selector: string) {
    await this.page.click(selector, { button: 'right' });
});
```

## Assertion Steps

```typescript
Then('I should see {string}', async function (this: CustomWorld, text: string) {
    await expect(this.page.locator(`text=${text}`)).toBeVisible();
});

Then('I should not see {string}', async function (this: CustomWorld, text: string) {
    await expect(this.page.locator(`text=${text}`)).not.toBeVisible();
});

Then('the {string} field should contain {string}', async function (
    this: CustomWorld, 
    fieldName: string, 
    value: string
) {
    const input = this.page.locator(`[data-testid="${fieldName}"]`);
    await expect(input).toHaveValue(value);
});

Then('the {string} should be disabled', async function (this: CustomWorld, elementName: string) {
    await expect(this.page.locator(`[data-testid="${elementName}"]`)).toBeDisabled();
});

Then('the {string} should be enabled', async function (this: CustomWorld, elementName: string) {
    await expect(this.page.locator(`[data-testid="${elementName}"]`)).toBeEnabled();
});

Then('I should be on the {string} page', async function (this: CustomWorld, pageName: string) {
    await expect(this.page).toHaveURL(new RegExp(pageName.toLowerCase()));
});

Then('the page title should be {string}', async function (this: CustomWorld, title: string) {
    await expect(this.page).toHaveTitle(title);
});
```

## Wait Steps

```typescript
When('I wait for {int} seconds', async function (this: CustomWorld, seconds: number) {
    await this.page.waitForTimeout(seconds * 1000);
});

When('I wait for the page to load', async function (this: CustomWorld) {
    await this.page.waitForLoadState('networkidle');
});

When('I wait for {string} to be visible', async function (this: CustomWorld, selector: string) {
    await this.page.waitForSelector(selector, { state: 'visible' });
});

When('I wait for {string} to disappear', async function (this: CustomWorld, selector: string) {
    await this.page.waitForSelector(selector, { state: 'hidden' });
});
```

## Table/List Steps

```typescript
Then('the table should have {int} rows', async function (this: CustomWorld, count: number) {
    const rows = await this.page.locator('table tbody tr').count();
    expect(rows).toBe(count);
});

Then('row {int} should contain {string}', async function (
    this: CustomWorld, 
    rowIndex: number, 
    text: string
) {
    const row = this.page.locator(`table tbody tr:nth-child(${rowIndex})`);
    await expect(row).toContainText(text);
});

When('I click on row {int} in the table', async function (this: CustomWorld, rowIndex: number) {
    await this.page.click(`table tbody tr:nth-child(${rowIndex})`);
});
```

## Modal/Dialog Steps

```typescript
When('I confirm the dialog', async function (this: CustomWorld) {
    await this.page.click('[data-testid="modal-confirm"]');
});

When('I cancel the dialog', async function (this: CustomWorld) {
    await this.page.click('[data-testid="modal-cancel"]');
});

When('I close the modal', async function (this: CustomWorld) {
    await this.page.click('[data-testid="modal-close"]');
});

Then('I should see a modal with title {string}', async function (
    this: CustomWorld, 
    title: string
) {
    await expect(this.page.locator('[data-testid="modal-title"]')).toHaveText(title);
});
```

## Toast/Notification Steps

```typescript
Then('I should see a success message {string}', async function (
    this: CustomWorld, 
    message: string
) {
    await expect(this.page.locator('.toast-success')).toContainText(message);
});

Then('I should see an error message {string}', async function (
    this: CustomWorld, 
    message: string
) {
    await expect(this.page.locator('.toast-error')).toContainText(message);
});

Then('I should see a warning message', async function (this: CustomWorld) {
    await expect(this.page.locator('.toast-warning')).toBeVisible();
});
```
