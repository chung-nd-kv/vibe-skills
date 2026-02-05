# Step Definition Template

## Basic Structure

```typescript
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '@support/world';
import { ExamplePage } from '@page_objects/example_page';

// Initialize page object
let examplePage: ExamplePage;

Given('I am on the example page', async function (this: CustomWorld) {
    examplePage = new ExamplePage(this.page);
    await examplePage.navigate();
});

When('I enter {string} in the username field', async function (this: CustomWorld, username: string) {
    await examplePage.enterUsername(username);
});

Then('I should see the dashboard', async function (this: CustomWorld) {
    await examplePage.expectLoginSuccess();
});
```

## Step Patterns with Parameters

### String Parameters
```typescript
When('I enter {string} in the {string} field', async function (
    this: CustomWorld, 
    value: string, 
    fieldName: string
) {
    await this.page.fill(`[data-testid="${fieldName}"]`, value);
});
```

### Integer Parameters
```typescript
Then('I should see {int} items in the list', async function (
    this: CustomWorld, 
    count: number
) {
    const items = await this.page.locator('.list-item').count();
    expect(items).toBe(count);
});
```

### Optional Parameters
```typescript
When('I click the {string} button( again)', async function (
    this: CustomWorld, 
    buttonName: string
) {
    await this.page.click(`button:has-text("${buttonName}")`);
});
```

### Data Tables
```typescript
Given('the following users exist:', async function (
    this: CustomWorld, 
    dataTable: DataTable
) {
    const users = dataTable.hashes();
    // users = [{ name: 'John', role: 'admin' }, { name: 'Jane', role: 'user' }]
    for (const user of users) {
        await createUser(user.name, user.role);
    }
});
```

### Doc Strings
```typescript
When('I enter the following text:', async function (
    this: CustomWorld, 
    docString: string
) {
    await this.page.fill('textarea', docString);
});
```

## Complete Example File

```typescript
/**
 * Step definitions for Login feature
 * Feature: features/auth/login.feature
 */
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '@support/world';
import { LoginPage } from '@page_objects/login_page';
import { DashboardPage } from '@page_objects/dashboard_page';
import { getAccountCredentials } from '@support/fixture/account';
import config from '@config/index';

let loginPage: LoginPage;
let dashboardPage: DashboardPage;

// Background steps
Given('I am on the login page', async function (this: CustomWorld) {
    loginPage = new LoginPage(this.page);
    await loginPage.navigate();
});

Given('I am logged in as {string}', async function (this: CustomWorld, userType: string) {
    const accounts = getAccountCredentials();
    const { username, password } = accounts['KiotVietTest'].users[userType];
    
    loginPage = new LoginPage(this.page);
    await loginPage.navigate();
    await loginPage.login(username, password);
    
    dashboardPage = new DashboardPage(this.page);
    await dashboardPage.waitForDashboardLoad();
});

// When steps
When('I enter username {string}', async function (this: CustomWorld, username: string) {
    await loginPage.enterUsername(username);
});

When('I enter password {string}', async function (this: CustomWorld, password: string) {
    await loginPage.enterPassword(password);
});

When('I click the login button', async function (this: CustomWorld) {
    await loginPage.clickLoginButton();
});

When('I login with valid credentials', async function (this: CustomWorld) {
    const accounts = getAccountCredentials();
    const { username, password } = accounts['KiotVietTest'].users.admin;
    await loginPage.login(username, password);
});

// Then steps
Then('I should be redirected to the dashboard', async function (this: CustomWorld) {
    dashboardPage = new DashboardPage(this.page);
    await dashboardPage.waitForDashboardLoad();
    await dashboardPage.expectUrl(/\/dashboard/);
});

Then('I should see error message {string}', async function (this: CustomWorld, message: string) {
    await loginPage.expectErrorMessage(message);
});

Then('I should remain on the login page', async function (this: CustomWorld) {
    await loginPage.expectUrl(/\/login/);
});
```

## Hooks Integration

```typescript
import { Before, After, BeforeAll, AfterAll } from '@cucumber/cucumber';
import { CustomWorld } from '@support/world';

Before({ tags: '@login' }, async function (this: CustomWorld) {
    // Setup specific to login tests
    await this.page.context().clearCookies();
});

After({ tags: '@login' }, async function (this: CustomWorld) {
    // Cleanup after login tests
    await this.page.evaluate(() => localStorage.clear());
});
```

## Error Handling Pattern

```typescript
When('I perform a risky action', async function (this: CustomWorld) {
    try {
        await this.page.click('[data-testid="risky-button"]', { timeout: 5000 });
    } catch (error) {
        // Take screenshot for debugging
        await this.page.screenshot({ 
            path: `test-results/screenshots/error-${Date.now()}.png` 
        });
        throw new Error(`Failed to perform risky action: ${error.message}`);
    }
});
```

## Waiting Strategies

```typescript
// Wait for element
await this.page.waitForSelector('[data-testid="loaded"]');

// Wait for navigation
await this.page.waitForURL('**/dashboard');

// Wait for network idle
await this.page.waitForLoadState('networkidle');

// Wait for specific response
await this.page.waitForResponse(response => 
    response.url().includes('/api/data') && response.status() === 200
);

// Custom wait condition
await this.page.waitForFunction(() => {
    return document.querySelector('.loading') === null;
});
```
